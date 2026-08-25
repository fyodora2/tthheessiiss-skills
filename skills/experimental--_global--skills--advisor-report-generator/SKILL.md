---
name: "advisor-report-generator"
description: "Use when writing a progress report, milestone update, or diagnostic briefing for a thesis advisor or supervisor. Structures it around the research question, what changed since the last report, current results with their statistical support, and the open decisions the advisor needs to weigh in on."
---

# Advisor Report Generator Skill — Danışman Raporlama

## Temel İlke
> Danışman hocaya sunulan rapor koddaki ayrıntılarda boğulmamalı; yüksek seviyeli akademik motivasyon, sağlam teorik temeller, istatistiksel kanıtlar ve net karar seçenekleri sunmalıdır.

---

## Rapor Türleri ve Yapıları

### 1. Haftalık / Dönemsel İlerleme Raporu

- **1. Yönetici Özeti:** Bu hafta ne başarıldı? (3-4 maddelik yüksek seviyeli özet)
- **2. Araştırma Bağlamı ve Motivasyon:** Bu haftaki çalışmanın genel projedeki yeri ve hipotezlerle ilişkisi (`hypothesis-framing`).
- **3. Yöntem ve Teorik Güncellemeler:** Algoritmik/matematiksel değişiklikler, formülasyon kaydındaki sembolizmle uyumlu biçimde.
- **4. Deneysel Bulgular ve İstatistiksel Analiz:** Tablo/grafik destekli sonuçlar (`statistical-validity`).
- **5. Karşılaşılan Engeller ve Olumsuz Sonuçlar:** Neden çalışmadı? Ampirik kanıtlar (`empirical-rigor`).
- **6. Danışman Görüşüne Sunulan Maddeler ve Sonraki Adımlar:** Hocadan onay/tavsiye beklenen net kararlar.

### 2. Deney & Derin İnceleme Raporu

- **1. Deneyin Amacı ve Hipotez:** Hangi H₀/H₁ hipotezi test ediliyor?
- **2. Deney Kurulumu:** Basitleştirilmiş parametre tablosu (kod detayları olmadan).
- **3. Karşılaştırmalı Analiz:** Baseline'lar ile kıyaslama (`fair-comparison`).
- **4. Ana Çıkarımlar ve Teorik Yorum:** Bulguların literatürdeki karşılığı.

---

## Danışman Raporlama Kuralları

1. **Ham Kod Yerine Pseudocode / Akış Şeması:** Rapora asla 50 satırlık ham Python kodu koymayın. Bunun yerine pseudocode veya yüksek seviyeli mimari şeması sunun.
2. **Formülasyon Bütünlüğü:** Projenin formülasyon kaydındaki matematiksel sembolizm ile %100 uyumlu olun (aşağıdaki nota bakın).
3. **Dürüst Raporlama:** Olumsuz sonuçları gizlemeyin; neden başarısız olduğunu ampirik verilerle açıklayın.
4. **Zaman Tasarruflu Tasarım:** Hocanın raporu 2 dakikada tarayıp ana mesajı anlayabileceği kalın punto vurgular ve özet tablolar kullanın.

---

## Not — Formülasyon Kaydının Çözümlenmesi

Formülasyon kaydı, projenin kanonik denklemlerini, sembollerini ve parametre değerlerini tutan belgedir (tipik olarak `FORMULATION.md`). Dinamik olarak çözümle: sırasıyla `.claude/context/`, `.agents/context/` ve proje köküne bak. Böyle bir dosya yoksa, hangi belgenin bağlayıcı olduğunu kullanıcıya sor — varsayma. Dosya varsa kullanıcı-kilitli kabul et: kod ile kayıt çelişirse düzeltilen koddur.

