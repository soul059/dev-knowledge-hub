# 6. Reproducibility

> **Unit 8 · Practical Training** — Ensuring consistent, repeatable results across runs

---

## Chapter Overview

Run the same neural network code twice and you'll likely get different results. This non-determinism comes from **random weight initialization**, **data shuffling**, **GPU parallel computation**, and other sources of randomness baked into modern deep learning frameworks. Reproducibility — the ability to get the **exact same results** every time — is essential for debugging, comparing experiments fairly, and publishing scientific work. This chapter covers every source of randomness and how to control it.

---

## 1. Why Reproducibility Matters

```
  ┌──────────────────────────────────────────────────────────────┐
  │  WITHOUT REPRODUCIBILITY:                                    │
  │                                                              │
  │  Run 1: accuracy = 93.2%                                    │
  │  Run 2: accuracy = 94.1%     "Is the improvement from      │
  │  Run 3: accuracy = 92.8%      my code change or just        │
  │                                random variation?"            │
  │                                                              │
  │  → Can't tell if your change actually helped!               │
  │  → Can't debug (bug might not reproduce)                    │
  │  → Can't publish (reviewers can't verify)                   │
  │  → Can't deploy (which run's model is "the" model?)        │
  ├──────────────────────────────────────────────────────────────┤
  │  WITH REPRODUCIBILITY:                                       │
  │                                                              │
  │  Run 1: accuracy = 93.2%   (seed=42, commit=abc123)        │
  │  Run 2: accuracy = 93.2%   (same seed, same commit)        │
  │  Run 3: accuracy = 93.2%   (same seed, same commit)        │
  │                                                              │
  │  Changed code → accuracy = 94.1%                            │
  │  → DEFINITELY an improvement!                               │
  └──────────────────────────────────────────────────────────────┘
```

---

## 2. Sources of Randomness in Deep Learning

```
  ┌──────────────────────────────────────────────────────────────┐
  │  SOURCE                      │ COMPONENT        │ FIXABLE? │
  ├──────────────────────────────┼──────────────────┼──────────┤
  │  Weight initialization       │ torch/numpy      │ ✓ Yes    │
  │  Data shuffling              │ DataLoader       │ ✓ Yes    │
  │  Dropout masks               │ torch            │ ✓ Yes    │
  │  Data augmentation           │ random/torch     │ ✓ Yes    │
  │  Batch normalization (train) │ batch statistics │ ✓ Yes    │
  │  Python hash randomization   │ Python           │ ✓ Yes    │
  │  NumPy random operations     │ numpy            │ ✓ Yes    │
  │  CUDA convolution algorithms │ cuDNN            │ ✓ Mostly │
  │  Atomic GPU operations       │ CUDA             │ ⚠ Hard   │
  │  Multi-threaded DataLoader   │ Workers          │ ⚠ Hard   │
  │  Floating-point non-assoc.   │ GPU math         │ ✗ No     │
  └──────────────────────────────┴──────────────────┴──────────┘
```

---

## 3. Setting Seeds: The Complete Guide

### The Comprehensive Seed Function

```python
import random
import os
import numpy as np
import torch

def set_seed(seed: int = 42):
    """
    Set all random seeds for reproducibility.
    
    This function sets seeds for:
    - Python's random module
    - NumPy
    - PyTorch (CPU and GPU)
    - CUDA (if available)
    - Python hash seed
    """
    # 1. Python's built-in random module
    random.seed(seed)
    
    # 2. Python hash seed (affects dict ordering, set hashing)
    os.environ['PYTHONHASHSEED'] = str(seed)
    
    # 3. NumPy
    np.random.seed(seed)
    
    # 4. PyTorch CPU
    torch.manual_seed(seed)
    
    # 5. PyTorch GPU (all GPUs)
    torch.cuda.manual_seed(seed)
    torch.cuda.manual_seed_all(seed)  # for multi-GPU
    
    # 6. cuDNN deterministic mode
    torch.backends.cudnn.deterministic = True
    torch.backends.cudnn.benchmark = False
    
    # 7. PyTorch deterministic algorithms (PyTorch >= 1.8)
    # This makes ALL PyTorch operations deterministic
    # Some operations will raise errors if no deterministic implementation exists
    torch.use_deterministic_algorithms(True)
    
    # 8. Set CUBLAS workspace config for deterministic CUDA ops
    os.environ['CUBLAS_WORKSPACE_CONFIG'] = ':4096:8'
    
    print(f"All seeds set to {seed}")


# Usage — call BEFORE anything else:
set_seed(42)
model = MyModel()
optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)
train(model, optimizer, train_loader)
```

