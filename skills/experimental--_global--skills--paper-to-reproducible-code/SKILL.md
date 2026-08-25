---
name: "paper-to-reproducible-code"
description: "Use when checking that a paper's reported tables, hyperparameters, and equations match what the code actually runs. Audits paper values against YAML/JSON configs, surfaces hyperparameters the paper leaves undocumented, and enforces one-to-one fidelity between published numbers and run parameters."
---

# Paper To Reproducible Code Skill — Makale ↔ Kod Parametre Denetimi

## Temel Kural
> Makale metninde veya tablolarında yazan denklem ve hiperparametreler ile `configs/*.yaml` ve Python fonksiyonları %100 örtüşmelidir.  
> Projede kilitli bir formülasyon kaydı varsa (aşağıdaki nota bakın), çelişki durumunda düzeltilen koddur.

---

## Parametre Denetim Protokolü

```markdown
## Parametre Denetim Raporu — <Paper / Deney Adı>

### 1. Parametre Eşleşme Matrisi
| Parametre | Paper / Tablo Değeri | Code Config Değeri | Durum |
|-----------|----------------------|--------------------|-------|
| Learning Rate | 3e-4 (Tablo 2) | `lr: 0.0003` | ✅ Eşleşti |
| Discount Factor (γ) | 0.99 | `gamma: 0.99` | ✅ Eşleşti |
| GAE Lambda (λ) | 0.95 | `lambda: 0.95` | ✅ Eşleşti |
| Batch Size | 256 | `batch_size: 512` | ❌ UYUMSUZ |
| Target Update Freq | 1000 steps | `target_update: 500` | ❌ UYUMSUZ |

### 2. Makalede Belgelenmemiş Hiperparametreler
Kodda değeri olup makalede hiç geçmeyen parametreler. Bunlar tekrarlanabilirlik açığıdır
ve ya makaleye ya da eke eklenmelidir.

| Parametre | Koddaki Değer | Sonucu etkiler mi? |
|---|---|---|
| `grad_clip` | 0.5 | Evet — eğitim kararlılığı |
| `obs_normalize` | True | Evet — çoğu ortamda büyük fark |
| `n_eval_episodes` | 10 | Evet — varyans tahmini |
```

---

## Otomatik Uyum Denetim Scripti

```python
# scripts/audit_paper_params.py
import yaml
import json
import sys
from pathlib import Path

def audit_config(config_path: Path, expected_params: dict):
    """Compare yaml/json config against expected paper values."""
    with open(config_path) as f:
        config = yaml.safe_load(f)

    mismatches = []
    for key, expected_val in expected_params.items():
        actual_val = config.get(key)
        if actual_val != expected_val:
            mismatches.append((key, expected_val, actual_val))

    if mismatches:
        print(f"❌ Parametre Uyumsuzluğu ({config_path.name}):")
        for key, exp, act in mismatches:
            print(f"   - {key}: Beklenen={exp}, Koddaki={act}")
        return False

    print(f"✅ Tüm parametreler makale ile uyumlu: {config_path.name}")
    return True
```

---

## Uyumsuzluk Bulunduğunda

Bir uyumsuzluk iki şeyden biridir; hangisi olduğunu belirlemeden düzeltme yapma:

1. **Makale yanlış** — kod doğru çalıştı, tabloya yanlış değer yazıldı. Makale düzeltilir, sonuçlar geçerli kalır.
2. **Kod yanlış** — rapor edilen sonuçlar aslında farklı bir konfigürasyonla üretildi. Bu bir TYPE B bug'dır: önceki sonuçlar geçersizdir, yeniden çalıştırma gerekir.

İkisini karıştırmak, sessizce sonuç uydurmakla aynı kapıya çıkar.

---

## Not — Formülasyon Kaydı

Projede formülasyon kaydı (kanonik denklemleri, sembolleri ve parametre değerlerini tutan `FORMULATION.md` benzeri bir belge) varsa dinamik olarak çözümle: sırasıyla `.claude/context/`, `.agents/context/` ve proje köküne bak. Böyle bir dosya yoksa hangi belgenin bağlayıcı olduğunu kullanıcıya sor — varsayma. Varsa kullanıcı-kilitli kabul et: kod ile kayıt çelişirse düzeltilen koddur.

