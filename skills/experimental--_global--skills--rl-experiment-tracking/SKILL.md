---
name: "rl-experiment-tracking"
description: "Use when instrumenting RL training runs: WandB or MLflow integration with RLlib or SB3, logging training curves and evaluation returns, defining an evaluation protocol separate from training, and comparing policies across seeds for publication. For querying runs that already finished, use wandb-mlflow-api."
---

# RL Experiment Tracking Skill — RL Deney Takibi

## Temel Kural
> RL eğitimi stokastiktir. Tek bir seed'in sonucu hiçbir şeyi kanıtlamaz.

Bu skill **yeni koşuları enstrümante etmek** içindir. Biten koşuları çekip analiz etmek için `wandb-mlflow-api` kullan.

---

## RL'ye Özgü Tekrarlanabilirlik Gereksinimleri

RL deneyleri standart ML'den daha karmaşık rastgelelik kaynakları içerir:

```python
import random
import numpy as np
import torch
import gymnasium as gym

# 1. Python ve numpy seed
SEED = 42
random.seed(SEED)
np.random.seed(SEED)
torch.manual_seed(SEED)

# 2. Ortam seed (KRİTİK — çoğu zaman unutuluyor)
env = gym.make("MyEnv-v0")
obs, _ = env.reset(seed=SEED)  # Her sıfırlamada aynı başlangıç

# 3. RLlib için
config.debugging(seed=SEED)  # Tüm worker'lar için seed propagasyonu

# 4. SB3 için
model = PPO("MlpPolicy", env, seed=SEED, verbose=1)

# 5. Vektörel ortam için farklı seed'ler (determinizm ≠ hepsi aynı)
vec_env = make_vec_env("MyEnv-v0", n_envs=4, seed=SEED)
# Her ortam SEED+i seed'i alır: 42, 43, 44, 45
```

---

## Minimum Raporlama Standardı (Henderson et al., 2018)

> RL paperları için: En az **5 farklı seed** ile çalıştır, **ortalama ± std** raporla.

Önerilen seed seti: `[42, 123, 456, 789, 1024]`

```python
seeds = [42, 123, 456, 789, 1024]
results = {}

for seed in seeds:
    env = make_env(seed=seed)
    model = train_model(env, seed=seed, timesteps=1_000_000)

    # Eval: 100 episode, test ortamında
    eval_env = make_eval_env(seed=seed + 10000)  # Train seed'inden farklı
    episode_rewards = evaluate_policy(model, eval_env, n_eval_episodes=100)
    results[seed] = episode_rewards

# İstatistiksel raporlama
all_means = [np.mean(results[s]) for s in seeds]
print(f"Mean ± Std: {np.mean(all_means):.2f} ± {np.std(all_means):.2f}")
print(f"Min/Max: {min(all_means):.2f} / {max(all_means):.2f}")
```

---

## WandB Entegrasyonu

### RLlib ile
```python
import wandb
from ray.air.integrations.wandb import WandbLoggerCallback

config.callbacks(
    callbacks_class=[
        WandbLoggerCallback(
            project="my_rl_project",
            group="ppo_baseline",
            job_type="train",
            tags=["ppo", "custom_env", "seed42"],
        )
    ]
)
```

### SB3 ile
```python
from stable_baselines3.common.callbacks import BaseCallback
import wandb

wandb.init(
    project="my_rl_project",
    config={
        "algorithm": "PPO",
        "env": "CustomEnv-v1",
        "total_timesteps": 1_000_000,
        "seed": 42,
        "lr": 3e-4,
    },
    name=f"ppo_seed{SEED}",
)

class WandBCallback(BaseCallback):
    def _on_step(self) -> bool:
        if self.n_calls % 1000 == 0:
            wandb.log({
                "train/mean_reward": self.locals.get("mean_reward", 0),
                "train/timesteps": self.num_timesteps,
            })
        return True

model.learn(total_timesteps=1_000_000, callback=WandBCallback())
```

**Grup ve seed'i config'e yaz.** `group="ppo_baseline"` ve `config["seed"]` olmadan sonradan seed'lere göre toplama yapmak elle eşleştirmeye dönüşür.

---

## Değerlendirme Protokolü

