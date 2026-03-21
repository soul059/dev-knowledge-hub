# 4. GPU Training

> **Unit 9 · Deep Learning Frameworks** — Leveraging GPUs for fast neural network training

---

## Chapter Overview

Training a neural network on a CPU might take days; the same model on a GPU takes hours or minutes. GPUs excel at deep learning because neural network training is fundamentally about **massive parallel matrix operations** — exactly what GPUs were built for. This chapter explains why GPUs are essential, covers CUDA basics, moving models and data to GPU, multi-GPU training strategies (DataParallel and DistributedDataParallel), mixed precision training for 2× speedup, memory management, and common GPU errors.

---

## 1. Why GPUs for Deep Learning?

### CPU vs GPU Architecture

```
  CPU: Few powerful cores (serial processing)
  ┌────────────────────────────────────────────────┐
  │  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐      │
  │  │Core 1│  │Core 2│  │Core 3│  │Core 4│      │
  │  │ Fast │  │ Fast │  │ Fast │  │ Fast │      │
  │  │ Smart│  │ Smart│  │ Smart│  │ Smart│      │
  │  └──────┘  └──────┘  └──────┘  └──────┘      │
  │  4-64 cores, each with complex control logic  │
  │  Great for sequential tasks with branching    │
  └────────────────────────────────────────────────┘
  
  GPU: Thousands of simple cores (parallel processing)
  ┌────────────────────────────────────────────────┐
  │  ┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐ │
  │  │ ││ ││ ││ ││ ││ ││ ││ ││ ││ ││ ││ ││ │ │
  │  ┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐ │
  │  │ ││ ││ ││ ││ ││ ││ ││ ││ ││ ││ ││ ││ │ │
  │  ┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐ │
  │  │ ││ ││ ││ ││ ││ ││ ││ ││ ││ ││ ││ ││ │ │
  │  (continues... thousands of small cores)      │
  │  5,000-16,000+ cores, simple control logic    │
  │  Perfect for: same operation on many elements │
  └────────────────────────────────────────────────┘
```

### Why Neural Networks Map to GPUs

```
  Neural network training = MATRIX MULTIPLICATIONS
  
  Forward pass:  h = W × x + b     (matrix multiply)
  Backward pass: ∂L/∂W = ∂L/∂h × xᵀ  (matrix multiply)
  Weight update: W = W - α × ∂L/∂W    (element-wise)
  
  Matrix multiplication: C[i,j] = Σ A[i,k] × B[k,j]
  
  Each element C[i,j] is INDEPENDENT:
  ┌──────────────────────────────────────────────┐
  │  C[0,0] can be computed independently from   │
  │  C[0,1], C[1,0], C[1,1], ...               │
  │                                              │
  │  → Compute ALL elements simultaneously!      │
  │  → 10,000 GPU cores = 10,000 elements at once│
  └──────────────────────────────────────────────┘
  
  Speedup example:
  ┌──────────────────────────────────────────────┐
  │  Matrix multiply: (1000×1000) × (1000×1000) │
  │  CPU (8 cores):   ~200 ms                    │
  │  GPU (5000 cores): ~2 ms   → 100× faster!  │
  └──────────────────────────────────────────────┘
```

---

## 2. CUDA Basics

```
  CUDA = Compute Unified Device Architecture (NVIDIA)
  
  ┌──────────────────────────────────────────────────────────────┐
  │  Software Stack:                                             │
  │                                                              │
  │  Your PyTorch Code                                           │
  │       ↓                                                      │
  │  PyTorch CUDA Backend                                        │
  │       ↓                                                      │
  │  cuDNN (optimized DL primitives)                             │
  │       ↓                                                      │
  │  cuBLAS (optimized linear algebra)                           │
  │       ↓                                                      │
  │  CUDA Runtime                                                │
  │       ↓                                                      │
  │  NVIDIA GPU Driver                                           │
  │       ↓                                                      │
  │  NVIDIA GPU Hardware                                         │
  └──────────────────────────────────────────────────────────────┘
```

