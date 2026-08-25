---
name: "rl-reward-shaping"
description: "Use when designing or changing a reward function in RL. Enforces potential-based shaping so the optimal policy is provably preserved, and checks for reward hacking and perverse incentives before the reward reaches training."
---

# RL Reward Shaping Skill — Reward Design & Anti-Hacking

## Core Rule
> Reward shaping must be potential-based ($\Phi(s)$) to guarantee optimal policy invariance.  
> Unbounded or unnormalized rewards cause value function explosion and reward hacking.

---

## 1. Potential-Based Reward Shaping Formula

$$\mathcal{R}'(s, a, s') = \mathcal{R}(s, a, s') + \gamma \Phi(s') - \Phi(s)$$

- $\mathcal{R}$: Original sparse ground-truth reward
- $\Phi(s)$: State potential function (e.g. negative distance to goal)
- $\gamma$: Discount factor

Ng, Harada & Russell (1999) proved that shaping of exactly this form leaves the optimal policy unchanged. Any shaping term that is **not** expressible as $\gamma\Phi(s') - \Phi(s)$ can change which policy is optimal — that is a change to the research question, not a training convenience.

### Anti-Reward Hacking Checklist
- [ ] **No Infinite Loop Exploit**: Agent cannot collect infinite positive reward by cycling between two states.
- [ ] **Survival Incentive Alignment**: In penalty-per-step environments, agent cannot commit immediate suicide to minimize cumulative loss.
- [ ] **Reward Bounding**: Output is normalized or bounded to $[-1.0, +1.0]$ or $[0, 1]$.
- [ ] **Shaping Is Reported**: Any shaping used in the experiments is stated in the paper. Unreported shaping makes a baseline comparison unfair.

