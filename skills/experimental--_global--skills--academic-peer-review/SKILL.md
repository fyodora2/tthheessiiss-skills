---
name: "academic-peer-review"
description: "Use when reviewing an academic paper draft, methodology section, or experimental design the way a conference referee would. Produces a NeurIPS/ICML/ICLR/IEEE-style review scoring soundness, originality, clarity, and significance, with an explicit recommendation. Not for copy-editing or prose polish."
---

# Academic Peer-Review Skill — Top-Tier Conference Review Simulation

## Core Rule
> Review papers as a rigorous, unbiased top-tier reviewer (NeurIPS/ICML/IEEE).  
> Identify methodology flaws, unbacked claims, missing baselines, and clarity gaps before submission.

---

## 1. Peer Review Evaluation Axes

1. **Soundness (1-5)**: Are theoretical proofs correct? Are experiments reproducible and statistically valid?
2. **Originality (1-5)**: Is the contribution novel compared to prior literature?
3. **Clarity (1-5)**: Is the writing clear, well-structured, and mathematically precise?
4. **Significance (1-5)**: Does the paper advance the state-of-the-art or solve a key research gap?

---

## 2. Review Report Template

```markdown
# 📝 Academic Peer-Review Report

## Summary
Brief summary of the paper's core contributions and methodology.

## Strengths
- S1: High empirical rigor with 5-seed confidence intervals.
- S2: Formal mathematical formulation.

## Weaknesses & Critical Questions
- W1: Missing baseline comparison with [Method X].
- W2: Hyperparameter sensitivity not plotted for learning rate.

## Recommendation
- [ ] Strong Accept (8)
- [ ] Accept (7)
- [ ] Weak Accept (6)
- [ ] Borderline Reject (5)
- [ ] Reject (3-4)

## Requested Revisions
1. Add 95% Confidence Interval error bars to Figure 3.
2. Clarify assumptions behind Equation 4 in Section 3.2.
```