### Understanding Each Seed

```
  ┌──────────────────────────────────────────────────────────────┐
  │  SEED                          WHAT IT CONTROLS              │
  ├──────────────────────────────────────────────────────────────┤
  │                                                              │
  │  random.seed(42)               Python random module:         │
  │                                • random.random()             │
  │                                • random.shuffle()            │
  │                                • random.choice()             │
  │                                                              │
  │  np.random.seed(42)            NumPy operations:             │
  │                                • np.random.randn()           │
  │                                • np.random.shuffle()         │
  │                                • sklearn's train_test_split  │
  │                                                              │
  │  torch.manual_seed(42)         PyTorch CPU operations:       │
  │                                • torch.randn()               │
  │                                • Weight initialization       │
  │                                • Dropout masks               │
  │                                                              │
  │  torch.cuda.manual_seed(42)    PyTorch GPU operations:       │
  │                                • CUDA random number gen      │
  │                                • GPU dropout masks           │
  │                                                              │
  │  torch.cuda.manual_seed_all()  ALL GPUs (multi-GPU):         │
  │                                • Same as above, all devices  │
  │                                                              │
  │  PYTHONHASHSEED=42             Python hash function:          │
  │                                • dict iteration order        │
  │                                • set iteration order         │
  └──────────────────────────────────────────────────────────────┘
```

---

## 4. Deterministic Algorithms

### cuDNN Benchmark Mode

```python
# cuDNN BENCHMARK mode:
# When True: cuDNN tests multiple convolution algorithms and picks the fastest
# Problem: the "fastest" choice may differ between runs → non-deterministic!

# For SPEED (non-deterministic):
torch.backends.cudnn.benchmark = True     # ← default for many codebases
torch.backends.cudnn.deterministic = False

# For REPRODUCIBILITY (deterministic but slower):
torch.backends.cudnn.benchmark = False    # ← always use same algorithm
torch.backends.cudnn.deterministic = True # ← use deterministic implementations
```

### PyTorch Deterministic Algorithms

```python
# PyTorch 1.8+ provides a global flag for deterministic operations
torch.use_deterministic_algorithms(True)

# This makes the following operations deterministic:
# - torch.nn.Conv1d, Conv2d, Conv3d (backward)
# - torch.nn.MaxPool3d
# - torch.bmm
# - torch.Tensor.scatter_add_
# - torch.gather
# - torch.index_add_
# - torch.index_copy_
# - torch.index_put_ (with accumulate=True)
# - torch.Tensor.put_ (with accumulate=True)
# - torch.Tensor.scatter_ (with reduce='add' or 'multiply')
# - torch.nn.functional.interpolate (some modes)

# IMPORTANT: Some operations will raise RuntimeError if no deterministic
# implementation exists. In that case, use:
torch.use_deterministic_algorithms(True, warn_only=True)
# This logs a warning instead of raising an error
```

### Non-Deterministic Operations

```
  ┌──────────────────────────────────────────────────────────────┐
  │  OPERATIONS THAT MAY BE NON-DETERMINISTIC ON GPU:           │
  │                                                              │
  │  • torch.Tensor.index_add_()                                │
  │  • torch.Tensor.scatter_add_()                              │
  │  • torch.gather() backward                                  │
  │  • torch.bmm() backward                                     │
  │  • Atomic operations (multiple threads write to same addr)  │
  │  • Convolution backward (cuDNN)                             │
  │  • MaxPool backward                                          │
  │  • Interpolation backward                                   │
  │                                                              │
  │  Root cause: GPU floating-point operations are not           │
  │  associative — (a+b)+c ≠ a+(b+c) for floats.               │
  │  The ORDER of operations matters, and GPU parallel           │
  │  execution doesn't guarantee order.                         │
  └──────────────────────────────────────────────────────────────┘
```

