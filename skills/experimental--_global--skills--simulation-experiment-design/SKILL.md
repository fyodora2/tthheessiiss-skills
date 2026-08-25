---
name: "simulation-experiment-design"
description: "Use when designing a simulation-based experiment for research. Bridges experimental design (control variables, statistical power, confound identification) with simulation-specific concerns: simulator non-determinism, environment version pinning, warm-up periods, and threats to sim-to-real validity."
---

# Simulation Experiment Design Skill — Simülasyon Deneyi Tasarımı

## Temel Kural
> Simülasyon ucuz olduğu için fazla deney yapmak cazip gelir.  
> Ama plansız çok sayıda deney, hata düzeltmesi yapılmamış çok sayıda sonuç üretir.

---

## Deney Tasarım Formu (Her Deneyden Önce Doldur)

```markdown
## Deney Tasarım Belgesi — <Deney Adı>

**Tarih**: YYYY-MM-DD
**Bağlı Araştırma Sorusu**: <hypothesis-framing çıktısı>

### 1. Amaç
Bu deney hangi soruyu cevaplayacak?
→ [Bir cümle]

### 2. Bağımsız Değişkenler (Manipüle Edilenler)
| Değişken      | Değerler                  | Neden bu değerler? |
|---------------|---------------------------|-------------------|
| Ödül şekli    | baseline, potential, dense | Ablation          |
| Öğrenme oranı | 1e-4, 3e-4, 1e-3          | Grid search       |

### 3. Bağımlı Değişkenler (Ölçülenler)
| Metrik           | Ölçüm Yöntemi    | Başarı Kriteri |
|------------------|------------------|----------------|
| Mean episode reward | 100 eval ep.  | > 200          |
| Sample efficiency | Steps to 150 reward | < 500K    |

### 4. Kontrol Altına Alınanlar (Sabitler)
- Algoritma: PPO
- Ağ mimarisi: [64, 64] MLP
- Seed'ler: [42, 123, 456, 789, 1024]
- Ortam: HalfCheetah-v4, mujoco==2.3.7
- Donanım: RTX 4090, CUDA 12.1

### 5. Potansiyel Confound'lar
- Ortam versiyonu farkı (simülatör güncellenmesi)
- GPU non-determinizm
- Paralel env sayısı etkisi

### 6. Bütçe
- Her konfigürasyon: 5 seed × 1M step × ~2 saat = 10 saat
- Toplam konfigürasyon: 6
- Toplam süre: ~60 GPU saati
- Deadline: YYYY-MM-DD
```

**Bu belge deney başlamadan önce yazılır.** Sonuçları gördükten sonra "aslında şunu ölçüyorduk" demek, hipotezi veriye uydurmaktır.

---

## Simülatör Non-Determinizm Yönetimi

Simülatörler seed'e rağmen non-deterministik olabilir:

```python
import os
import gymnasium as gym

# 1. Thread sayısını sabitle (paralel işlem non-determinizm kaynağı)
os.environ["OMP_NUM_THREADS"] = "1"
os.environ["MKL_NUM_THREADS"] = "1"

# 2. Her sıfırlamada tam seed chain
class DeterministicWrapper(gym.Wrapper):
    """Ensures full seed propagation through the simulation stack."""

    def reset(self, seed=None, **kwargs):
        if seed is not None:
            self.env.np_random, _ = gym.utils.seeding.np_random(seed)
        return super().reset(seed=seed, **kwargs)

# 3. Floating point determinizm (dikkat: yavaşlatır)
import torch
torch.use_deterministic_algorithms(True)
os.environ["CUBLAS_WORKSPACE_CONFIG"] = ":4096:8"

# 4. Aynı başlangıç noktası doğrulama
env = DeterministicWrapper(gym.make("HalfCheetah-v4"))
obs1, _ = env.reset(seed=42)
obs2, _ = env.reset(seed=42)
assert (obs1 == obs2).all(), "Simülatör deterministic değil!"
```

Tam determinizm her zaman gerekmez ve pahalıdır. Gerçekten gereken şey **aynı seed ile aynı sonucu tekrar üretebilmek**; bunu bir kez doğrula, sonra determinizmi kapatıp hızlı koş.

---

## Deney Faktöriyel Tasarımı

```python
from itertools import product
from dataclasses import dataclass

@dataclass
class ExperimentConfig:
    reward_type: str
    learning_rate: float
    seed: int

    @property
    def name(self) -> str:
        return f"reward={self.reward_type}_lr={self.learning_rate:.0e}_seed={self.seed}"

reward_types = ["baseline", "potential", "dense"]
learning_rates = [1e-4, 3e-4]
seeds = [42, 123, 456]

all_configs = [
    ExperimentConfig(r, lr, s)
    for r, lr, s in product(reward_types, learning_rates, seeds)
]

print(f"Toplam deney sayısı: {len(all_configs)}")  # 18

hours_per_exp = 2.0
print(f"Tahmini süre: {len(all_configs) * hours_per_exp:.0f} GPU saat")
```