### Checking GPU Availability

```python
import torch

# Check if CUDA is available
print(f"CUDA available: {torch.cuda.is_available()}")
print(f"CUDA version: {torch.version.cuda}")
print(f"cuDNN version: {torch.backends.cudnn.version()}")

# GPU information
if torch.cuda.is_available():
    print(f"GPU count: {torch.cuda.device_count()}")
    print(f"GPU name: {torch.cuda.get_device_name(0)}")
    print(f"GPU memory: {torch.cuda.get_device_properties(0).total_mem / 1e9:.1f} GB")
    
    # Current memory usage
    print(f"Allocated: {torch.cuda.memory_allocated(0) / 1e6:.1f} MB")
    print(f"Cached: {torch.cuda.memory_reserved(0) / 1e6:.1f} MB")

# Device selection pattern
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
print(f"Using device: {device}")
```

---

## 3. Moving Model and Data to GPU

### The .to(device) Pattern

```python
# ── Standard GPU training setup ──

device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')

# Move MODEL to GPU (once, at setup)
model = MLP().to(device)

# Move DATA to GPU (every batch)
for X, y in train_loader:
    X = X.to(device)       # move input to GPU
    y = y.to(device)       # move labels to GPU
    
    outputs = model(X)     # forward pass on GPU
    loss = criterion(outputs, y)
    loss.backward()        # backward pass on GPU
    optimizer.step()       # update on GPU
```

```
  Data Flow in GPU Training:
  
  CPU Memory                          GPU Memory
  ┌──────────────┐                   ┌──────────────────┐
  │  DataLoader   │                   │  Model (weights) │
  │  loads batch  │──── .to(device) ──│  Activations     │
  │  from disk    │       ──→        │  Gradients       │
  │               │                   │  Optimizer state │
  │  X, y on CPU  │                   │  X, y on GPU     │
  └──────────────┘                   └──────────────────┘
        ↑                                    │
   Disk/SSD                            GPU computes
  (dataset)                            forward + backward
                                       + weight update
```

### pin_memory for Faster Transfer

```python
# pin_memory=True speeds up CPU→GPU data transfer
train_loader = DataLoader(
    dataset,
    batch_size=128,
    shuffle=True,
    num_workers=4,
    pin_memory=True,  # ← pre-allocates in page-locked CPU memory
)

# Then use non_blocking=True for async transfer
for X, y in train_loader:
    X = X.to(device, non_blocking=True)  # async transfer
    y = y.to(device, non_blocking=True)
```

---

## 4. Multi-GPU Training

### DataParallel (DP) — Simple but Suboptimal

```
  DataParallel: Split batch across GPUs
  
  ┌───────────────────────────────────────────────────────────┐
  │  Input batch: 128 samples                                │
  │                                                           │
  │  GPU 0 (master):  GPU 1:         GPU 2:        GPU 3:   │
  │  32 samples       32 samples     32 samples    32 samples│
  │  ┌─────────┐     ┌─────────┐    ┌─────────┐  ┌─────────┐│
  │  │ forward │     │ forward │    │ forward │  │ forward ││
  │  │ backward│     │ backward│    │ backward│  │ backward││
  │  └────┬────┘     └────┬────┘    └────┬────┘  └────┬────┘│
  │       │               │              │             │     │
  │       └───────────────┼──────────────┼─────────────┘     │
  │                       ↓                                   │
  │              GPU 0: gather gradients                      │
  │              GPU 0: update weights                        │
  │              GPU 0: broadcast weights to all GPUs         │
  └───────────────────────────────────────────────────────────┘
  
  Problem: GPU 0 does MORE work (gather + update + broadcast)
  → GPU 0 becomes bottleneck → poor scaling!
```

