---
name: "fair-comparison"
description: "Use when benchmarking a proposed method against baselines. Enforces equal computational budgets (environment steps, GPU hours), equal hyperparameter tuning effort per method, and one standardized evaluation protocol, so the comparison actually supports the claim being made."
---

# Fair Comparison Skill — Baseline Evaluation Protocol

## Core Rule
> Baselines must never be evaluated under intentionally weak settings or unequal resource budgets.

---

## 1. Equal Budget Protocol

1. **Environmental Steps & Epochs**: The proposed method and all baseline algorithms must be trained for the EXACT same number of environment interaction steps (e.g., 1M steps) or training epochs.
2. **Hyperparameter Tuning Parity**: Do NOT use default hyperparameters for baselines while extensively tuning your proposed method. Either tune all methods equally or use published optimal baseline parameters.
3. **Identical Evaluation Environments**: Evaluate all algorithms on identical seeds and observation/action wrappers.

