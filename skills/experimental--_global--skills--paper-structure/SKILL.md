---
name: "paper-structure"
description: "Use when writing an academic paper, conference submission, journal article, or thesis chapter. Enforces IMRaD structure with section-by-section guidance on what belongs where, what claim each section must carry, and the structural failures reviewers punish."
---

# Paper Structure Skill — Akademik Yazı Yapısı

## Temel Kural
> Akademik yazının gücü içeriğin değil yapının netliğinden gelir.

---

## IMRaD Yapısı

```
📄 Başlık
📝 Özet (Abstract)
─────────────────────────────────
1. Giriş (Introduction)
2. İlgili Çalışmalar (Related Work)  ← Kongreye göre değişir
3. Yöntem (Methodology / Methods)
4. Sonuçlar (Results)
5. Tartışma (Discussion)
6. Sonuç (Conclusion)
─────────────────────────────────
Teşekkür (Acknowledgments)
Kaynakça (References)
Ekler (Appendix)  ← Gerekirse
```

---

## Bölüm Başına Yazım Standartları

### Başlık
- Spesifik, bilgi taşıyan başlık (merak uyandıran DEĞİL)
- Anahtar kelimeler başlıkta geçmeli (SEO + indeksleme)
- Maksimum 15-20 kelime

```
✅ "BERT-Based Turkish Sentiment Analysis with Domain-Adaptive Pre-training"
❌ "A Novel Approach to Sentiment Analysis"  → Aşırı genel
❌ "Can Machines Understand Turkish Emotions?"  → Merak uyandırıcı ama belirsiz
```

### Özet (Abstract) — 150-300 kelime

Zorunlu unsurlar (bu sırayla):
1. **Problem**: Ne sorununu çözüyorsunuz?
2. **Motivasyon**: Neden önemli?
3. **Yöntem**: Ne yapıyorsunuz?
4. **Bulgular**: Ne buldunuz? (sayısal değerlerle)
5. **Sonuç**: Ne anlama geliyor?

```
❌ Özette atıf yapmayın
❌ "Bu çalışmada..." ile başlamayın
✅ Direkt problem ifadesiyle başlayın
✅ Sayısal sonuçları özete ekleyin ("X metriğinde %Y artış")
```

### Giriş (Introduction)

Paragraf yapısı:
```
Para 1: Alanın önemi / büyük resim
Para 2: Mevcut sorun / boşluk (problem gap)
Para 3: Önerilen yaklaşım (ne yapıldı)
Para 4: Katkılar listesi (bullet points kabul edilebilir)
Para 5: Makalenin yapısı ("Bölüm 2'de... Bölüm 3'te...")
```

**Altın Kural**: Giriş "Neden bu çalışmaya ihtiyaç var?" sorusunu yanıtlamalıdır.

### Yöntem (Methodology)

- Başkası aynı sonucu üretebilmeli (**tekrarlanabilirlik**)
- Her karar gerekçelendirilmeli ("X yerine Y kullandık çünkü...")
- Varsayımlar açıkça belirtilmeli
- Sözde kod veya akış diyagramı ekleyin

### Sonuçlar (Results)

- Sadece bulgular — yorum yok (yorumlar Discussion'a)
- Tablo > uzun sayı listesi
- Grafikler net etiketli, açıklayıcı başlıklı
- Her tablo/şekil metin içinde atıfla bağlanmalı

### Tartışma (Discussion)

Yapı önerisi:
```
1. Ana bulguyu yorumla (ne anlama geliyor?)
2. Beklentilerle karşılaştır (hipotez doğrulandı mı?)
3. İlgili çalışmalarla karşılaştır
4. Kısıtlamalar (dürüstçe)
5. Pratik çıkarımlar
6. Gelecek çalışma önerileri
```

### Sonuç (Conclusion)

- Yeni bilgi sunma (Giriş/Discussion'ı kopyalamak değil)
- En önemli 2-3 katkı
- Bir cümlelik büyük resim etkisi

---

## Geçiş Mantığı

Bölümler arası mantıksal akış:
```
Giriş son paragrafı → "Bu çalışmada şunu yapıyoruz"
    ↓
Yöntem → "Şu şekilde tasarladık"
    ↓
Sonuçlar → "Şu bulguları elde ettik"
    ↓
Tartışma → "Bu bulgular şu anlama geliyor"
    ↓
Sonuç → "Genel çıkarım budur"
```

---

## Yaygın Yapısal Hatalar

| Hata | Açıklama |
|---|---|
| Sonuç = Özet | Sonuç bölümü yeni içgörü sunmalı |
| Yorum Sonuçlarda | Yorumlar Tartışma'ya ait |
| Eksik Kısıtlamalar | Her çalışmanın sınırlılığı vardır |
| Giriş'te Sonuç | Giriş'te bulgulardan bahsetme |
| Orphan Figure | Metin içinde atıf olmayan tablo/şekil |