---

## 5. DataLoader Workers and Reproducibility

```python
# DataLoader with multiple workers can introduce non-determinism
# Each worker has its own random state!

def seed_worker(worker_id):
    """Ensure each DataLoader worker has a deterministic seed."""
    worker_seed = torch.initial_seed() % 2**32
    np.random.seed(worker_seed)
    random.seed(worker_seed)

# Create a generator for the DataLoader
g = torch.Generator()
g.manual_seed(42)

train_loader = torch.utils.data.DataLoader(
    dataset,
    batch_size=128,
    shuffle=True,
    num_workers=4,
    worker_init_fn=seed_worker,  # ← seed each worker!
    generator=g,                  # ← seed the shuffling!
    persistent_workers=True,      # keep workers alive (optional)
)
```

```
  Without worker_init_fn:
  ┌──────────────────────────────────────────────┐
  │  Worker 0: random seed = 98765  (random!)    │
  │  Worker 1: random seed = 12345  (random!)    │
  │  Worker 2: random seed = 55555  (random!)    │
  │  Worker 3: random seed = 33333  (random!)    │
  │  → Different augmentations each run!         │
  └──────────────────────────────────────────────┘
  
  With worker_init_fn:
  ┌──────────────────────────────────────────────┐
  │  Worker 0: random seed = f(42, 0) = 42      │
  │  Worker 1: random seed = f(42, 1) = 43      │
  │  Worker 2: random seed = f(42, 2) = 44      │
  │  Worker 3: random seed = f(42, 3) = 45      │
  │  → Same augmentations every run!             │
  └──────────────────────────────────────────────┘
```

---

## 6. Documenting Experiments

### What to Record for Each Experiment

```
  ┌──────────────────────────────────────────────────────────────┐
  │  EXPERIMENT LOG — Minimum Required Information              │
  │                                                              │
  │  IDENTIFICATION:                                             │
  │  □ Experiment name / ID                                     │
  │  □ Date and time                                            │
  │  □ Author                                                    │
  │  □ Purpose / hypothesis                                     │
  │                                                              │
  │  CODE:                                                       │
  │  □ Git commit hash                                          │
  │  □ Branch name                                              │
  │  □ Diff from base (if uncommitted changes)                  │
  │                                                              │
  │  ENVIRONMENT:                                                │
  │  □ Python version                                           │
  │  □ PyTorch version                                          │
  │  □ CUDA version                                             │
  │  □ GPU model                                                │
  │  □ pip freeze / conda list (all package versions!)          │
  │                                                              │
  │  HYPERPARAMETERS:                                            │
  │  □ Learning rate                                            │
  │  □ Batch size                                               │
  │  □ Optimizer and its parameters                             │
  │  □ Architecture details                                     │
  │  □ Regularization (dropout, weight decay)                   │
  │  □ Number of epochs                                         │
  │  □ Random seed                                              │
  │                                                              │
  │  DATA:                                                       │
  │  □ Dataset name and version                                 │
  │  □ Train/val/test split sizes                               │
  │  □ Preprocessing steps                                      │
  │  □ Data augmentation                                        │
  │                                                              │
  │  RESULTS:                                                    │
  │  □ Final train loss and accuracy                            │
  │  □ Final val loss and accuracy                              │
  │  □ Test accuracy (if evaluated)                             │
  │  □ Training time                                            │
  │  □ Loss curves (saved as images)                            │
  └──────────────────────────────────────────────────────────────┘
```

### Automated Experiment Logging

