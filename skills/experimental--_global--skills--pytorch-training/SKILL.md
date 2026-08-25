---
name: "pytorch-training"
description: "Use when writing a PyTorch training loop, designing a network architecture, diagnosing training instability (NaNs, exploding gradients, a model that will not learn), or reducing GPU memory usage. Covers loop structure, gradient management, torch.amp mixed precision, checkpointing, and determinism."
---

# PyTorch Training Skill — PyTorch Eğitim Döngüleri

## Temel Kural
> PyTorch sizi korumaz — `optimizer.zero_grad()` unutmak gradient birikimine yol açar; bunu bilinçli olarak kullananlar dışında her batch başında çağırın.

---

## Standart Eğitim Döngüsü Şablonu

```python
import torch
import torch.nn as nn
from torch.utils.data import DataLoader
from torch.amp import GradScaler, autocast   # güncel API: torch.amp
from pathlib import Path
import logging

logger = logging.getLogger(__name__)


def train_epoch(
    model: nn.Module,
    loader: DataLoader,
    optimizer: torch.optim.Optimizer,
    criterion: nn.Module,
    device: torch.device,
    scaler: GradScaler | None = None,
) -> dict[str, float]:
    """Single training epoch."""
    model.train()
    total_loss = 0.0
    n_batches = 0

    for batch_idx, (inputs, targets) in enumerate(loader):
        inputs = inputs.to(device, non_blocking=True)   # non_blocking: pin_memory ile hızlanma
        targets = targets.to(device, non_blocking=True)

        # Gradient sıfırla — HER batch öncesi
        optimizer.zero_grad(set_to_none=True)  # set_to_none=True daha hızlı

        if scaler:
            # Mixed precision — hız ve GPU bellek tasarrufu
            with autocast(device_type="cuda"):
                outputs = model(inputs)
                loss = criterion(outputs, targets)
            scaler.scale(loss).backward()
            scaler.unscale_(optimizer)          # clip'ten ÖNCE unscale şart
            torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
            scaler.step(optimizer)
            scaler.update()
        else:
            outputs = model(inputs)
            loss = criterion(outputs, targets)
            loss.backward()
            torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
            optimizer.step()

        total_loss += loss.item()
        n_batches += 1

        if batch_idx % 100 == 0:
            logger.debug("Batch %d/%d | Loss: %.4f", batch_idx, len(loader), loss.item())

    return {"loss": total_loss / n_batches}


@torch.no_grad()
def validate(
    model: nn.Module,
    loader: DataLoader,
    criterion: nn.Module,
    device: torch.device,
) -> dict[str, float]:
    """Validation loop."""
    model.eval()  # Dropout ve BatchNorm'u eval moduna al
    total_loss = 0.0

    for inputs, targets in loader:
        inputs = inputs.to(device, non_blocking=True)
        targets = targets.to(device, non_blocking=True)

        outputs = model(inputs)
        loss = criterion(outputs, targets)
        total_loss += loss.item()

    return {"val_loss": total_loss / len(loader)}
```

> `scaler.unscale_(optimizer)` çağrılmadan `clip_grad_norm_` uygulamak sessizce yanlıştır — ölçeklenmiş gradyanları kırpar, yani gerçekte hiç kırpmaz.

---

## Checkpoint Yönetimi

```python
def save_checkpoint(
    model: nn.Module,
    optimizer: torch.optim.Optimizer,
    epoch: int,
    metrics: dict,
    path: Path,
    is_best: bool = False,
) -> None:
    """Save training checkpoint."""
    checkpoint = {
        "epoch": epoch,
        "model_state_dict": model.state_dict(),
        "optimizer_state_dict": optimizer.state_dict(),
        "metrics": metrics,
    }
    torch.save(checkpoint, path / f"checkpoint_epoch_{epoch:04d}.pt")

    if is_best:
        torch.save(checkpoint, path / "best_model.pt")


def load_checkpoint(model: nn.Module, optimizer, path: Path) -> int:
    """Load checkpoint and return epoch number."""
    # weights_only=True — güvenilmeyen checkpoint'ler için zorunlu
    checkpoint = torch.load(path, map_location="cpu", weights_only=True)
    model.load_state_dict(checkpoint["model_state_dict"])
    optimizer.load_state_dict(checkpoint["optimizer_state_dict"])
    return checkpoint["epoch"]
```

---

## GPU Bellek Optimizasyonu

```python
# 1. Mixed Precision (AMP) — hız ve bellek
scaler = GradScaler()
with autocast(device_type="cuda"):
    output = model(input)

# 2. Gradient Checkpointing — bellek vs hesaplama tradeoff
from torch.utils.checkpoint import checkpoint_sequential
output = checkpoint_sequential(model, segments=4, input=x)

# 3. Bellek profillemesi
with torch.profiler.profile(
    activities=[torch.profiler.ProfilerActivity.CPU, torch.profiler.ProfilerActivity.CUDA],
    profile_memory=True,
) as prof:
    output = model(input)
print(prof.key_averages().table(sort_by="cuda_memory_usage", row_limit=10))

# 4. Bellek temizleme (OOM durumlarında)
torch.cuda.empty_cache()
import gc; gc.collect()
```

---

## Yaygın Sorunlar ve Çözümleri

| Sorun | Belirti | Çözüm |
|---|---|---|
| Gradient patlaması | Loss aniden NaN/Inf | `clip_grad_norm_(max_norm=1.0)` ekle |
| Gradient kaybı | Loss hiç düşmüyor | Learning rate artır, normalizasyon ekle |
| `zero_grad` unutma | Loss garip dalgalanma | Her batch başında `optimizer.zero_grad()` |
| `model.eval()` unutma | Validation çok değişken | Her eval döngüsünde `model.eval()` |
| CUDA OOM | RuntimeError: CUDA out of memory | Batch size küçült, AMP kullan, gradient checkpoint |
| CPU'da kalan tensor | Yavaş eğitim | `.to(device)` kontrolü, DataLoader `pin_memory=True` |
| Loss düşüyor ama metrik artmıyor | Loss ile metrik uyumsuz | Loss'un gerçekten hedef metriği optimize ettiğini doğrula |

---

## Tekrarlanabilirlik

```python
import torch
import random
import numpy as np
import os

def set_seed(seed: int = 42) -> None:
    """Set all random seeds for reproducibility."""
    torch.manual_seed(seed)
    torch.cuda.manual_seed_all(seed)
    np.random.seed(seed)
    random.seed(seed)
    os.environ["PYTHONHASHSEED"] = str(seed)

    # Deterministic ops — uyarı: yavaşlatır
    torch.backends.cudnn.deterministic = True
    torch.backends.cudnn.benchmark = False  # benchmark=True farklı kernel seçebilir

set_seed(42)
```

`DataLoader` çok worker'lı ise `worker_init_fn` ve `generator` da ayarlanmalı; aksi halde veri sırası seed'e rağmen değişir.

---

## Kontrol Listesi

- [ ] `model.train()` / `model.eval()` doğru çağrılıyor mu?
- [ ] `optimizer.zero_grad()` her batch başında var mı?
- [ ] Gradient clipping uygulandı mı (AMP'de `unscale_` sonrası)?
- [ ] Mixed precision (`torch.amp`) kullanıldı mı?
- [ ] Checkpoint kaydetme ve `weights_only=True` ile yükleme var mı?
- [ ] Seed sabitlendi mi (DataLoader worker'ları dahil)?
- [ ] DataLoader `pin_memory=True, num_workers>0` ayarlandı mı?
- [ ] Validation `@torch.no_grad()` altında mı?

