# Distributed Training

## Overview

When models or datasets become too large for a single GPU, **distributed training** spreads the workload across multiple GPUs or machines. This chapter covers data parallelism, model parallelism, and the frameworks that make distributed training accessible: **PyTorch DDP**, **DeepSpeed**, **Horovod**, and **Ray Train**.

---

## Why Distributed Training?

```
  Single GPU limitations:
  • GPU memory: 24-80 GB (model may not fit)
  • Training time: days/weeks for large models
  • Dataset: too large to process sequentially

  Distributed training:
  • Split work across 2, 4, 8, ... N GPUs
  • Near-linear speedup (8 GPUs ≈ 8× faster)
  • Train models that don't fit on one GPU
```

---

## Data Parallelism vs. Model Parallelism

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  DATA PARALLELISM:                                           │
  │  Same model on each GPU, different data                      │
  │                                                                │
  │  GPU 0: Model copy + Batch 0 → gradients₀  ┐               │
  │  GPU 1: Model copy + Batch 1 → gradients₁  ├→ AllReduce    │
  │  GPU 2: Model copy + Batch 2 → gradients₂  ┤  (average)    │
  │  GPU 3: Model copy + Batch 3 → gradients₃  ┘  → update all │
  │                                                                │
  │  Best when: model fits on one GPU, want faster training      │
  │                                                                │
  │  MODEL PARALLELISM:                                          │
  │  Different model parts on different GPUs                     │
  │                                                                │
  │  GPU 0: Layers 1-6     ──→ activations ──→                  │
  │  GPU 1: Layers 7-12    ──→ activations ──→                  │
  │  GPU 2: Layers 13-18   ──→ activations ──→                  │
  │  GPU 3: Layers 19-24   ──→ loss                             │
  │                                                                │
  │  Best when: model too large for one GPU                      │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## PyTorch Distributed Data Parallel (DDP)

```python
import torch
import torch.distributed as dist
import torch.nn as nn
from torch.nn.parallel import DistributedDataParallel as DDP
from torch.utils.data import DataLoader, DistributedSampler
import os


def setup(rank, world_size):
    os.environ["MASTER_ADDR"] = "localhost"
    os.environ["MASTER_PORT"] = "12355"
    dist.init_process_group("nccl", rank=rank, world_size=world_size)
    torch.cuda.set_device(rank)


def cleanup():
    dist.destroy_process_group()


def train(rank, world_size):
    setup(rank, world_size)
    
    # Model on this GPU
    model = nn.Sequential(
        nn.Linear(784, 256), nn.ReLU(),
        nn.Linear(256, 10),
    ).to(rank)
    
    # Wrap with DDP
    model = DDP(model, device_ids=[rank])
    
    # Distributed sampler ensures each GPU gets different data
    dataset = MyDataset()
    sampler = DistributedSampler(dataset, num_replicas=world_size, rank=rank)
    loader = DataLoader(dataset, batch_size=64, sampler=sampler)
    
    optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)
    
    for epoch in range(10):
        sampler.set_epoch(epoch)  # shuffle differently each epoch
        for batch_x, batch_y in loader:
            batch_x, batch_y = batch_x.to(rank), batch_y.to(rank)
            
            loss = nn.functional.cross_entropy(model(batch_x), batch_y)
            optimizer.zero_grad()
            loss.backward()      # gradients auto-synced by DDP!
            optimizer.step()
    
    cleanup()


# Launch on 4 GPUs
import torch.multiprocessing as mp
mp.spawn(train, args=(4,), nprocs=4, join=True)

# Or from command line:
# torchrun --nproc_per_node=4 train.py
```

---

## DeepSpeed (Microsoft)

```python
# DeepSpeed enables training very large models with ZeRO optimization
# ZeRO: Zero Redundancy Optimizer — partitions optimizer states,
# gradients, and parameters across GPUs

# deepspeed_config.json
# {
#   "train_batch_size": 256,
#   "gradient_accumulation_steps": 4,
#   "fp16": {"enabled": true},
#   "zero_optimization": {
#     "stage": 2,
#     "offload_optimizer": {"device": "cpu"},
#     "allgather_partitions": true
#   }
# }

import deepspeed

model = MyLargeModel()
optimizer = torch.optim.AdamW(model.parameters(), lr=1e-4)

# Initialize DeepSpeed
model_engine, optimizer, _, _ = deepspeed.initialize(
    model=model,
    optimizer=optimizer,
    config="deepspeed_config.json",
)

for batch in dataloader:
    loss = model_engine(batch)
    model_engine.backward(loss)
    model_engine.step()

# Launch: deepspeed --num_gpus=8 train.py
```

---

## Comparison of Frameworks

```
  ┌─────────────────┬──────────────┬──────────────┬──────────────┐
  │ Feature         │ PyTorch DDP  │ DeepSpeed    │ Horovod      │
  ├─────────────────┼──────────────┼──────────────┼──────────────┤
  │ Data parallel   │ ✓✓           │ ✓✓           │ ✓✓           │
  │ Model parallel  │ Manual       │ ✓ (ZeRO-3)  │ Limited      │
  │ Mixed precision │ via AMP      │ Built-in     │ via AMP      │
  │ CPU offload     │              │ ✓            │              │
  │ Ease of use     │ Medium       │ Medium       │ Easy         │
  │ Best for        │ Standard DDP │ Very large   │ Multi-node   │
  │                 │              │ models       │              │
  └─────────────────┴──────────────┴──────────────┴──────────────┘
```

---

## MLOps Considerations

```
  Distributed training + MLOps:

  1. LOGGING: Only log from rank 0 (avoid duplicate logs)
     if rank == 0:
         mlflow.log_metric("loss", loss)

  2. CHECKPOINTING: Save model state for fault tolerance
     torch.save(model.state_dict(), f"ckpt_epoch_{epoch}.pt")

  3. RESOURCE MANAGEMENT: Request GPUs via orchestrator
     Kubeflow: set_gpu_limit(4)
     K8s: resources.limits.nvidia.com/gpu: 4

  4. COST: Multi-GPU is expensive!
     Spot/preemptible instances + checkpointing = cost savings
```

---

## Revision Questions

1. **When should you use data parallelism vs. model parallelism?**
2. **How does PyTorch DDP synchronize gradients across GPUs?**
3. **What is DeepSpeed's ZeRO optimization?**
4. **What MLOps practices are important for distributed training?**

---

[Previous: 04-automated-training.md](04-automated-training.md) | [Next: ../05-Model-Versioning-and-Registry/01-model-versioning.md](../05-Model-Versioning-and-Registry/01-model-versioning.md)
