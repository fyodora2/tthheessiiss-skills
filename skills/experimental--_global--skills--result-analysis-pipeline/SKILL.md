---
name: "result-analysis-pipeline"
description: "Use when turning raw experiment logs into reportable output: aggregating runs into statistics across seeds, generating publication-quality vector figures (PDF), and emitting LaTeX table fragments ready to include in a paper."
---

# Result Analysis Pipeline Skill — Log-to-LaTeX Table & Figure Pipeline

## Core Rule
> Manual copy-pasting of experiment results into paper tables is forbidden.  
> Raw log parsing, statistical computation, PDF figure plotting, and TeX table generation must be automated.

A number typed by hand into a table is a number nobody can reproduce.

---

## 1. Automated Analysis Pipeline Flow

```
[Raw Log Files (JSON/CSV/WandB)]
  │
  ├── 1. Parse Logs & Extract Multi-Seed Runs
  ├── 2. Compute Mean, 95% Confidence Intervals, & Welch's t-test  → see statistical-validity
  ├── 3. Plot Colorblind-Friendly PDF Figures (Seaborn / Matplotlib)
  └── 4. Export Publication LaTeX Table Fragment (`results/table_summary.tex`)
```

---

## 2. Publication-Ready LaTeX Table Generator

```python
import pandas as pd

def generate_latex_table(df_stats: pd.DataFrame, output_path: str):
    """Generate a clean booktabs LaTeX table from statistical DataFrame."""
    latex_str = df_stats.to_latex(
        index=False,
        column_format="lcccc",
        caption="Empirical benchmark performance across 5 random seeds.",
        label="tab:main_results",
        escape=False
    )
    with open(output_path, "w") as f:
        f.write(latex_str)
```

---

## 3. Figure Standards

- **Vector output only** — save as PDF, never PNG. A rasterised figure in a camera-ready is a desk-reject risk at some venues.
- **Colorblind-safe palette** — `seaborn.color_palette("colorblind")`. Never encode a distinction by colour alone; pair it with linestyle or marker.
- **Shaded confidence bands, not raw seed spaghetti** — plot the mean with a 95% CI band across seeds.
- **Font size matched to the paper** — set `matplotlib` font size so the figure is legible at final column width without scaling.
- **Axis labels carry units** — "Return" is not a label; "Episodic Return" with a step-count x-axis is.

```python
import matplotlib.pyplot as plt
plt.rcParams.update({
    "figure.figsize": (3.5, 2.5),   # single column
    "font.size": 9,
    "pdf.fonttype": 42,              # embed Type 42 fonts, required by some venues
    "savefig.bbox": "tight",
})
```

---

## 4. Regeneration Rule

The pipeline must be re-runnable end to end with one command, so that a change in the logs propagates to every table and figure in the paper without manual intervention.

```bash
python scripts/analyze_results.py --results_dir results/ --out paper/figures/
```