```python
import json
import subprocess
import sys
from datetime import datetime
from pathlib import Path

def log_experiment(config, results, save_dir='experiments'):
    """Automatically log experiment details."""
    Path(save_dir).mkdir(exist_ok=True)
    
    experiment = {
        'timestamp': datetime.now().isoformat(),
        
        # Code version
        'git_commit': subprocess.getoutput('git rev-parse HEAD').strip(),
        'git_branch': subprocess.getoutput('git rev-parse --abbrev-ref HEAD').strip(),
        'git_dirty': 'modified' in subprocess.getoutput('git status --porcelain'),
        
        # Environment
        'python_version': sys.version,
        'pytorch_version': torch.__version__,
        'cuda_available': torch.cuda.is_available(),
        'cuda_version': torch.version.cuda if torch.cuda.is_available() else None,
        'gpu_name': torch.cuda.get_device_name(0) if torch.cuda.is_available() else None,
        
        # Hyperparameters
        'config': config,
        
        # Results
        'results': results,
    }
    
    # Save with timestamp-based filename
    filename = f"{save_dir}/exp_{datetime.now().strftime('%Y%m%d_%H%M%S')}.json"
    with open(filename, 'w') as f:
        json.dump(experiment, f, indent=2)
    
    print(f"Experiment logged to {filename}")
    return filename

# Usage:
config = {
    'seed': 42,
    'learning_rate': 1e-3,
    'batch_size': 128,
    'epochs': 30,
    'hidden_size': 256,
    'dropout': 0.3,
    'optimizer': 'AdamW',
    'weight_decay': 1e-4,
}

results = {
    'best_val_acc': 0.9821,
    'final_train_loss': 0.0234,
    'final_val_loss': 0.0567,
    'training_time_seconds': 342.5,
}

log_experiment(config, results)
```

---

## 7. Version Control for Experiments

### Using Git for Experiment Tracking

```
  ┌──────────────────────────────────────────────────────────────┐
  │  BEST PRACTICES FOR EXPERIMENT VERSION CONTROL              │
  │                                                              │
  │  1. COMMIT BEFORE EACH EXPERIMENT                           │
  │     git add -A && git commit -m "exp: description"          │
  │                                                              │
  │  2. TAG IMPORTANT EXPERIMENTS                               │
  │     git tag -a exp-v1-baseline -m "Baseline MLP"           │
  │     git tag -a exp-v2-dropout -m "Added dropout 0.3"       │
  │                                                              │
  │  3. USE BRANCHES FOR MAJOR CHANGES                          │
  │     git checkout -b experiment/resnet-architecture          │
  │                                                              │
  │  4. SAVE MODEL CHECKPOINTS WITH GIT HASH                   │
  │     model_abc123_epoch30.pth  (abc123 = git hash)          │
  │                                                              │
  │  5. DON'T COMMIT:                                           │
  │     • Large datasets (use .gitignore)                       │
  │     • Model weights (use model registry)                    │
  │     • TensorBoard logs (too large)                          │
  │     • __pycache__, .pyc files                               │
  └──────────────────────────────────────────────────────────────┘
```

### Requirements File

```python
# Save exact package versions:
# pip freeze > requirements.txt

# Or for conda:
# conda env export > environment.yml

# In your training script, log the versions:
import pkg_resources
installed = {pkg.key: pkg.version for pkg in pkg_resources.working_set}
print(f"torch=={installed.get('torch', 'N/A')}")
print(f"numpy=={installed.get('numpy', 'N/A')}")
print(f"torchvision=={installed.get('torchvision', 'N/A')}")
```

---

## 8. Challenges with GPU Non-Determinism

