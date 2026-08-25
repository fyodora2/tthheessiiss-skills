---
name: "rl-paper-implementation"
description: "Use when implementing an RL algorithm from a paper, reproducing a published baseline, or checking an implementation against reported numbers. Covers pseudocode-to-code translation, recovering hyperparameters the paper omits, documenting deviations, and staged verification against the paper's reported results."
---

# RL Paper Implementation Skill — Makale → Kod

## Temel Kural
> Bir algoritmayı "aşağı yukarı" implement etmek yeterli değildir.  
> Her uygulama detayı bir implementation deviation'dır ve raporlanmalıdır.

---

## Aşama 1 — Makaleyi Analiz Et (Kod Yazmadan Önce)

```markdown
## Algoritma Analiz Notu — <Algoritma Adı> (<Kaynak>)

### Pseudocode Satır Satır
[ ] 1. Satır → ne yapıyor?
[ ] 2. ...

### Kritik Hyperparametreler
| Parametre      | Paper Değeri | Kaynak (bölüm/tablo) |
|----------------|-------------|----------------------|
| Learning rate  | 3e-4        | Tablo 2              |
| Gamma          | 0.99        | Bölüm 3.1            |
| Batch size     | 256         | Tablo 2              |

### Belirsizlikler / Eksiklikler
[ ] Ağ mimarisi açıkça belirtilmemiş (Appendix'e bak)
[ ] Normalizasyon detayı belirsiz
[ ] Exploration stratejisi belirsiz

### Resmi Kod / Kaynak
- Orijinal kod: <URL>
- Başka implementasyon: <URL>
```

**Makalelerin sürekli atladığı detaylar** — bunları bulamadıysan varsaydığını yaz:
gözlem normalizasyonu, ödül ölçekleme/kırpma, gradient clipping değeri, ağ ağırlık başlatma şeması, learning rate scheduler, advantage normalizasyonu, epsilon değerleri (Adam `eps`, PPO `clip`), episode başlangıç rastgeleliği (no-op / sticky actions).

Bu detaylar RL'de tipik olarak algoritmik yeniliklerden daha büyük performans farkı yaratır.

---

## Aşama 2 — Implementation Deviation Kaydı

Makaleden herhangi bir sapma varsa **her sapma belgelenmelidir**:

```
Implementation Deviations from <Paper Title> (Author et al., Year)
================================================================

[DEV-001] Optimizer: Adam (paper) → AdamW (this impl.)
  Neden: weight decay ayrımı daha temiz
  Etki: ~%0.2 performans farkı (test edildi, ihmal edilebilir)

[DEV-002] LR Scheduler: Yok (paper) → Cosine annealing ekledik
  Neden: Uzun eğitimlerde stabilite
  Etki: Son epoch'larda %3-5 daha iyi val performance

[DEV-003] Target network update: Her 1000 step (paper) → Her 500 step
  Neden: Ortamımız daha hızlı değişiyor
  Etki: Bilinmiyor — ablation gerekli (TODO: DEV-003-ablation)
```

Etkisi "bilinmiyor" olan bir sapma, sonucu bildirirken belirtilmesi gereken bir sınırlılıktır. Sessizce bırakılamaz.

---

## Aşama 3 — Sanity Check Protokolü

Tam eğitime başlamadan önce implementasyonun sağlıklı olduğunu doğrula:

```python
# tests/test_algorithm_sanity.py

def test_policy_output_shape():
    """Policy doğru shape çıktı veriyor mu?"""
    policy = MyPolicy(obs_dim=4, action_dim=2)
    obs = torch.randn(32, 4)
    action, log_prob, value = policy(obs)

    assert action.shape == (32, 2)
    assert log_prob.shape == (32,)
    assert value.shape == (32,)

def test_value_network_bounds():
    """Value function makul bir aralıkta mı?"""
    value = critic(obs)
    assert value.abs().max() < 1000, f"Unrealistic value: {value.max()}"

def test_gradient_flows():
    """Gradient tüm parametrelere akıyor mu?"""
    loss = compute_loss(batch)
    loss.backward()

    for name, param in model.named_parameters():
        assert param.grad is not None, f"No gradient: {name}"
        assert not torch.isnan(param.grad).any(), f"NaN gradient: {name}"

def test_overfit_single_batch():
    """Tek batch üzerinde overfit edebiliyor mu? (temel doğruluk testi)"""
    batch = collect_batch(n_steps=100)
    initial_loss = compute_loss(batch).item()

    for _ in range(200):
        loss = compute_loss(batch)
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()

    final_loss = compute_loss(batch).item()
    assert final_loss < initial_loss * 0.1, \
        f"Model tek batch'i overfit edemedi: {initial_loss:.3f} → {final_loss:.3f}"

def test_cartpole_learns():
    """Basit CartPole-v1 üzerinde öğrenme oluyor mu?"""
    # CartPole max=500, random baseline ~20
    agent = MyAlgorithm(env="CartPole-v1", seed=42)
    result = agent.train(total_steps=50_000)

    assert result["final_mean_reward"] > 100, \
        f"CartPole baseline geçilemedi: {result['final_mean_reward']:.1f}"
```

> `test_overfit_single_batch` en değerli testtir. Geçemiyorsa problem hiperparametrede değil, kodda bir yerdedir — gradient akmıyor, loss yanlış tanımlı ya da veri hedefle eşleşmiyor demektir.

---

## Aşama 4 — Baseline Reproduksiyon Protokolü

```
1. Paper'dan tam hyperparameter tablosunu al
2. Tam olarak aynı ortamı kur (versiyon dahil)
3. Aynı seed'leri dene (paper seed açıkladıysa)
4. %15 içinde misiniz? → Başarılı reproduksiyon
   %15'i aşıyorsanız → DEV notlarını gözden geçir

Kabul Kriteri (NeurIPS Reproducibility Challenge standardı):
   ✅ Mean ±15% içinde (paper ortalama vs biz)
   ✅ Aynı trend (ortamlar arası sıralama korunuyor)
   ❌ Tamamen farklı davranış → Implementation hatası
```

Trend'in korunması mutlak sayıdan önemlidir. Ortamlar arası sıralama tutuyorsa ama seviye kayıyorsa sebep genelde ortam sürümü veya normalizasyondur; sıralama bozulduysa algoritmada gerçek bir hata vardır.

```markdown
=== Reproduksiyon Raporu ===
Paper: <başlık>
Ortam: <gym versiyonu>, <kütüphane versiyonları>
Seed: <kullanılan seed'ler>

| Metrik | Paper | Bizim | Fark |
|--------|-------|-------|------|
| HalfCheetah-v4 | 12345 | 11890 | -3.7% ✅ |
```

---

## Kontrol Listesi

**Makale Okuma**
- [ ] Pseudocode tam olarak anlaşıldı mı?
- [ ] Tüm hyperparametreler kaynağıyla listelendi mi?
- [ ] Belirsizlikler ve varsayımlar not alındı mı?
- [ ] Resmi kod incelendi mi?

**Implementation**
- [ ] Her deviation [DEV-XXX] ile belgelendi mi?
- [ ] Sanity check testleri geçiyor mu?
- [ ] Tek batch overfit testi geçiyor mu?
- [ ] CartPole/basit ortamda öğrenme oluyor mu?

**Reproduksiyon**
- [ ] Paper sonuçları %15 içinde yeniden üretildi mi?
- [ ] Trend (ortamlar arası sıralama) korunuyor mu?
- [ ] Etkisi bilinmeyen deviasyonlar sınırlılık olarak yazıldı mı?

