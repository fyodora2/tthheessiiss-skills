---
name: "ray-rllib"
description: "Use when working with Ray, RLlib, Ray Tune, or Ray Data: configuring RLlib algorithms, registering custom models and environments, defining Tune search spaces and schedulers, and scaling rollouts across workers."
---

# Ray RLlib Skill — Dağıtık Pekiştirmeli Öğrenme

## Temel Kural
> Ray'in gücü tek makine çok çekirdek → çok makine dağıtık geçişini şeffaf yapmasından gelir. Ama yanlış konfigürasyon, dağıtıktan fayda yerine overhead üretir.

---

## Hızlı Başlangıç — RLlib ile PPO

```python
import ray
from ray.rllib.algorithms.ppo import PPOConfig
from ray.tune.registry import register_env

# 1. Ortamı kaydet
register_env("my_env", lambda config: MyCustomEnv(**config))

# 2. Ray'i başlat
ray.init()

# 3. Algoritma konfigürasyonu
config = (
    PPOConfig()
    .environment(
        env="my_env",
        env_config={},         # Ortam yapılandırması
    )
    .env_runners(
        num_env_runners=4,     # Paralel worker sayısı
        num_envs_per_env_runner=1,
    )
    .training(
        lr=3e-4,
        gamma=0.99,
        lambda_=0.95,          # GAE lambda
        clip_param=0.2,        # PPO clip
        train_batch_size=4096,
        minibatch_size=128,
        num_epochs=10,
    )
    .resources(
        num_gpus=1,            # Trainer GPU'su
    )
    .framework("torch")
    .debugging(seed=42)        # Tekrarlanabilirlik
)

# 4. Eğitim
algo = config.build()

for i in range(100):
    result = algo.train()
    print(f"Iter {i}: reward={result['env_runners']['episode_return_mean']:.2f}")

    if i % 10 == 0:
        checkpoint = algo.save()
        print(f"Checkpoint: {checkpoint}")

ray.shutdown()
```

> RLlib API'si sürümler arası hızlı değişiyor (`num_workers` → `num_env_runners`, `sgd_minibatch_size` → `minibatch_size`, `episode_reward_mean` → `episode_return_mean`). Kod yazmadan önce kurulu sürümü kontrol et: `python -c "import ray; print(ray.__version__)"`.

---

## Worker Sayısı Seçimi

`num_env_runners` doğrudan CPU çekirdek sayısına eşitlenmez:

- Toplam kullanılan CPU ≈ `num_env_runners × num_cpus_per_env_runner + 1` (trainer için)
- Ortam adımı ucuzsa (basit dinamik) çok worker overhead getirir; `num_envs_per_env_runner` ile vektörleştirmek daha verimli
- Ortam adımı pahalıysa (simülatör, fizik motoru) worker sayısını artırmak gerçekten ölçekler
- Ölçmeden ayarlama: `result["env_runners"]["num_env_steps_sampled_lifetime"]` ile saniyedeki adım sayısını kıyasla

---

## Ray Tune — Hyperparameter Search

```python
from ray import tune
from ray.tune.schedulers import ASHAScheduler
from ray.tune.search.optuna import OptunaSearch

def train_rl(config):
    """Trainable function for Ray Tune."""
    algo = (
        PPOConfig()
        .environment(env="my_env")
        .training(
            lr=config["lr"],
            gamma=config["gamma"],
            clip_param=config["clip_param"],
        )
        .build()
    )

    for _ in range(50):
        result = algo.train()
        tune.report({"reward": result["env_runners"]["episode_return_mean"]})

# Search space
search_space = {
    "lr": tune.loguniform(1e-5, 1e-3),
    "gamma": tune.uniform(0.95, 0.999),
    "clip_param": tune.uniform(0.1, 0.3),
}

searcher = OptunaSearch(metric="reward", mode="max")
scheduler = ASHAScheduler(metric="reward", mode="max", grace_period=10)

analysis = tune.run(
    train_rl,
    config=search_space,
    num_samples=50,
    search_alg=searcher,
    scheduler=scheduler,
    resources_per_trial={"cpu": 4, "gpu": 0.5},
    storage_path="./ray_results",
    name="ppo_hyperparam_search",
)

best_config = analysis.get_best_config(metric="reward", mode="max")
print("En iyi konfigürasyon:", best_config)
```

> **Akademik uyarı:** Tune ile bulunan en iyi konfigürasyon, aynı bütçe baseline'lara da verilmediyse adil karşılaştırma değildir. Arama bütçesini (deneme sayısı, adım sayısı) makalede raporla.

---

## Custom Model Kaydı

```python
from ray.rllib.models import ModelCatalog
from ray.rllib.models.torch.torch_modelv2 import TorchModelV2
import torch
import torch.nn as nn

class CustomNetwork(TorchModelV2, nn.Module):
    """Custom neural network for RLlib."""

    def __init__(self, obs_space, action_space, num_outputs, model_config, name):
        TorchModelV2.__init__(self, obs_space, action_space, num_outputs, model_config, name)
        nn.Module.__init__(self)

        self.network = nn.Sequential(
            nn.Linear(obs_space.shape[0], 256),
            nn.ReLU(),
            nn.Linear(256, 256),
            nn.ReLU(),
        )
        self.policy_head = nn.Linear(256, num_outputs)
        self.value_head = nn.Linear(256, 1)
        self._value = None

    def forward(self, input_dict, state, seq_lens):
        features = self.network(input_dict["obs"].float())
        self._value = self.value_head(features)
        return self.policy_head(features), state

    def value_function(self):
        return self._value.squeeze(1)

ModelCatalog.register_custom_model("custom_net", CustomNetwork)

config.training(
    model={"custom_model": "custom_net", "custom_model_config": {}}
)
```

---

## Ray Cluster Yapılandırması

```yaml
# cluster.yaml
cluster_name: rl_training

provider:
  type: aws
  region: us-east-1

head_node_type:
  instance_type: g4dn.xlarge  # GPU'lu head node
  resources:
    CPU: 4
    GPU: 1

worker_node_types:
  - node_type_name: cpu_worker
    resources:
      CPU: 8
    min_workers: 2
    max_workers: 10
```

```bash
ray up cluster.yaml          # Cluster başlat
ray submit cluster.yaml train.py
ray dashboard cluster.yaml   # http://localhost:8265
```

> Dashboard portunu genel ağa açma. Kimlik doğrulaması yoktur; açık bir Ray portu uzaktan iş çalıştırmaya izin verir.

---

## Kontrol Listesi

- [ ] Ortam `register_env()` ile kaydedildi mi?
- [ ] `seed` konfigürasyona eklendi mi (`.debugging(seed=...)`)?
- [ ] Worker sayısı ölçülerek mi ayarlandı, tahminle mi?
- [ ] Checkpoint sıklığı yeterli mi?
- [ ] Tune arama bütçesi baseline'lara da verildi mi?
- [ ] Sonuçlar WandB veya MLflow ile loglanıyor mu?
- [ ] Uzun eğitimlerde bellek izlendi mi?
- [ ] Kurulu Ray sürümünün API'si doğrulandı mı?

---

## Desteklenen Algoritmalar (Özet)

| Algoritma | Action Space | Kullanım |
|---|---|---|
| PPO | Discrete + Continuous | Genel amaçlı, kararlı |
| SAC | Continuous | Sample efficient, off-policy |
| DQN | Discrete | Basit discrete problemler |
| IMPALA | Discrete + Continuous | Çok büyük ölçek, asenkron |
| APPO | Discrete + Continuous | Asenkron PPO, büyük ölçek |