```
  WHY PERFECT GPU REPRODUCIBILITY IS HARD:
  
  ┌──────────────────────────────────────────────────────────────┐
  │                                                              │
  │  1. FLOATING-POINT NON-ASSOCIATIVITY                        │
  │     (1.0 + 1e-8) + 1e-8 ≠ 1.0 + (1e-8 + 1e-8)            │
  │     GPU parallel reduction doesn't guarantee order          │
  │                                                              │
  │  2. ATOMIC OPERATIONS                                       │
  │     Multiple GPU threads adding to same memory location     │
  │     Order is non-deterministic → result varies              │
  │                                                              │
  │  3. cuDNN ALGORITHM SELECTION                               │
  │     Different GPU models may select different algorithms    │
  │     Even same GPU with different driver versions!           │
  │                                                              │
  │  4. MULTI-GPU TRAINING                                      │
  │     Communication order between GPUs varies                 │
  │     Gradient all-reduce order is non-deterministic          │
  │                                                              │
  │  5. DIFFERENT GPU ARCHITECTURES                             │
  │     V100 vs A100 vs RTX 3090 have different hardware       │
  │     Same code + same seed → different results!             │
  └──────────────────────────────────────────────────────────────┘
  
  PRACTICAL REALITY:
  ┌──────────────────────────────────────────────────────────────┐
  │                                                              │
  │  CPU-only:                                                   │
  │  → Perfect reproducibility ✓                                │
  │  → Very slow ✗                                              │
  │                                                              │
  │  GPU with all deterministic flags:                          │
  │  → Near-perfect reproducibility (same GPU model) ✓         │
  │  → 10-30% slower ✗                                         │
  │  → Some ops may error ✗                                     │
  │                                                              │
  │  GPU without deterministic flags:                           │
  │  → Typical variation: ±0.1-0.5% accuracy                   │
  │  → Fastest ✓                                                │
  │                                                              │
  │  RECOMMENDATION:                                             │
  │  Use deterministic mode for debugging and comparisons.      │
  │  Use non-deterministic mode for final training (faster).    │
  │  Run multiple seeds and report mean ± std for papers.       │
  └──────────────────────────────────────────────────────────────┘
```

---

## 9. Complete Reproducible Training Template

```python
import random
import os
import numpy as np
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader
from torchvision import datasets, transforms
import json
from datetime import datetime

# ═══════════════════════════════════════════════════════════
#  CONFIGURATION
# ═══════════════════════════════════════════════════════════

CONFIG = {
    'seed': 42,
    'batch_size': 128,
    'learning_rate': 1e-3,
    'weight_decay': 1e-4,
    'epochs': 30,
    'hidden_size': 256,
    'dropout': 0.3,
    'num_workers': 4,
    'device': 'cuda' if torch.cuda.is_available() else 'cpu',
}

# ═══════════════════════════════════════════════════════════
#  REPRODUCIBILITY SETUP
# ═══════════════════════════════════════════════════════════

def set_seed(seed):
    random.seed(seed)
    os.environ['PYTHONHASHSEED'] = str(seed)
    np.random.seed(seed)
    torch.manual_seed(seed)
    torch.cuda.manual_seed(seed)
    torch.cuda.manual_seed_all(seed)
    torch.backends.cudnn.deterministic = True
    torch.backends.cudnn.benchmark = False
    # Use warn_only=True to avoid errors from non-deterministic ops
    torch.use_deterministic_algorithms(True, warn_only=True)
    os.environ['CUBLAS_WORKSPACE_CONFIG'] = ':4096:8'

set_seed(CONFIG['seed'])

def seed_worker(worker_id):
    worker_seed = torch.initial_seed() % 2**32
    np.random.seed(worker_seed)
    random.seed(worker_seed)

g = torch.Generator()
g.manual_seed(CONFIG['seed'])

# ═══════════════════════════════════════════════════════════
#  DATA
# ═══════════════════════════════════════════════════════════

transform = transforms.Compose([
    transforms.ToTensor(),
    transforms.Normalize((0.1307,), (0.3081,))
])

train_data = datasets.MNIST('./data', train=True, download=True, transform=transform)
test_data = datasets.MNIST('./data', train=False, transform=transform)

train_loader = DataLoader(
    train_data, batch_size=CONFIG['batch_size'], shuffle=True,
    num_workers=CONFIG['num_workers'],
    worker_init_fn=seed_worker, generator=g,
    pin_memory=True
)

test_loader = DataLoader(
    test_data, batch_size=512,
    num_workers=CONFIG['num_workers'],
    pin_memory=True
)

# ═══════════════════════════════════════════════════════════
#  MODEL
# ═══════════════════════════════════════════════════════════

device = torch.device(CONFIG['device'])

model = nn.Sequential(
    nn.Flatten(),
    nn.Linear(784, CONFIG['hidden_size']),
    nn.ReLU(),
    nn.Dropout(CONFIG['dropout']),
    nn.Linear(CONFIG['hidden_size'], CONFIG['hidden_size'] // 2),
    nn.ReLU(),
    nn.Dropout(CONFIG['dropout']),
    nn.Linear(CONFIG['hidden_size'] // 2, 10)
).to(device)

optimizer = optim.AdamW(
    model.parameters(),
    lr=CONFIG['learning_rate'],
    weight_decay=CONFIG['weight_decay']
)

criterion = nn.CrossEntropyLoss()

# ═══════════════════════════════════════════════════════════
#  TRAINING
# ═══════════════════════════════════════════════════════════

best_acc = 0
for epoch in range(CONFIG['epochs']):
    model.train()
    total_loss = 0
    
    for X, y in train_loader:
        X, y = X.to(device), y.to(device)
        optimizer.zero_grad()
        loss = criterion(model(X), y)
        loss.backward()
        optimizer.step()
        total_loss += loss.item()
    
    # Evaluate
    model.eval()
    correct = 0
    with torch.no_grad():
        for X, y in test_loader:
            X, y = X.to(device), y.to(device)
            correct += (model(X).argmax(1) == y).sum().item()
    
    acc = correct / len(test_data)
    avg_loss = total_loss / len(train_loader)
    
    if acc > best_acc:
        best_acc = acc
        torch.save({
            'epoch': epoch,
            'model_state_dict': model.state_dict(),
            'optimizer_state_dict': optimizer.state_dict(),
            'accuracy': acc,
            'config': CONFIG,
        }, 'best_model.pth')
    
    print(f"Epoch {epoch+1:2d} | Loss: {avg_loss:.4f} | Acc: {acc:.4f}")

# ═══════════════════════════════════════════════════════════
#  LOG RESULTS
# ═══════════════════════════════════════════════════════════

results = {
    'best_accuracy': best_acc,
    'config': CONFIG,
    'pytorch_version': torch.__version__,
    'cuda_available': torch.cuda.is_available(),
    'timestamp': datetime.now().isoformat(),
}

with open('experiment_results.json', 'w') as f:
    json.dump(results, f, indent=2)

print(f"\nBest accuracy: {best_acc:.4f}")
print(f"Results saved to experiment_results.json")
```