```python
# DataParallel — one line!
model = nn.DataParallel(model)
model = model.to(device)

# Everything else stays the same
for X, y in train_loader:
    X, y = X.to(device), y.to(device)
    output = model(X)       # automatically splits batch across GPUs
    loss = criterion(output, y)
    loss.backward()
    optimizer.step()
```

### DistributedDataParallel (DDP) — Recommended

```
  DDP: Each GPU runs a SEPARATE process
  
  ┌───────────────────────────────────────────────────────────┐
  │  Process 0 (GPU 0):  Process 1 (GPU 1):                  │
  │  32 samples          32 samples                           │
  │  ┌─────────────┐    ┌─────────────┐                      │
  │  │ Own DataLoader│   │ Own DataLoader│   (separate data!) │
  │  │ forward      │    │ forward      │                      │
  │  │ backward     │    │ backward     │                      │
  │  │ gradients    │←──→│ gradients    │   ← All-Reduce     │
  │  │ update       │    │ update       │   (average grads)   │
  │  └─────────────┘    └─────────────┘                      │
  │                                                           │
  │  All-Reduce: each GPU sends AND receives gradients       │
  │  → NO bottleneck GPU!                                    │
  │  → Near-linear scaling!                                  │
  └───────────────────────────────────────────────────────────┘
```

```python
import torch.distributed as dist
from torch.nn.parallel import DistributedDataParallel as DDP
from torch.utils.data.distributed import DistributedSampler

def setup(rank, world_size):
    """Initialize distributed training."""
    dist.init_process_group("nccl", rank=rank, world_size=world_size)
    torch.cuda.set_device(rank)

def cleanup():
    dist.destroy_process_group()

def train(rank, world_size):
    setup(rank, world_size)
    
    # Create model on this GPU
    model = MLP().to(rank)
    model = DDP(model, device_ids=[rank])
    
    # Each GPU gets different data!
    sampler = DistributedSampler(dataset, num_replicas=world_size, rank=rank)
    loader = DataLoader(dataset, batch_size=32, sampler=sampler)
    
    optimizer = optim.Adam(model.parameters(), lr=1e-3)
    criterion = nn.CrossEntropyLoss()
    
    for epoch in range(num_epochs):
        sampler.set_epoch(epoch)  # important for shuffling!
        for X, y in loader:
            X, y = X.to(rank), y.to(rank)
            optimizer.zero_grad()
            loss = criterion(model(X), y)
            loss.backward()     # gradients are automatically all-reduced!
            optimizer.step()
    
    cleanup()

# Launch with:
# torchrun --nproc_per_node=4 train.py
# or
# python -m torch.distributed.launch --nproc_per_node=4 train.py
```

### DP vs DDP Comparison

```
  ┌──────────────────────┬──────────────────┬──────────────────────┐
  │  Feature             │  DataParallel    │  DistributedDataPar. │
  ├──────────────────────┼──────────────────┼──────────────────────┤
  │  Processes           │  Single process  │  Multiple processes  │
  │  Communication       │  Through GPU 0   │  All-Reduce (equal) │
  │  Scaling             │  Poor (>2 GPUs)  │  Near-linear         │
  │  Code complexity     │  1 line          │  ~20 lines           │
  │  Multi-node          │  No              │  Yes                 │
  │  GIL issue           │  Yes (Python GIL)│  No (separate procs) │
  │  Recommendation      │  Quick testing   │  ALWAYS use this!    │
  └──────────────────────┴──────────────────┴──────────────────────┘
```

---

## 5. Mixed Precision Training

### What is Mixed Precision?

