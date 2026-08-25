---
name: "rl-offline-dataset"
description: "Use when training RL on pre-collected static data with no environment interaction. Covers Conservative Q-Learning (CQL), Implicit Q-Learning (IQL), D4RL dataset formats, and auditing distribution shift between the behavior policy and the learned policy."
---

# Offline RL Skill — Static Dataset Training & CQL/IQL Algorithms

## Core Rule
> Offline RL MUST account for Out-Of-Distribution (OOD) action extrapolation errors.  
> Standard Q-learning fails offline due to overestimation bias on unseen actions; CQL or IQL penalty terms are required.

---

## 1. Conservative Q-Learning (CQL) Loss Objective

$$\min_Q \alpha \cdot \mathbb{E}_{s \sim \mathcal{D}} \left[ \log \sum_a \exp(Q(s, a)) - \mathbb{E}_{a \sim \mathcal{D}}[Q(s, a)] \right] + \frac{1}{2} \mathbb{E}_{(s, a, s') \sim \mathcal{D}} \left[ \left( Q(s, a) - (r + \gamma \max_{a'} \hat{Q}(s', a')) \right)^2 \right]$$

- $\mathcal{D}$: Static offline dataset (e.g. D4RL trajectories)
- $\alpha$: Conservative penalty trade-off weight

---

## 2. Offline Dataset Processing Template (D4RL Format)

```python
import numpy as np
import torch

class OfflineReplayBuffer:
    """Offline RL Replay Buffer reading static D4RL or custom datasets."""
    def __init__(self, dataset_dict: dict):
        self.observations = torch.FloatTensor(dataset_dict["observations"])
        self.actions = torch.FloatTensor(dataset_dict["actions"])
        self.next_observations = torch.FloatTensor(dataset_dict["next_observations"])
        self.rewards = torch.FloatTensor(dataset_dict["rewards"])
        self.terminals = torch.FloatTensor(dataset_dict["terminals"])
        self.size = len(self.observations)

    def sample(self, batch_size: int):
        indices = np.random.randint(0, self.size, size=batch_size)
        return (
            self.observations[indices],
            self.actions[indices],
            self.next_observations[indices],
            self.rewards[indices],
            self.terminals[indices],
        )
```

---

## 3. CQL vs IQL — Choosing

| | CQL | IQL |
|---|---|---|
| Mechanism | Penalises Q-values on OOD actions | Never queries OOD actions at all (expectile regression) |
| Hyperparameter sensitivity | High — `α` needs tuning per dataset | Lower — more robust out of the box |
| Compute | Heavier (logsumexp over actions) | Lighter |
| Good default when | Dataset is narrow and you need strong conservatism | You want a solid baseline with minimal tuning |

---

## 4. Distribution Shift Audit

Before trusting any offline result:

- [ ] **Behaviour policy coverage** — plot the action distribution in $\mathcal{D}$ against the learned policy's actions. Non-overlapping support means the reported return is extrapolation, not evaluation.
- [ ] **Dataset provenance recorded** — which policy generated it, at what skill level (`medium`, `expert`, `medium-replay`). Results are not comparable across dataset variants.
- [ ] **No online tuning leak** — hyperparameters selected using online rollouts invalidate the offline claim. Offline model selection must use offline criteria.
- [ ] **Normalised scores reported** — for D4RL, report the normalised score, not raw return, so numbers are comparable to published baselines.