---

## 10. Summary Table

| Source of Randomness | How to Fix | Performance Impact |
|---------------------|------------|-------------------|
| **Python random** | `random.seed(42)` | None |
| **NumPy random** | `np.random.seed(42)` | None |
| **PyTorch CPU** | `torch.manual_seed(42)` | None |
| **PyTorch GPU** | `torch.cuda.manual_seed_all(42)` | None |
| **Python hashing** | `PYTHONHASHSEED=42` | None |
| **cuDNN benchmark** | `cudnn.benchmark = False` | ~5-15% slower |
| **cuDNN deterministic** | `cudnn.deterministic = True` | ~5-10% slower |
| **Deterministic algorithms** | `use_deterministic_algorithms(True)` | ~10-30% slower |
| **DataLoader workers** | `worker_init_fn` + `generator` | None |
| **CUBLAS workspace** | `CUBLAS_WORKSPACE_CONFIG=:4096:8` | Negligible |
| **GPU atomic operations** | Cannot fully fix | N/A |
| **Cross-GPU communication** | Cannot fully fix | N/A |

---

## 11. Revision Questions

1. **List all the random seeds you need to set for a fully reproducible PyTorch training run.** Explain what each seed controls and write the complete seed-setting function.

2. **What is `torch.backends.cudnn.benchmark` and why does setting it to `True` break reproducibility?** What is the performance cost of setting it to `False`?

3. **Explain why GPU computations can be non-deterministic even with all seeds set.** What role does floating-point non-associativity play?

4. **How do you handle DataLoader workers for reproducibility?** Write the code for `worker_init_fn` and explain why each DataLoader worker needs its own seed.

5. **What information should you record for each experiment to ensure full reproducibility?** Create a checklist of at least 10 items.

6. **A researcher claims their model achieves 95.0% accuracy. When you run their code with the same seed, you get 94.8%. List three reasons why the results might differ despite using the same seed.**

---

| [← 05 Debugging Neural Networks](05-debugging-neural-networks.md) | [Unit 8 Home](README.md) | [Unit 9 → Deep Learning Frameworks](../09-Deep-Learning-Frameworks/) |
|:---|:---:|---:|