```
  Standard Training:        Mixed Precision Training:
  FP32 everywhere           FP16 forward + backward
  (32-bit floats)           FP32 weight updates
  
  ┌────────────────┐        ┌────────────────────────────────┐
  │  Weights: FP32 │        │  Master weights: FP32          │
  │  Activations:  │        │  Forward/Backward: FP16        │
  │    FP32        │        │  Loss scaling: prevents FP16   │
  │  Gradients:    │        │    underflow                   │
  │    FP32        │        │  Weight update: FP32           │
  └────────────────┘        └────────────────────────────────┘
  
  Benefits:
  ✓ ~2× faster (FP16 operations are faster on modern GPUs)
  ✓ ~2× less memory (FP16 = half the bytes)
  ✓ Can fit ~2× larger batch size!
  ✓ Minimal accuracy impact (when done correctly)
  
  FP32: 1 sign + 8 exponent + 23 mantissa = 32 bits
  FP16: 1 sign + 5 exponent + 10 mantissa = 16 bits
                                       ↑
                              Less precision but OK
                              for forward/backward
```

### PyTorch AMP (Automatic Mixed Precision)

```python
from torch.cuda.amp import autocast, GradScaler

model = MLP().to(device)
optimizer = optim.Adam(model.parameters(), lr=1e-3)
criterion = nn.CrossEntropyLoss()

# Create gradient scaler
scaler = GradScaler()

for epoch in range(num_epochs):
    model.train()
    for X, y in train_loader:
        X, y = X.to(device), y.to(device)
        optimizer.zero_grad()
        
        # Forward pass in mixed precision
        with autocast():
            outputs = model(X)           # FP16 operations
            loss = criterion(outputs, y) # FP16 loss computation
        
        # Backward pass with scaled gradients
        scaler.scale(loss).backward()    # scale loss to prevent FP16 underflow
        scaler.step(optimizer)           # unscale gradients, then optimizer.step()
        scaler.update()                  # adjust scale factor
    
    # Validation (also use autocast for speed)
    model.eval()
    with torch.no_grad():
        with autocast():
            for X, y in val_loader:
                X, y = X.to(device), y.to(device)
                outputs = model(X)
```

### Why Loss Scaling is Needed

```
  FP16 range: ≈ 6e-5 to 65504
  
  Problem: Many gradients are VERY small (e.g., 1e-7)
  → They become 0 in FP16 → training breaks!
  
  Solution: Loss Scaling
  1. Multiply loss by scale factor (e.g., 1024)
  2. All gradients are now 1024× larger → fit in FP16
  3. Before optimizer.step(), divide gradients by 1024
  4. Net effect: gradients are computed in FP16 without underflow
  
  GradScaler does this automatically!
  It also dynamically adjusts the scale factor:
  - If no overflow: increase scale (more precision)
  - If overflow (NaN/Inf): decrease scale, skip step
```

---

## 6. Memory Management

### Understanding GPU Memory Usage

```
  GPU Memory Breakdown:
  
  ┌────────────────────────────────────────────────────┐
  │  Model Parameters          ████  ~500 MB           │
  │  (weights + biases)                                │
  ├────────────────────────────────────────────────────┤
  │  Optimizer State          ████████  ~1000 MB       │
  │  (Adam: 2× model size)                            │
  ├────────────────────────────────────────────────────┤
  │  Activations              ████████████████  ~2 GB  │
  │  (saved for backward)     scales with batch size!  │
  ├────────────────────────────────────────────────────┤
  │  Gradients                ████  ~500 MB            │
  │  (same size as params)                             │
  ├────────────────────────────────────────────────────┤
  │  Temporary buffers        ██  ~200 MB              │
  │  (cuDNN workspace, etc)                            │
  ├────────────────────────────────────────────────────┤
  │  PyTorch cache            ██  ~200 MB              │
  │  (memory allocator cache)                          │
  └────────────────────────────────────────────────────┘
  Total ≈ 4.4 GB
```

### Reducing Memory Usage

