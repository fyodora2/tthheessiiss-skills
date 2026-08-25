---
name: "latex-bibtex-manager"
description: "Use when writing or editing LaTeX (.tex), building academic tables, managing BibTeX (.bib) entries, or diagnosing pdflatex/latexmk compilation errors. Covers NeurIPS, ICML, ICLR, IEEE, and ACM templates."
---

# LaTeX & BibTeX Manager Skill — Akademik Dizgi ve Kaynakça

## Temel Kural
> LaTeX belgesi ve BibTeX kaynakçası sıfır hata ve sıfır çakışma ile derlenmelidir.  
> Tüm atıflar `academic-integrity` kurallarına uygun olarak doğrulanmalıdır.

---

## 1. Şablon Yapısı (NeurIPS / ICML / IEEE)

```latex
\documentclass{article}

% Şablon Paketleri
\usepackage[final]{neurips_2026}
\usepackage[utf8]{inputenc}
\usepackage{microtype}
\usepackage{graphicx}
\usepackage{booktabs}   % Yayın kalitesinde tablolar (top/mid/bottomrule)
\usepackage{amsmath,amssymb,amsfonts}
\usepackage{hyperref}

\title{<Paper Title>}
\author{Author 1 \quad Author 2 \\ Department / Institution}

\begin{document}
\maketitle

\begin{abstract}
<Abstract text here>
\end{abstract}

\section{Introduction}
\label{sec:intro}

\section{Methodology}
\label{sec:method}

\section{Experiments}
\label{sec:experiments}

% Dışarıdan tablo dahil etme
\input{figures/table_results.tex}

\section{Conclusion}
\label{sec:conclusion}

\bibliographystyle{plainnat}
\bibliography{references}

\end{document}
```

---

## 2. BibTeX Kaynakça Yönetimi (`references.bib`)

```bibtex
@inproceedings{schulman2017proximal,
  title     = {Proximal policy optimization algorithms},
  author    = {Schulman, John and Wolski, Filip and Dhariwal, Prafulla and Radford, Alec and Klimov, Oleg},
  booktitle = {arXiv preprint arXiv:1707.06347},
  year      = {2017}
}
```

### BibTeX Temizlik Kontrol Listesi
- [ ] Mükerrer anahtarlar (duplicate keys) yok.
- [ ] Yazar isimleri `Lastname, Firstname` veya `Firstname Lastname` kıvamında tutarlı.
- [ ] Dergi/Konferans isimleri kısaltılmamış veya standart kısaltmada.
- [ ] arXiv makaleleri için `eprint` ve `archivePrefix` alanları eksiksiz.

---

## 3. Otomatik Derleme ve Hata Ayıklama Scripti

```bash
# LaTeX Derleme (pdflatex / bibtex / pdflatex x2)
latexmk -pdf -halt-on-error main.tex

# Temizlik (geçici aux, log, bbl dosyaları)
latexmk -c
```

---

## 4. Sık Görülen Derleme Hataları

| Hata | Sebep | Çözüm |
|---|---|---|
| `Undefined control sequence` | Eksik `\usepackage` | Hatanın verdiği satırdaki komutun paketini ekle |
| `Citation ... undefined` | `.bib` içinde anahtar yok ya da bibtex çalışmadı | Anahtarı kontrol et, `latexmk` ile tam döngü çalıştır |
| `Overfull \hbox` | Satır taşması, genelde uzun URL veya tablo | `\usepackage{microtype}`, tabloyu `\resizebox` ile daralt |
| `File ... not found` | Şablon `.sty` dosyası proje kökünde değil | Konferans şablonunu proje köküne kopyala |
| `Missing $ inserted` | Metin modunda matematik sembolü | İlgili sembolü `$...$` içine al |

