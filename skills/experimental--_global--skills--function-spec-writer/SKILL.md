---
name: "function-spec-writer"
description: "Use before implementing any function, to produce a complete FunctionSpec: type-annotated signature, Google-style docstring, preconditions and postconditions, edge cases, and concrete test cases. Runs before the implementation is written, not after."
---

# Function Spec Writer Skill — Fonksiyon Spec Yazarı

## Temel Kural
> Yazmadan önce tasarla. Belirsiz spec, hatalı implementasyona yol açar.

---

## FunctionSpec Üretim Protokolü

### Adım 1 — Fonksiyonun Amacını Netleştir
```
Tek cümle: "Bu fonksiyon [girdi alır] ve [çıktı üretir / yan etki yapar]."

Eğer "ve" birden fazla kez kullanıyorsanız → fonksiyonu böl.
```

### Adım 2 — İmzayı Tasarla (Type Contract)
```python
def function_name(
    param1: Type1,
    param2: Type2,
    *,  # keyword-only sınırı (gerekirse)
    optional_param: Type3 = default,
) -> ReturnType:
```

Tip seçim kılavuzu:
- `str` değil `list[str]` ya da `tuple[str, ...]` — mümkün olan en spesifik tip
- `Optional[X]` yerine `X | None` (Python 3.10+)
- `dict` yerine `TypedDict` veya `dataclass` (karmaşık yapılar için)
- `Any` yasak — kullanmak zorundaysanız gerekçe belirt

### Adım 3 — Ön/Son Koşulları Belirle

```python
# Ön koşullar (Pre-conditions) — fonksiyon çağrılmadan önce doğru olmalı:
# - param1 boş olmamalı
# - param2 pozitif olmalı
# - param1 ile param2 aynı boyutta olmalı

# Son koşullar (Post-conditions) — fonksiyon döndükten sonra doğru olmalı:
# - Sonuç her zaman [0, 1] aralığında
# - Boş girdi → ValueError fırlatılır
# - Sonuç değişmez (immutable) olmalı
```

### Adım 4 — Edge Case Listesi

Sistematik edge case kategorileri:
```
Veri edge case'leri:
□ Boş girdi ([], "", None, 0)
□ Tek elemanlı girdi
□ Maksimum boyut
□ Negatif değerler
□ Sıfır
□ Özel karakterler (unicode, emoji)

Tip edge case'leri:
□ Yanlış tip (str yerine int)
□ None değeri

Sınır değerleri:
□ int.min, int.max
□ float('inf'), float('-inf'), float('nan')
□ Tam eşik değeri (threshold tam 0.5 ise → 0.5 ne döner?)
```

---

## FunctionSpec JSON Şablonu

```json
{
  "function_name": "calculate_precision",
  "module": "metrics",
  "priority": 1,
  "signature": "def calculate_precision(y_true: np.ndarray, y_pred: np.ndarray) -> float",
  "return_type": "float",
  "docstring": "Calculate precision score.\n\nArgs:\n    y_true: Ground truth binary labels of shape (n_samples,).\n    y_pred: Predicted binary labels of shape (n_samples,).\n\nReturns:\n    Precision score as float in [0.0, 1.0].\n\nRaises:\n    ValueError: If arrays have different shapes.\n    ValueError: If arrays are empty.",
  "parameters": [
    {
      "name": "y_true",
      "type": "np.ndarray",
      "description": "Ground truth labels",
      "constraints": "Shape (n,), values in {0, 1}, non-empty"
    },
    {
      "name": "y_pred",
      "type": "np.ndarray",
      "description": "Predicted labels",
      "constraints": "Same shape as y_true"
    }
  ],
  "preconditions": [
    "y_true.shape == y_pred.shape",
    "len(y_true) > 0",
    "all values in {0, 1}"
  ],
  "postconditions": [
    "0.0 <= result <= 1.0",
    "No exception if preconditions met"
  ],
  "edge_cases": [
    "All predictions negative → precision = 0.0",
    "All predictions correct → precision = 1.0",
    "Empty arrays → ValueError",
    "Shape mismatch → ValueError"
  ],
  "test_cases": [
    {
      "description": "Perfect precision",
      "inputs": {"y_true": [1,0,1,0], "y_pred": [1,0,1,0]},
      "expected": 1.0
    },
    {
      "description": "No true positives",
      "inputs": {"y_true": [0,0,0], "y_pred": [1,1,1]},
      "expected": 0.0
    },
    {
      "description": "Empty arrays raise ValueError",
      "inputs": {"y_true": [], "y_pred": []},
      "expected_exception": "ValueError"
    }
  ],
  "dependencies": ["numpy"],
  "file_path": "src/metrics/precision.py",
  "test_file_path": "tests/test_metrics/test_precision.py"
}
```

---

## Kalite Kontrol

Spec tesliminden önce:
- [ ] İmza tip-annotated mı?
- [ ] Docstring Args + Returns + Raises içeriyor mu?
- [ ] En az 1 mutlu yol (happy path) test case var mı?
- [ ] En az 1 hata/exception test case var mı?
- [ ] En az 2 edge case var mı?
- [ ] Ön/son koşullar açık mı?
- [ ] "Tek şey" yapıyor mu? (tek sorumluluk)