```python
from stable_baselines3.common.evaluation import evaluate_policy
from scipy import stats
import numpy as np

def proper_evaluation(
    model,
    env_fn,
    n_eval_episodes: int = 100,
    eval_seed: int = 99999,  # Train seed'inden bağımsız
) -> dict:
    """Proper RL policy evaluation."""
    eval_env = env_fn(seed=eval_seed)

    episode_rewards, episode_lengths = evaluate_policy(
        model,
        eval_env,
        n_eval_episodes=n_eval_episodes,
        deterministic=True,  # Stokastik politika için False da dene
        return_episode_rewards=True,
    )

    mean = np.mean(episode_rewards)
    std = np.std(episode_rewards)
    ci = stats.t.interval(0.95, df=len(episode_rewards)-1, loc=mean, scale=stats.sem(episode_rewards))

    return {
        "mean_reward": mean,
        "std_reward": std,
        "ci_95": ci,
        "min_reward": min(episode_rewards),
        "max_reward": max(episode_rewards),
        "n_episodes": n_eval_episodes,
        "mean_episode_length": np.mean(episode_lengths),
    }
```

> **Değerlendirme ortamı eğitim ortamından ayrı seed almalı.** Aynı seed ile değerlendirmek, ezberlenmiş bir başlangıç dağılımında test etmektir.

---

## Öğrenme Eğrisi Raporlama

```python
import matplotlib.pyplot as plt
import numpy as np

def plot_learning_curves(results_per_seed: dict, title: str, window: int = 10) -> None:
    """Plot mean ± std learning curves across seeds."""
    fig, ax = plt.subplots(figsize=(10, 6))

    all_rewards = np.array(list(results_per_seed.values()))  # [n_seeds, n_steps]

    mean = all_rewards.mean(axis=0)
    std = all_rewards.std(axis=0)
    steps = np.arange(len(mean)) * 1000  # Her 1000 timestep

    # Yumuşatma
    mean_smooth = np.convolve(mean, np.ones(window)/window, mode='valid')

    ax.plot(steps[:len(mean_smooth)], mean_smooth, label="Mean", linewidth=2)
    ax.fill_between(
        steps[:len(mean_smooth)],
        mean_smooth - std[:len(mean_smooth)],
        mean_smooth + std[:len(mean_smooth)],
        alpha=0.2,
        label="±1 Std"
    )

    ax.set_xlabel("Timesteps")
    ax.set_ylabel("Episode Reward")
    ax.set_title(f"{title} (N={len(results_per_seed)} seeds)")
    ax.legend()
    ax.grid(True, alpha=0.3)

    plt.tight_layout()
    plt.savefig(f"{title.lower().replace(' ', '_')}.pdf", bbox_inches='tight')
```

Yumuşatma penceresini figür açıklamasında belirt — pencere boyutu eğrinin görünümünü ciddi biçimde değiştirir ve belirtilmezse karşılaştırma yanıltıcı olur.

---

## RL Deney Raporlama Şablonu

```
RL Deney Raporu
════════════════════════════════════════
Algoritma:      PPO
Ortam:          CustomEnv-v1 (açıklama)
Framework:      Ray RLlib 2.X
Seed'ler:       [42, 123, 456, 789, 1024]
Total Steps:    1,000,000 / seed
Donanım:        NVIDIA RTX 4090

Hiperparametreler:
  lr:           3e-4
  gamma:        0.99
  clip_param:   0.2

Değerlendirme: 100 episode / seed, deterministic policy

Sonuçlar:
  Mean Reward:   XXX.X ± XX.X  (mean ± std, N=5 seed)
  %95 GA:        [XXX.X, XXX.X]
  Min/Max seed:  XXX.X / XXX.X
  Best Seed:     42 (XXX.X)

Baseline:
  Random Policy: XX.X ± X.X
  Expert:        XXX.X (opsiyonel)

İstatistiksel Anlamlılık:
  vs Baseline: t=X.XX, p=0.00X (paired t-test, N=5)
```

> "Best Seed" raporlanabilir ama **asla ana sonuç olarak sunulmaz**. En iyi seed'i seçip onu raporlamak, seed üzerinden yapılmış gizli bir hiperparametre aramasıdır.

