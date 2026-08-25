---
name: "rl-environment-design"
description: "Use when building or modifying a Gymnasium or PettingZoo environment. Covers observation and action space definition, the reset/step API contract, seeding and determinism, termination versus truncation, and wrapper composition."
---

# RL Environment Design Skill — Gymnasium Environment Contracts

## Core Rule
> RL environments MUST strictly adhere to Gymnasium standards (`gymnasium.Env`).  
> Non-standard APIs break RLlib, Stable-Baselines3, and Ray Tune integration.

---

## 1. Gymnasium Standard Interface Template

```python
import gymnasium as gym
from gymnasium import spaces
import numpy as np

class AcademicRLEnv(gym.Env):
    """Standard Gymnasium environment for RL research."""
    metadata = {"render_modes": ["human", "rgb_array"], "render_fps": 30}

    def __init__(self, render_mode=None):
        super().__init__()
        self.render_mode = render_mode

        # 1. Observation & Action Space Definitions
        self.observation_space = spaces.Box(low=-1.0, high=1.0, shape=(8,), dtype=np.float32)
        self.action_space = spaces.Box(low=-1.0, high=1.0, shape=(2,), dtype=np.float32)

    def reset(self, seed=None, options=None):
        super().reset(seed=seed)
        # Seed initialization for reproducibility
        if seed is not None:
            self.np_random, _ = gym.utils.seeding.np_random(seed)

        obs = self._get_obs()
        info = self._get_info()
        return obs, info

    def step(self, action):
        # Action bounds check
        action = np.clip(action, self.action_space.low, self.action_space.high)

        # State transition
        self._update_state(action)

        obs = self._get_obs()
        reward = self._compute_reward()
        terminated = self._check_terminated()
        truncated = self._check_truncated()
        info = self._get_info()

        return obs, reward, terminated, truncated, info
```

---

## 2. Terminated vs Truncated

These are not interchangeable and conflating them corrupts value bootstrapping.

- **`terminated=True`** — the episode ended because the MDP says so (goal reached, agent died, absorbing state). The value of the next state is genuinely 0, so the learner must NOT bootstrap.
- **`truncated=True`** — the episode was cut off externally (time limit, step budget). The next state still has value, so the learner MUST bootstrap.

Returning `terminated=True` for a time limit silently biases every value estimate downward. This is one of the most common and hardest-to-spot RL bugs.

---

## 3. Environment Contract Checklist

- [ ] `reset()` accepts `seed` and `options`, calls `super().reset(seed=seed)`, returns `(obs, info)`
- [ ] `step()` returns the 5-tuple `(obs, reward, terminated, truncated, info)`
- [ ] Every returned observation is inside `observation_space` — verify with `observation_space.contains(obs)`
- [ ] `dtype` matches the space exactly (`np.float32` mismatch silently upcasts and breaks some backends)
- [ ] Two `reset(seed=42)` calls produce identical trajectories under identical actions
- [ ] Passes `gymnasium.utils.env_checker.check_env(env)`
- [ ] Wrappers applied in a documented order — normalisation before or after frame-stacking is not the same experiment

