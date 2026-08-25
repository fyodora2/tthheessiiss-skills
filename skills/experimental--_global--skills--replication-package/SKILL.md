---
name: "replication-package"
description: "Use when packaging code and data for paper submission, camera-ready, or public release alongside a publication. Covers what the package must contain, environment pinning, seed and config capture, a runnable end-to-end reproduction entry point, the README a reviewer needs, and the release workflow through Git tags and Zenodo DOI archiving."
---

# Replication Package Skill — Tekrarlama Paketi ve Yayın

## Temel Kural
> "Kodu paylaşıyorum" yeterli değildir. Çalıştırılabilir, belgelenmiş bir paket zorunludur.  
> "Kod mevcuttur, talep üzerine gönderilebilir" kabul edilemez.

ACM/IEEE artifact evaluation standartları temel alınmıştır.

---

## Paket İçeriği Kontrol Listesi

### Zorunlu Bileşenler
- [ ] **README.md** — Kurulum ve çalıştırma talimatları
- [ ] **requirements.txt veya environment.yml** — Tüm bağımlılıklar sürüm numarasıyla
- [ ] **Kaynak Kodu** — Temiz, yorumlanmış
- [ ] **Veri veya Veri İndirme Scripti** — Ham ve işlenmiş veri
- [ ] **Çalıştırma Scriptleri** — Her deneyi tekrar eden script
- [ ] **Çıktı Örnekleri** — Beklenen sonuçlar (referans için)
- [ ] **SEED dosyası veya yapılandırması** — Tüm random seed değerleri

### Önerilen Bileşenler (Opsiyonel)
- [ ] **Dockerfile** — Çevre izolasyonu (her zaman gerekmez — conda genellikle yeterli)
- [ ] **Makefile** — Tek komutla çalıştırma
- [ ] **Jupyter Notebook** — Analiz için
- [ ] **Figür Üretim Scripti** — Her figürü yeniden üreten script

---

## README.md Şablonu

```markdown
# [Makale Başlığı] — Tekrarlama Paketi

## Gereksinimler
- İşletim Sistemi: [Ubuntu 20.04 / macOS 12 / Windows 10]
- Python: 3.XX
- GPU: [gerekli/opsiyonel — model adı]
- Disk: XX GB
- RAM: XX GB

## Kurulum

### Conda ile
conda env create -f environment.yml
conda activate [env-name]

### pip ile
python -m venv venv
source venv/bin/activate  # Linux/Mac
venv\Scripts\activate     # Windows
pip install -r requirements.txt

## Veri İndirme
bash scripts/download_data.sh

Beklenen veri boyutu: ~XX GB
Beklenen süre: ~XX dakika

## Deneyleri Çalıştırma

### Ana Deneyi Tekrarlama (Tablo 2)
python run_experiments.py --config configs/main.yaml

Beklenen çalışma süresi: ~XX saat (GPU olmadan: ~XX saat)

### Tek Bir Deneyi Çalıştırma
python run_experiments.py --config configs/main.yaml --experiment [exp_name]

## Beklenen Sonuçlar
| Metrik | Makale | Beklenen Aralık |
|---|---|---|
| Accuracy | 0.923 | 0.920 – 0.926 |
| F1-Macro | 0.918 | 0.915 – 0.921 |

Not: Küçük sayısal farklar kabul edilebilir (±0.005);
     donanım farklılığı ve kütüphane versiyonundan kaynaklanabilir.

## Sorun Giderme
- [Yaygın hata 1]: [Çözüm]
- [Yaygın hata 2]: [Çözüm]

## Atıf
[BibTeX kaydı]
```

---

## Çevre Sabitleme

```bash
# Tam çevre dışa aktarımı
pip freeze > requirements_exact.txt
conda env export > environment_exact.yml

# Platform bağımsız (sadece direkt bağımlılıklar)
pip-compile requirements.in > requirements.txt

# Versiyon doğrulama scripti
python scripts/check_environment.py
```

---

## Artifact Evaluation Badge Kriterleri

Modern konferanslar (NeurIPS, ICML, ACL vb.) üç rozet sunar:

| Rozet | Kriter |
|---|---|
| **Artifacts Available** | Kod ve veri erişilebilir arşivde (Zenodo, GitHub Release) |
| **Artifacts Evaluated — Functional** | Çalışıyor, temel işlevleri doğrulandı |
| **Results Reproduced** | Bağımsız ekip orijinal sonuçları yeniden üretti |

---

# Yayın Akışı (Public Release)

## Yayın Öncesi Temizlik

- [ ] Tüm hardcoded path'ler kaldırıldı (örn. `/home/user/data/`)
- [ ] Debug print'ler temizlendi
- [ ] Kullanılmayan import'lar kaldırıldı
- [ ] TODO/FIXME'ler ya çözüldü ya belgelendi
- [ ] Büyük dosyalar (model ağırlıkları, veri) `.gitignore`'da
- [ ] Git geçmişinde hassas veri yok (`git log -S "password"` ile kontrol)

---

## Git Tag ve Zenodo Arşivleme

```bash
# 1. Camera-ready versiyonunu tag'le
git tag -a "v1.0.0-paper" -m "
Camera-ready code release

Corresponds to: <Paper Title>
Venue: <ICML/NeurIPS/ICLR 2026>
ArXiv: arxiv.org/abs/XXXX.XXXXX

Reproducibility:
  python scripts/reproduce_all.py --table 1
"

git push origin v1.0.0-paper

# 2. GitHub → Releases → "Create release from tag"
# 3. Zenodo bağlantısı varsa otomatik DOI alır
#    (github.com/settings → Zenodo integration)

# 4. README'ye DOI badge ekle
# [![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.XXXXXXX.svg)](...)
```

---

## Koda Erişim Beyanı (Paper'a Eklenmeli)

```latex
\section*{Code and Data Availability}
The source code, trained models, and experiment configurations 
are publicly available at \url{https://github.com/username/repo} 
(DOI: \url{https://doi.org/10.5281/zenodo.XXXXXXX}).
All experiments can be reproduced using:
\texttt{python scripts/reproduce\_all.py --paper <venue>-2026}.
```

---

## Gönderim Öncesi Son Kontrol

```bash
# Repo temizliği
git status  # Uncommitted dosya olmamalı
git log --oneline -10  # Geçmiş temiz mi?

# Büyük dosya kontrolü
find . -size +50M -not -path "./.git/*"

# Hassas veri kontrolü
git log --all -S "password" --oneline
git log --all -S "api_key" --oneline

# Sıfırdan kurulum testi (conda clean environment)
conda create -n release_test python=3.11 -y
conda activate release_test
pip install -r requirements.txt
python scripts/reproduce_all.py --quick  # 1 seed, hızlı doğrulama
```

---

## Arşivleme Platformları

Uzun süreli erişilebilirlik için:
- **Zenodo**: https://zenodo.org — DOI ataması yapıyor
- **OSF**: https://osf.io — Akademik araştırma odaklı
- **GitHub + Release**: Kod için; veri için LFS kullan
- **HuggingFace Hub**: ML modelleri ve veri setleri için

> ⚠️ GitHub URL'si tek başına yeterli değildir — silinebilir. Zenodo DOI oluşturun.