Konfigürasyon sayısı hızla patlar. Tam faktöriyel yerine, bir seferde tek faktör değiştiren ablation dizisi genelde hem daha ucuz hem de yorumlanması daha kolaydır.

---

## İstatistiksel Güç Analizi (Kaç Seed Yeterli?)

```python
import numpy as np
from scipy.stats import norm

def required_seeds(
    effect_size: float,    # Beklenen fark / std (Cohen's d)
    alpha: float = 0.05,   # Tip I hata
    power: float = 0.80,   # 1 - Tip II hata
) -> int:
    """Kaç seed gerekli sorusunu istatistiksel olarak yanıtla."""
    z_alpha = norm.ppf(1 - alpha / 2)
    z_beta = norm.ppf(power)
    n = ((z_alpha + z_beta) / effect_size) ** 2
    return int(np.ceil(n))

# RL için tipik değerler (grup başına):
# Küçük etki (d=0.2) → ~393 seed (imkansız)
# Orta etki  (d=0.5) → ~63 seed  (çok fazla)
# Büyük etki (d=0.8) → ~25 seed
# Çok büyük  (d=1.0) → ~16 seed
```

Bunun rahatsız edici sonucu: **5 seed yalnızca çok büyük etkileri tespit edebilir.** Alanın standardı olan 5 seed, küçük iyileştirmeler için istatistiksel olarak yetersizdir. Küçük bir fark bulup 5 seed ile "anlamlı" ilan etmek, güç analizini görmezden gelmektir. Ya seed sayısını artır, ya da etkinin küçük olduğunu ve tespit gücünün sınırlı kaldığını açıkça yaz.

---

## Ortam Varyant Tasarımı

Tek ortamda test yetmez — genelleştirilebilirlik için varyantlar:

```python
# Eğitim ortamları
TRAIN_CONFIGS = [
    {"gravity": 9.81, "friction": 0.8},   # Normal
    {"gravity": 9.81, "friction": 0.4},   # Düşük sürtünme
    {"gravity": 15.0, "friction": 0.8},   # Yüksek yerçekimi
]

# Test ortamları (hiç görülmemiş koşullar)
TEST_CONFIGS = [
    {"gravity": 12.0, "friction": 0.6},   # Ara değer (interpolation)
    {"gravity": 20.0, "friction": 1.2},   # Aşırı değer (extrapolation)
]

TRAIN_SEEDS = [42, 123, 456, 789, 1024]
TEST_SEEDS  = [9999, 8888, 7777]  # Train seed'lerinden farklı
```

Interpolation ve extrapolation performansını ayrı raporla — aradaki fark, yöntemin gerçekten genelleştiğini mi yoksa eğitim dağılımını mı ezberlediğini gösterir.

---

## Sim-to-Real Geçerlilik Tehditleri

Simülasyon sonucundan gerçek dünya iddiası çıkarılacaksa açıkça ele alınmalı:

- **Gözlem gürültüsü** — simülatör mükemmel sensör verir; gerçek sistem vermez
- **Aktüatör gecikmesi** — simülasyonda anlık, gerçekte gecikmeli ve doygun
- **Modellenmemiş dinamik** — sürtünme, esneklik, sıcaklık sürüklenmesi
- **Reality gap ölçümü** — domain randomization uygulandıysa, hangi parametrelerin hangi aralıkta rastgeleleştirildiğini yaz

Bunlar ele alınmadıysa, sonuç bir simülasyon sonucudur ve öyle sunulmalıdır.

---

## Deney Tamamlama Kontrol Listesi

**Başlamadan Önce**
- [ ] Deney tasarım belgesi dolduruldu mu?
- [ ] Bütçe hesaplandı mı? (GPU saat, deadline)
- [ ] Determinizm bir kez doğrulandı mı?
- [ ] Seed sayısı güç analiziyle gerekçelendirildi mi?
- [ ] Ortam ve kütüphane sürümleri sabitlendi mi?

**Çalışırken**
- [ ] WandB/MLflow logları geliyor mu?
- [ ] İlk 10K step'te makul bir öğrenme var mı?
- [ ] NaN/Inf var mı?
- [ ] Checkpoint'ler düzgün kaydediliyor mu?

**Bitiminde**
- [ ] Tüm seed'ler tamamlandı mı (yarım kalanlar sessizce atlanmadı mı)?
- [ ] Sonuçlar Git'e commit edildi mi (`result:` tipi ile)?
- [ ] İstatistiksel analiz yapıldı mı?
- [ ] Tespit gücü sınırlıysa bu bir sınırlılık olarak yazıldı mı?

