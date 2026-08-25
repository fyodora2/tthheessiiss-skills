---
name: "research-debug"
description: "Use when debugging code in an academic research project — RL experiments, simulation pipelines, or anything whose output was or will be reported in a paper. Classifies the bug by research impact (isolated, result-altering, or methodology-breaking), decides whether already-reported results survive it, and constrains the fix so it cannot introduce data leakage or shift the hypothesis under test. Run this before any general debugging skill in research contexts."
---

# Research Debug Skill — Debugging in Research Contexts

## Core Rule
> Debugging in research code must answer two questions simultaneously:  
> (1) Is it technically correct?  
> (2) Does it invalidate previously reported academic claims or results?

**Ordering.** In a research context this skill runs *first*. It does not find the bug — it decides what finding the bug is allowed to cost. Once a bug is classified TYPE A (isolated), hand off to a general debugging skill for the mechanical fix. TYPE B and TYPE C never proceed to a fix without the steps below.

---

## Step 0 — Bug & Fix Classification (Before Any Code Edit)

Before making any code modifications or applying bug fixes, answer these 4 questions:

```
BUG & FIX INTEGRITY AUDIT
══════════════════════════════════════════════════════════

❓ 1. DOES THE FIX / BUG VIOLATE ACADEMIC INTEGRITY?
   • Does the fix introduce data leakage?
   • Are test set inputs bleeding into training/debug pipelines?
   • Is the bug caused by local dev machine vs. server/cluster
     (Windows vs. Linux, CUDA version, CPU cores) environment differences?
   • Are theoretical assumptions from cited papers being violated?
   → If YES: STOP. Academic integrity violation — resolve that first.

❓ 2. DOES THE FIX DISTORT EXPERIMENTAL GOALS? (Goal Distortion)
   • Does the fix change the research question being tested?
   • Does it alter baseline comparison equality?
   → If YES: STOP. Consult the user before changing code.

❓ 3. BUG IMPACT CLASSIFICATION
   ├── TYPE A (Isolated Software Bug): Does not affect experimental metrics
   │     (e.g. logging format, plot label, CLI typo).
   │     └─ Action: Fix safely, run unit tests. A general debugging skill can take it from here.
   │
   ├── TYPE B (Result-Altering Bug): Alters numerical outputs, rewards, or metrics
   │     of reported experiments.
   │     └─ Action: Mark previous run logs INVALID. Tag the commit `BUG-IMPACT: B`.
   │        Schedule a re-run before any affected number is used again.
   │
   └── TYPE C (Methodology-Breaking Bug): Violates paper pseudocode or the project's
         theoretical formulation.
         └─ Action: STOP IMMEDIATELY. Notify the user. Requires re-formulation
            and full re-evaluation.

❓ 4. EMPIRICAL EVIDENCE CHECK
   • Has the bug been verified with full log files, tracebacks, or pytest output?
   → If NO: STOP. No speculative debugging — empirical evidence is required
     before any code change.
```

---

## Step 1 — Blast Radius

For TYPE B and C, before fixing, enumerate what the bug touched:

- Which experiment runs used the buggy code path? (`git log` the file against run dates)
- Which figures, tables, or claims in the draft depend on those runs?
- Was anything already submitted, shared with an advisor, or published?

Write this down before the fix. After the fix it is much harder to reconstruct honestly.

---

## Step 2 — Fix Constraints

The fix must not:
- introduce train/test leakage in the name of making something work
- change the hypothesis being tested
- change baseline conditions asymmetrically (fixing your method but not the baselines)
- silently alter a random seed, evaluation protocol, or metric definition

If the only fix available does one of these, that is a research decision, not a code decision. Escalate.

---

## Note — the formulation registry

Where the project keeps a formulation registry — a document such as `FORMULATION.md` holding the canonical equations, symbols and parameter values — resolve it dynamically: look in `.claude/context/`, `.agents/context/`, then the project root. If no such file exists, ask the user which document is authoritative rather than assuming. Where it does exist, treat it as user-locked: if the code and the registry disagree, the code is what gets fixed.

