---
name: "statistical-validity"
description: "Use when interpreting numerical results, comparing a method against baselines, or reporting a performance difference. Covers confidence intervals, choosing between Welch's t-test and Mann-Whitney U, Cohen's d effect sizes, multiple-comparison correction, and the leakage and selection-bias checks that must pass before a difference is claimed."
---

# Statistical Validity Skill — Statistical Testing & Effect Size Analysis

## Core Rule
> Raw mean values without confidence intervals or p-values are statistically meaningless.

---

## 1. Statistical Analysis Pipeline

```python
import numpy as np
from scipy import stats

def compute_academic_stats(sample_a: list, sample_b: list):
    """Compute 95% Confidence Intervals, Welch's t-test, and Cohen's d."""
    a, b = np.array(sample_a), np.array(sample_b)

    # 1. 95% Confidence Interval
    ci_a = stats.t.interval(0.95, len(a)-1, loc=np.mean(a), scale=stats.sem(a))
    ci_b = stats.t.interval(0.95, len(b)-1, loc=np.mean(b), scale=stats.sem(b))

    # 2. Welch's t-test (unequal variances)
    t_stat, p_val = stats.ttest_ind(a, b, equal_var=False)

    # 3. Cohen's d Effect Size
    pooled_std = np.sqrt((np.std(a, ddof=1)**2 + np.std(b, ddof=1)**2) / 2)
    cohens_d = (np.mean(a) - np.mean(b)) / pooled_std

    return {
        "mean_a": np.mean(a), "ci_95_a": ci_a,
        "mean_b": np.mean(b), "ci_95_b": ci_b,
        "p_value": p_val, "cohens_d": cohens_d
    }
```

---

## 2. Choosing the Right Test

- **Welch's t-test** — default for comparing two independent samples with possibly unequal variances. Do not assume equal variance without checking.
- **Mann-Whitney U** — when the distribution is clearly non-normal or the sample is small and skewed (common with RL returns).
- **Multiple comparisons** — when comparing against several baselines, correct the family-wise error rate (Bonferroni for a few comparisons, Benjamini-Hochberg for many). An uncorrected p < 0.05 across ten comparisons means nothing.
- **Effect size is not optional** — a significant p-value with a negligible Cohen's d is a statement about sample size, not about the method.

---

## 3. Selection Bias and Leakage Checks

Before any statistical claim is made, confirm:

1. **Selection bias** — the dataset or environment distribution matches the deployment scenario the claim is about. State the mismatch explicitly if it does not.
2. **Data leakage** — preprocessing statistics (scaling mean/std, normalisation constants, vocabulary) are computed strictly on the training fold and applied to validation and test, never fitted on the full set.
3. **Evaluation bias** — the evaluation protocol was fixed before results were seen, not chosen after inspecting them.

A test that passes on leaked data proves nothing; resolve these before computing p-values.