```python
# 1. Gradient checkpointing (trade compute for memory)
from torch.utils.checkpoint import checkpoint

class BigModel(nn.Module):
    def __init__(self):
        super().__init__()
        self.block1 = nn.Sequential(nn.Linear(1000, 1000), nn.ReLU())
        self.block2 = nn.Sequential(nn.Linear(1000, 1000), nn.ReLU())
        self.block3 = nn.Sequential(nn.Linear(1000, 10))
    
    def forward(self, x):
        # Don't save activations — recompute during backward
        x = checkpoint(self.block1, x, use_reentrant=False)
        x = checkpoint(self.block2, x, use_reentrant=False)
        x = self.block3(x)
        return x

# 2. Clear cache when switching between training and evaluation
torch.cuda.empty_cache()

# 3. Monitor memory usage
print(f"Allocated: {torch.cuda.memory_allocated() / 1e9:.2f} GB")
print(f"Cached:    {torch.cuda.memory_reserved() / 1e9:.2f} GB")
print(f"Max allocated: {torch.cuda.max_memory_allocated() / 1e9:.2f} GB")

# 4. Delete tensors you no longer need
del intermediate_tensor
torch.cuda.empty_cache()

# 5. Use in-place operations where safe
x = torch.relu_(x)  # in-place (note the underscore)
```

---

## 7. Common GPU Errors

```
  ┌──────────────────────────────────────────────────────────────────────┐
  │  ERROR                          │ CAUSE                │ FIX       │
  ├─────────────────────────────────┼──────────────────────┼───────────┤
  │  CUDA out of memory             │ Batch too large      │ Reduce    │
  │                                 │                      │ batch size│
  │                                 │                      │ Use AMP   │
  │                                 │                      │ Gradient  │
  │                                 │                      │ accum.    │
  ├─────────────────────────────────┼──────────────────────┼───────────┤
  │  RuntimeError: Expected all     │ Model on GPU,        │ Move both │
  │  tensors to be on the same      │ data on CPU          │ to same   │
  │  device                         │ (or vice versa)      │ device    │
  ├─────────────────────────────────┼──────────────────────┼───────────┤
  │  CUDA error: device-side assert │ Invalid index in     │ Check     │
  │  triggered                      │ embedding/gather     │ indices   │
  │                                 │ (e.g., index ≥       │ are valid │
  │                                 │  num_classes)        │           │
  ├─────────────────────────────────┼──────────────────────┼───────────┤
  │  CUDA error: unspecified launch │ NaN in computation   │ Check for │
  │  failure                        │ or kernel error      │ NaN, run  │
  │                                 │                      │ on CPU    │
  ├─────────────────────────────────┼──────────────────────┼───────────┤
  │  RuntimeError: CUDA error:      │ Driver mismatch or   │ Reinstall │
  │  no kernel image available      │ wrong CUDA version   │ PyTorch   │
  │                                 │                      │ for your  │
  │                                 │                      │ CUDA ver  │
  └─────────────────────────────────┴──────────────────────┴───────────┘
```

### Debugging GPU OOM

```python
# Strategy for finding maximum batch size:
def find_max_batch_size(model, input_shape, device, start=1, max_bs=4096):
    """Binary search for max batch size that fits in GPU memory."""
    model = model.to(device)
    low, high = start, max_bs
    best = start
    
    while low <= high:
        mid = (low + high) // 2
        try:
            torch.cuda.empty_cache()
            x = torch.randn(mid, *input_shape, device=device)
            with torch.no_grad():
                _ = model(x)
            del x
            torch.cuda.empty_cache()
            best = mid
            low = mid + 1
        except RuntimeError:
            high = mid - 1
            torch.cuda.empty_cache()
    
    return best

# max_bs = find_max_batch_size(model, (1, 28, 28), device)
# print(f"Maximum batch size: {max_bs}")
```

---

## 8. Worked Example: Full GPU Training Pipeline

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.cuda.amp import autocast, GradScaler
from torch.utils.data import DataLoader
from torchvision import datasets, transforms

# ── Setup ──
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
print(f"Device: {device}")
if torch.cuda.is_available():
    print(f"GPU: {torch.cuda.get_device_name(0)}")
    print(f"Memory: {torch.cuda.get_device_properties(0).total_mem / 1e9:.1f} GB")

