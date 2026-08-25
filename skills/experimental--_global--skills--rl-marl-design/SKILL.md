---
name: "rl-marl-design"
description: "Use when designing multi-agent RL environments or algorithms. Covers PettingZoo API compliance, centralized training with decentralized execution (CTDE), and the credit-assignment implications of choosing MAPPO, QMIX, or IPPO."
---

# MARL Design Skill — Multi-Agent Reinforcement Learning Architecture

## Core Rule
> Multi-Agent RL environments MUST comply with the PettingZoo API (`ParallelEnv` / `AECEnv`).  
> Multi-agent algorithms MUST follow Centralized Training with Decentralized Execution (CTDE) to avoid non-stationarity.

---

## 1. CTDE Architecture Pattern

```
[TRAINING PHASE - Centralized]
Joint States (S) + Joint Actions (A1, A2...) ──► Centralized Critic V(S) / Q(S, A1, A2)

[EXECUTION PHASE - Decentralized]
Local Obs (O_i) ──► Decentralized Policy Actor_i(a_i | O_i)
```

**Why it matters:** from any single agent's view, the other agents' changing policies make the environment non-stationary, which breaks the stationarity assumption every single-agent convergence result depends on. A centralized critic that sees the joint state restores stationarity during training, while decentralized actors keep execution deployable.

---

## 2. PettingZoo ParallelEnv Interface Template

```python
import numpy as np
from pettingzoo.utils.env import ParallelEnv
from gymnasium import spaces

class AcademicMARLEnv(ParallelEnv):
    """PettingZoo ParallelEnv template for multi-agent RL research."""
    metadata = {"name": "academic_marl_v0"}

    def __init__(self, num_agents=3):
        super().__init__()
        self.possible_agents = [f"agent_{i}" for i in range(num_agents)]
        self.agents = self.possible_agents[:]

        # Shared space definitions across agents
        self.observation_spaces = {
            agent: spaces.Box(low=-1.0, high=1.0, shape=(6,), dtype=np.float32)
            for agent in self.possible_agents
        }
        self.action_spaces = {
            agent: spaces.Discrete(4) for agent in self.possible_agents
        }

    def reset(self, seed=None, options=None):
        self.agents = self.possible_agents[:]
        observations = {agent: self._get_obs(agent) for agent in self.agents}
        infos = {agent: {} for agent in self.agents}
        return observations, infos

    def step(self, actions):
        # Actions is a dict mapping agent -> action
        rewards = {agent: self._compute_reward(agent, actions[agent]) for agent in self.agents}
        terminations = {agent: False for agent in self.agents}
        truncations = {agent: False for agent in self.agents}
        infos = {agent: {} for agent in self.agents}
        observations = {agent: self._get_obs(agent) for agent in self.agents}

        # Agents that are done must be removed from self.agents
        self.agents = [a for a in self.agents if not (terminations[a] or truncations[a])]

        return observations, rewards, terminations, truncations, infos
```

---

## 3. Algorithm Choice and Credit Assignment

| | IPPO | MAPPO | QMIX |
|---|---|---|---|
| Critic | Independent per agent | Centralized, shared | Mixing network over per-agent Q |
| Action space | Any | Any | Discrete only |
| Reward structure | Individual | Individual or shared | Shared team reward |
| Credit assignment | None — each agent optimises its own return | Centralized critic, but still no explicit decomposition | Monotonic mixing gives implicit decomposition |
| Reputation | Surprisingly strong baseline | Strong on cooperative benchmarks | Strong on SMAC-style discrete tasks |

**Do not skip IPPO as a baseline.** Independent learners are a stronger baseline than the MARL literature's framing suggests, and a proposed method that does not beat IPPO under an equal budget has not demonstrated that its multi-agent machinery is doing anything.

---

## 4. MARL Evaluation Notes

- **Report per-agent and team returns separately.** A high team return can hide one agent doing nothing.
- **Fix the agent count between comparisons.** Scaling agents changes the problem, not just the difficulty.
- **State the observability assumption explicitly** — full state, local observation, or communication-enabled. Results are not comparable across these.
- **Seed count matters more than in single-agent RL.** MARL variance across seeds is notoriously high; 5 seeds is a floor, not a target.

