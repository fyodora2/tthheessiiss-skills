---
name: "hypothesis-framing"
description: "Use when formulating a research question, defining a hypothesis, or setting up the experimental frame for a study. Turns a vague research direction into specific, measurable, falsifiable hypotheses with explicit null and alternative statements and named independent and dependent variables."
---

# Hypothesis Framing Skill — Hipotez Çerçeveleme

## Temel Kural
> İyi bir hipotez yanlışlanabilir, ölçülebilir ve net olmalıdır.

---

## Araştırma Sorusu → Hipotez Dönüşümü

### Adım 1 — Araştırma Sorusunu Yapılandır (PICO)

| Bileşen | Açıklama | Örnek |
|---|---|---|
| **P** — Population | Hangi grup/sistem? | "Türkçe metin sınıflandırma görevlerinde" |
| **I** — Intervention | Hangi müdahale/yöntem? | "BERT tabanlı modeller" |
| **C** — Comparison | Neyle karşılaştırma? | "geleneksel TF-IDF + SVM'e kıyasla" |
| **O** — Outcome | Hangi sonuç bekleniyor? | "F1-macro skoru daha yüksek mi?" |

### Adım 2 — Hipotez Çiftini Formüle Et

```
H₀ (Null Hipotez): [Fark yok / etki yok ifadesi]
Örnek: "BERT ve TF-IDF+SVM arasında F1-macro skoru bakımından 
        istatistiksel olarak anlamlı bir fark yoktur."

H₁ (Alternatif Hipotez): [Beklenen fark / etki ifadesi]
Örnek: "BERT tabanlı modeller, Türkçe metin sınıflandırma 
        görevlerinde TF-IDF+SVM'e kıyasla istatistiksel olarak 
        anlamlı biçimde daha yüksek F1-macro skoru elde eder."
```

### Adım 3 — Ölçülebilirlik Kontrolü

Hipotez şu soruları yanıtlamalıdır:
- [ ] Hangi metrik ölçülecek? (belirsiz → reddedilir)
- [ ] Kaç katılımcı/örnek ile? (sample size belirlendi mi?)
- [ ] Hangi koşulda? (kontrol değişkenleri tanımlandı mı?)
- [ ] Hangi eşikte anlamlı sayılacak? (α = 0.05 varsayılan)

---

## Hipotez Kalite Kontrol Listesi

- [ ] **Özgüllük**: "Daha iyi" yerine "X metriğinde Y kadar daha yüksek"
- [ ] **Yanlışlanabilirlik**: Test edilebilir mi? Hangi sonuç H0'ı reddeder?
- [ ] **Tek değişken**: Bir hipotezde yalnızca bir şey test ediliyor mu?
- [ ] **Null form**: H0 açıkça formüle edildi mi?
- [ ] **Yön belirtimi**: Tek yönlü mü (H1: A > B) çift yönlü mü (H1: A ≠ B)?
- [ ] **Etki büyüklüğü hedefi**: Pratik olarak anlamlı minimum etki ne?

---

## Yaygın Hipotez Hataları

| Hata | Örnek | Düzeltme |
|---|---|---|
| Belirsiz sonuç | "Performans artacaktır" | "F1-macro skoru ≥ 0.05 artacaktır" |
| Ölçülemez | "Kalite iyileşecektir" | "BLEU skoru artacaktır" |
| Çok değişkenli | "A hem hızlı hem doğru olacaktır" | İki ayrı hipotez |
| Döngüsel | "Daha iyi yöntem daha iyi sonuç verir" | Operasyonel tanım ekle |
| Doğrulanamaz | "İnsan uzmanlar fark edemeyecektir" | Ölçüm protokolü tanımla |

---

## Şablon

```markdown
## Araştırma Sorusu
[PICO formatında, bir paragraf]

## Hipotezler

**H₀**: [Null hipotez — fark yok ifadesi]

**H₁**: [Alternatif hipotez — beklenen etki, yön ve büyüklük]

## Operasyonel Tanımlar
- Bağımlı Değişken: [metrik adı, nasıl ölçülecek]
- Bağımsız Değişken: [manipüle edilen faktör]
- Kontrol Değişkenleri: [sabit tutulan faktörler]

## Karar Kuralı
H₀, [istatistiksel test adı] sonucunda p < [α] ve [etki büyüklüğü eşiği] 
koşullarının her ikisi de sağlandığında reddedilecektir.
```

