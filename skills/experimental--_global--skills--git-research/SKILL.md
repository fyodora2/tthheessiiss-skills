---
name: "git-research"
description: "Use for Git inside an academic research repository: experiment commit tags (exp:, data:, result:), paper and paper-draft branches, keeping model weights and large artifacts out of history, and commit hooks that record experimental provenance. For general Git mechanics, use git-engineering."
---

# Git Research Skill — Academic Research Git Workflows

## Core Rule
> Model weights (.pt/.pth/.onnx) and raw datasets must NEVER be committed to Git.  
> Use a research `.gitignore` template and commit tags (`exp:`, `result:`, `paper:`) for research tracking.

For general Git mechanics — branching, rebase, bisect, history cleanup — use `git-engineering` instead.

---

## 1. Academic Commit Message Types

```
<type>(<scope>): <description>

BUG-IMPACT: [A / B / C / None]
FORMULATION-REF: [EQ-01 / None]
```

### Research Commit Types
- `exp`: Running or configuring an empirical experiment
- `result`: Updating experimental log tables, plots, or statistical metrics
- `paper`: Editing LaTeX sections, figures, or BibTeX files
- `model`: Model architecture adjustments (code only)
- `data`: Data pipeline or preprocessing script updates

### BUG-IMPACT Classification
Used when a commit fixes a bug in code that produced reported results:

- **A** — isolated software bug; no reported result changes
- **B** — result-altering; previously reported numbers are now wrong and must be regenerated
- **C** — methodology-breaking; the experimental design itself was invalid

B and C require assessing which existing results are invalidated *before* the fix is committed.

---

## 2. Large Artifact Protection

Make sure `.gitignore` contains:
```gitignore
# Exclude model weights and datasets
*.pt
*.pth
*.onnx
*.npz
*.ckpt
*.safetensors
data/
results/logs/
wandb/
*.db
```

Before staging, verify nothing large or ignored is sneaking in:
```bash
git check-ignore -v <path>          # is this file already ignored?
git status -u                       # untracked, excluding ignored
find . -size +50M -not -path "./.git/*"
```

---

## 3. Paper Branches

- `main` — code that reproduces the current results
- `paper-draft` — LaTeX source under active writing
- `paper/<venue>-<year>` — frozen at submission, never rebased. This is what a reviewer or a future you checks out to reproduce the submitted numbers.

Tag the exact commit a submission was built from:
```bash
git tag -a "submission/neurips-2026" -m "State of the repo at NeurIPS 2026 submission"
```

