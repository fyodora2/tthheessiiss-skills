---
name: "dataset-documentation"
description: "Use when creating, publishing, or describing a dataset for academic or technical use. Produces Datasheets-for-Datasets style documentation: provenance, license, collection process, summary statistics, preprocessing steps, and known limitations."
---

# Dataset Documentation Skill — Veri Seti Belgeleme

## Temel Kural
> Belgelenmemiş veri güvenilmez veridir.

"Datasheet for Datasets" (Gebru et al., 2021) standardı esas alınmıştır.

---

## Zorunlu Belgeleme Bileşenleri

### 1. Köken (Provenance)
- Veri kim tarafından, ne zaman, hangi amaçla toplandı?
- Kaynak URL / DOI
- Lisans: CC-BY, CC0, MIT, özel vs.
- Kullanım kısıtlamaları var mı?

### 2. İçerik İstatistikleri

```python
import pandas as pd
import numpy as np

def dataset_summary(df: pd.DataFrame) -> dict:
    """Generate standard dataset statistics."""
    return {
        "n_samples": len(df),
        "n_features": df.shape[1],
        "missing_values": df.isnull().sum().to_dict(),
        "missing_pct": (df.isnull().sum() / len(df) * 100).to_dict(),
        "dtypes": df.dtypes.astype(str).to_dict(),
        "numeric_stats": df.describe().to_dict(),
        "class_distribution": df.iloc[:, -1].value_counts().to_dict()  # label column
    }
```

### 3. Bölümleme Bilgisi
```
Toplam: N = XXXXX örnek
├── Eğitim: N = XXXXX (%XX)
├── Validasyon: N = XXXXX (%XX)
└── Test: N = XXXXX (%XX)

Bölümleme yöntemi: [stratified random / chronological / site-based]
Bölümleme kodu/seed: seed=42
```

### 4. Etiket Dağılımı

```python
import matplotlib.pyplot as plt

# Sınıf dengesi görselleştirme
label_counts = df['label'].value_counts()
imbalance_ratio = label_counts.max() / label_counts.min()

if imbalance_ratio > 3:
    print(f"⚠️ Sınıf dengesizliği tespit edildi: {imbalance_ratio:.1f}:1")
    print("Oversampling, undersampling veya class-weight önerilir.")
```

### 5. Ön İşleme Adımları (Pipeline)
Her adım belgelenmeli:
```
1. Ham veri → [normalizasyon/standardizasyon]
2. Eksik değer işleme: [silindi/dolduruldu — hangi strateji?]
3. Outlier işleme: [IQR/Z-score — eşik ne?]
4. Feature engineering: [hangi özellikler türetildi]
5. Encoding: [one-hot/label/target encoding]
```

---

## Veri Seti Datasheet Şablonu

```markdown
# Veri Seti: [İsim]

## Genel Bilgi
- **Sürüm**: v1.0
- **Oluşturma Tarihi**: YYYY-MM
- **Güncel Sürüm Tarihi**: YYYY-MM
- **Lisans**: [lisans türü]
- **DOI / URL**: [bağlantı]

## Amaç
Bu veri seti [hangi görev için] oluşturulmuştur.

## İçerik İstatistikleri
| Metrik | Değer |
|---|---|
| Toplam Örnek | N = XXXXX |
| Özellik Sayısı | X |
| Hedef Değişken | [açıklama] |
| Eksik Değer | %X |
| Zaman Aralığı | YYYY – YYYY |

## Sınıf Dağılımı
| Sınıf | N | Yüzde |
|---|---|---|
| Sınıf A | XXX | XX% |
| Sınıf B | XXX | XX% |

## Toplama Süreci
[Nasıl toplandı, hangi araçlar kullanıldı, onay/etik süreçleri]

## Ön İşleme
[Adım adım ön işleme pipeline'ı]

## Bilinen Sınırlılıklar
- [Coğrafi kısıt, zaman kısıtı, temsil eksikliği vb.]

## Potansiyel Yanlılıklar
- [Seçilim yanlılığı, ölçüm yanlılığı vb.]

## Etik Değerlendirme
- Kişisel veri içeriyor mu? [Evet/Hayır]
- Anonimleştirme yapıldı mı? [Yöntem]
- IRB/Etik kurul onayı: [Evet/Hayır/N/A]

## Atıf
[BibTeX kaydı]
```

---

## Lisans Kontrol Tablosu

| Lisans | Ticari Kullanım | Değişiklik | Atıf Zorunluluğu |
|---|---|---|---|
| CC0 | ✅ | ✅ | ❌ |
| CC-BY 4.0 | ✅ | ✅ | ✅ |
| CC-BY-NC 4.0 | ❌ | ✅ | ✅ |
| CC-BY-SA 4.0 | ✅ | ✅ (aynı lisansla) | ✅ |
| MIT | ✅ | ✅ | ✅ |
| Özel/Kısıtlı | Kontrol et | Kontrol et | Kontrol et |

> ⚠️ **Uyarı**: Lisans kontrolü yapmadan veri seti kullanmayın.

