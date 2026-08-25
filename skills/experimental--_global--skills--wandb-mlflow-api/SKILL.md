---
name: "wandb-mlflow-api"
description: "Use when programmatically querying WandB or MLflow to pull run histories, metrics, and configs into Pandas DataFrames for statistical analysis or plotting. For instrumenting new runs, use rl-experiment-tracking."
---

# WandB & MLflow API Skill — Programatik Deney Sorgulama

## Temel Kural
> Manuel ekran görüntüsü veya WandB UI kopyalama yok.  
> Tüm deney verileri WandB/MLflow API ile programatik olarak çekilmeli ve saklanmalıdır.

Bu skill **biten koşuları sorgulamak** içindir. Yeni koşuları enstrümante etmek için `rl-experiment-tracking` kullan.

---

## 1. WandB API ile Run Çekme

```python
import wandb
import pandas as pd
from pathlib import Path

def fetch_wandb_runs(entity: str, project: str) -> pd.DataFrame:
    """Fetch all completed runs from a WandB project into a DataFrame."""
    api = wandb.Api()
    runs = api.runs(f"{entity}/{project}")

    summary_list = []
    config_list = []
    name_list = []

    for run in runs:
        if run.state == "finished":
            # Summary metrics
            summary_list.append(run.summary._json_dict)
            # Config / Hyperparameters
            config_list.append({k: v for k, v in run.config.items() if not k.startswith("_")})
            name_list.append(run.name)

    df_summary = pd.DataFrame(summary_list)
    df_config = pd.DataFrame(config_list)
    df_summary["run_name"] = name_list

    df_full = pd.concat([df_summary, df_config], axis=1)
    return df_full
```

### Tam Geçmiş (Öğrenme Eğrisi) Çekme

`run.summary` sadece son değeri verir. Eğri için geçmiş gerekir:

```python
def fetch_run_history(entity: str, project: str, metric: str = "train/mean_reward") -> pd.DataFrame:
    """Pull the full step-by-step history for every finished run."""
    api = wandb.Api()
    frames = []
    for run in api.runs(f"{entity}/{project}"):
        if run.state != "finished":
            continue
        # scan_history streams without the 500-row sampling of run.history()
        rows = list(run.scan_history(keys=["_step", metric]))
        df = pd.DataFrame(rows)
        df["run_name"] = run.name
        df["seed"] = run.config.get("seed")
        frames.append(df)
    return pd.concat(frames, ignore_index=True)
```

> `run.history()` downsamples to ~500 points by default. For publication figures use `scan_history`, otherwise the curve you plot is not the curve you trained.

---

## 2. MLflow Tracking API ile Metric Çekme

```python
import mlflow
import pandas as pd

def fetch_mlflow_experiment(experiment_name: str) -> pd.DataFrame:
    """Fetch runs from an MLflow experiment."""
    exp = mlflow.get_experiment_by_name(experiment_name)
    if not exp:
        raise ValueError(f"Experiment {experiment_name} not found")

    runs = mlflow.search_runs(experiment_ids=[exp.experiment_id])
    return runs


def fetch_mlflow_metric_history(run_id: str, key: str) -> pd.DataFrame:
    """Full metric history for one MLflow run."""
    client = mlflow.tracking.MlflowClient()
    history = client.get_metric_history(run_id, key)
    return pd.DataFrame([{"step": m.step, "value": m.value, "timestamp": m.timestamp} for m in history])
```

---

## 3. Seed'lere Göre Toplama

```python
def aggregate_over_seeds(df: pd.DataFrame, metric: str, group_cols: list[str]) -> pd.DataFrame:
    """Collapse per-seed runs into mean/std/count for reporting."""
    return (
        df.groupby(group_cols)[metric]
          .agg(["mean", "std", "count", "min", "max"])
          .reset_index()
          .rename(columns={"count": "n_seeds"})
    )
```

Çektiğin veriyi diske yaz (`results/runs.parquet`) — API'yi her analiz için tekrar çağırmak hem yavaş hem de sonuçları sessizce değiştirebilir (koşular silinebilir, yeniden adlandırılabilir).