# ── Data with pin_memory ──
transform = transforms.Compose([
    transforms.ToTensor(),
    transforms.Normalize((0.1307,), (0.3081,))
])
train_data = datasets.MNIST('./data', train=True, download=True, transform=transform)
train_loader = DataLoader(train_data, batch_size=256, shuffle=True,
                          num_workers=4, pin_memory=True)

# ── Model on GPU ──
model = nn.Sequential(
    nn.Flatten(),
    nn.Linear(784, 512), nn.BatchNorm1d(512), nn.ReLU(), nn.Dropout(0.3),
    nn.Linear(512, 256), nn.BatchNorm1d(256), nn.ReLU(), nn.Dropout(0.3),
    nn.Linear(256, 10)
).to(device)

optimizer = optim.AdamW(model.parameters(), lr=1e-3, weight_decay=1e-4)
criterion = nn.CrossEntropyLoss()
scaler = GradScaler()  # for mixed precision

# ── Training with mixed precision ──
for epoch in range(10):
    model.train()
    total_loss = 0
    correct = 0
    
    for X, y in train_loader:
        X = X.to(device, non_blocking=True)
        y = y.to(device, non_blocking=True)
        
        optimizer.zero_grad()
        
        with autocast():  # mixed precision
            outputs = model(X)
            loss = criterion(outputs, y)
        
        scaler.scale(loss).backward()
        scaler.step(optimizer)
        scaler.update()
        
        total_loss += loss.item() * X.size(0)
        correct += (outputs.argmax(1) == y).sum().item()
    
    acc = correct / len(train_data)
    avg_loss = total_loss / len(train_data)
    mem = torch.cuda.max_memory_allocated() / 1e9
    print(f"Epoch {epoch+1:2d} | Loss: {avg_loss:.4f} | Acc: {acc:.4f} | "
          f"GPU Mem: {mem:.2f} GB")

torch.cuda.reset_peak_memory_stats()
```

---

## 9. Summary Table

| Technique | When to Use | Speedup | Memory Impact |
|-----------|------------|---------|---------------|
| **Single GPU** | Default for most tasks | Baseline | Baseline |
| **pin_memory** | Always (CPU→GPU transfer) | ~10-20% | Negligible |
| **Mixed Precision (AMP)** | Always on modern GPUs | ~1.5-2× | ~50% less |
| **Gradient Accumulation** | Batch doesn't fit in memory | Same | Fits larger effective batch |
| **Gradient Checkpointing** | Very deep models | ~30% slower | ~60% less |
| **DataParallel** | Quick multi-GPU prototype | ~1.5-3× (2-4 GPU) | Same per GPU |
| **DistributedDataParallel** | Serious multi-GPU training | ~3.5-7× (4-8 GPU) | Same per GPU |

---

## 10. Revision Questions

1. **Why are GPUs so much faster than CPUs for neural network training?** Explain in terms of hardware architecture and the nature of neural network computations.

2. **Write the standard PyTorch code pattern for GPU training.** Include device selection, model placement, data transfer, and the training loop.

3. **Compare DataParallel and DistributedDataParallel.** Why is DDP recommended over DP? What is the "GPU 0 bottleneck" problem in DP?

4. **Explain mixed precision training.** What is loss scaling and why is it necessary? Write the PyTorch code using `autocast` and `GradScaler`.

5. **You encounter "CUDA out of memory" error with batch_size=64. List five strategies to reduce GPU memory usage**, in order of preference.

6. **What does `pin_memory=True` do in DataLoader and how does `non_blocking=True` in `.to(device)` help?** Explain the data transfer pipeline from disk to GPU.

---

| [← 03 Framework Comparison](03-framework-comparison.md) | [Unit 9 Home](README.md) | [05 → Model Saving & Loading](05-model-saving-loading.md) |
|:---|:---:|---:|
