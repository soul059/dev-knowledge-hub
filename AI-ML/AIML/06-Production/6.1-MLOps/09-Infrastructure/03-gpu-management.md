# 🎮 GPU Management

> **Unit 9 · Infrastructure · Chapter 3**
>
> Everything an ML engineer needs to know about GPU infrastructure — when to use GPUs vs CPUs, NVIDIA sharing techniques (MIG, time-slicing), monitoring with `nvidia-smi`, and optimizing cost-performance tradeoffs.

---

## Table of Contents

1. [Chapter Overview](#chapter-overview)
2. [GPU vs CPU for ML Workloads](#gpu-vs-cpu-for-ml-workloads)
3. [GPU Hardware Landscape](#gpu-hardware-landscape)
4. [NVIDIA GPU Sharing](#nvidia-gpu-sharing)
5. [GPU Monitoring](#gpu-monitoring)
6. [Cost-Performance Tradeoffs](#cost-performance-tradeoffs)
7. [GPU Management in Kubernetes](#gpu-management-in-kubernetes)
8. [Code Examples](#code-examples)
9. [Summary](#summary)
10. [Revision Questions](#revision-questions)
11. [Navigation](#navigation)

---

## Chapter Overview

GPUs are the most expensive component in any ML infrastructure stack, yet they are often **under-utilized** — studies show average GPU utilization in ML clusters is only 30–50 %. This chapter covers the fundamentals of GPU management: knowing when a GPU is warranted, choosing the right GPU tier, sharing GPUs across workloads, monitoring utilization, and making smart cost-performance decisions.

---

## GPU vs CPU for ML Workloads

### Why GPUs Are Faster for ML

```
CPU (Sequential)                    GPU (Parallel)
┌─────┐                            ┌──┬──┬──┬──┬──┬──┬──┬──┐
│Core │ → Process one operation    │C1│C2│C3│C4│C5│C6│C7│C8│  (thousands
│  1  │   at a time (per core)     ├──┼──┼──┼──┼──┼──┼──┼──┤   of CUDA
│     │                            │C9│10│11│12│13│14│15│16│   cores)
└─────┘                            └──┴──┴──┴──┴──┴──┴──┴──┘

  8-64 powerful cores               Thousands of simpler cores
  Great for branching logic         Great for matrix multiplication
  Low latency per task              High throughput for parallel tasks
```

### Decision Matrix: GPU vs CPU

| Factor | Use **CPU** | Use **GPU** |
|---|---|---|
| **Model type** | Sklearn, XGBoost, small NNs | Deep learning, transformers, CNNs |
| **Batch size** | Small (< 32) | Large (≥ 32) |
| **Tensor operations** | Minimal | Heavy (matmul, convolutions) |
| **Latency requirement** | < 5 ms (CPU may be faster for tiny models) | Throughput matters more than single-request latency |
| **Cost sensitivity** | High (CPUs are 5–10× cheaper) | Model performance justifies cost |
| **Inference volume** | Low (< 100 req/s) | High (> 100 req/s for DL models) |

### Latency Comparison Example

```
Model: BERT-base (110M parameters)
Input: Single sentence (128 tokens)

┌──────────────────────┬──────────┬──────────┐
│ Hardware             │ Latency  │ Cost/hr  │
├──────────────────────┼──────────┼──────────┤
│ CPU (c5.2xlarge)     │  ~120 ms │  $0.34   │
│ GPU T4 (g4dn.xl)    │   ~12 ms │  $0.53   │
│ GPU A10G (g5.xl)    │    ~6 ms │  $1.01   │
│ GPU A100 (p4d.xl)   │    ~3 ms │  $3.91   │
└──────────────────────┴──────────┴──────────┘

* GPU gives 10-40× speedup but at 1.5-11× the cost.
  The right choice depends on your latency SLA and budget.
```

---

## GPU Hardware Landscape

### NVIDIA GPU Tiers for ML

```
                          Performance
                              ▲
                              │
          H100 (80GB) ───────●                    Data Center
                              │                   (Training + Inference)
          A100 (80GB) ──────●│
          A100 (40GB) ─────●─┤
                              │
          L40S (48GB) ────●──┤                    Inference +
          A10G (24GB) ───●───┤                    Light Training
                              │
          T4   (16GB) ──●────┤                    Budget Inference
                              │
          L4   (24GB) ─●─────┤                    Efficient Inference
                              │
                              └──────────────────▶ Cost
                          Low                  High
```

### GPU Comparison Table

| GPU | VRAM | FP16 TFLOPS | Use Case | Cloud Cost (approx/hr) |
|---|---|---|---|---|
| **T4** | 16 GB | 65 | Budget inference, small models | $0.50–0.75 |
| **L4** | 24 GB | 121 | Efficient inference, video | $0.70–1.00 |
| **A10G** | 24 GB | 125 | Balanced inference + fine-tuning | $1.00–1.50 |
| **L40S** | 48 GB | 362 | Large model inference | $1.50–2.50 |
| **A100 40GB** | 40 GB | 312 | Training + large inference | $3.00–4.00 |
| **A100 80GB** | 80 GB | 312 | Large model training | $4.00–5.00 |
| **H100** | 80 GB | 990 | Frontier model training | $8.00–12.00 |

### Key Specs That Matter

| Spec | Why It Matters |
|---|---|
| **VRAM (GPU Memory)** | Determines max model size that fits in memory |
| **TFLOPS (FP16/BF16)** | Raw compute speed for training/inference |
| **Memory bandwidth** | How fast data moves to/from GPU cores |
| **NVLink / NVSwitch** | Multi-GPU communication speed for distributed training |
| **TDP (Power)** | Electricity cost and cooling requirements |
| **MIG support** | Whether the GPU can be partitioned (A100, H100 only) |

---

## NVIDIA GPU Sharing

### The Problem

```
Without Sharing:                    With Sharing:
┌────────────────────┐              ┌────────────────────┐
│     A100 GPU       │              │     A100 GPU       │
│                    │              │ ┌──────┬──────┐    │
│   Model A          │              │ │Model │Model │    │
│   (uses 10GB /     │              │ │  A   │  B   │    │
│    80GB = 12%)     │              │ │ 10GB │ 15GB │    │
│                    │              │ ├──────┼──────┤    │
│   68 GB WASTED!    │              │ │Model │Model │    │
│                    │              │ │  C   │  D   │    │
└────────────────────┘              │ │ 20GB │ 10GB │    │
                                    │ └──────┴──────┘    │
                                    │   69% utilized!    │
                                    └────────────────────┘
```

### Three Sharing Approaches

#### 1. Multi-Instance GPU (MIG) — Hardware Partitioning

```
┌────────────────────────── A100 80GB with MIG ──────────────────────────┐
│                                                                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                 │
│  │  MIG Slice   │  │  MIG Slice   │  │  MIG Slice   │                 │
│  │  3g.40gb     │  │  2g.20gb     │  │  2g.20gb     │                 │
│  │              │  │              │  │              │                 │
│  │  Model A     │  │  Model B     │  │  Model C     │                 │
│  │  (Training)  │  │  (Inference) │  │  (Inference) │                 │
│  │              │  │              │  │              │                 │
│  │ Isolated     │  │ Isolated     │  │ Isolated     │                 │
│  │ memory +     │  │ memory +     │  │ memory +     │                 │
│  │ compute      │  │ compute      │  │ compute      │                 │
│  └──────────────┘  └──────────────┘  └──────────────┘                 │
│                                                                        │
│  ✅ Full isolation (memory, compute, error containment)                │
│  ❌ Only A100 and H100 support MIG                                    │
│  ❌ Fixed partition sizes (7 predefined profiles)                     │
└────────────────────────────────────────────────────────────────────────┘
```

**MIG Profiles for A100 80GB:**

| Profile | GPU Memory | Compute (SMs) | Use Case |
|---|---|---|---|
| `1g.10gb` | 10 GB | 1/7 | Small inference workloads |
| `2g.20gb` | 20 GB | 2/7 | Medium model inference |
| `3g.40gb` | 40 GB | 3/7 | Large inference or fine-tuning |
| `4g.40gb` | 40 GB | 4/7 | Training medium models |
| `7g.80gb` | 80 GB | 7/7 | Full GPU (no sharing) |

```bash
# Enable MIG mode
sudo nvidia-smi -i 0 -mig 1

# Create MIG instances
sudo nvidia-smi mig -i 0 -cgi 9,14,14 -C
#   9  = 3g.40gb
#   14 = 2g.20gb (×2)

# List MIG devices
nvidia-smi mig -lgip
nvidia-smi mig -lgi

# Destroy MIG instances
sudo nvidia-smi mig -i 0 -dci
sudo nvidia-smi mig -i 0 -dgi
```

#### 2. Time-Slicing — Software Sharing

```
┌──────────────────── T4 GPU with Time-Slicing ────────────────────────┐
│                                                                       │
│  Time ──▶  ┌────┐┌────┐┌────┐┌────┐┌────┐┌────┐┌────┐              │
│            │ A  ││ B  ││ C  ││ A  ││ B  ││ C  ││ A  │ ...          │
│            └────┘└────┘└────┘└────┘└────┘└────┘└────┘              │
│                                                                       │
│  3 models share 1 GPU via context switching                          │
│                                                                       │
│  ✅ Works on ALL NVIDIA GPUs                                         │
│  ✅ Simple to configure                                              │
│  ❌ No memory isolation (OOM risk)                                   │
│  ❌ Context switching overhead (10-20%)                              │
└──────────────────────────────────────────────────────────────────────┘
```

**Time-slicing config for the NVIDIA GPU operator:**

```yaml
# gpu-time-slicing-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: time-slicing-config
  namespace: gpu-operator
data:
  any: |-
    version: v1
    sharing:
      timeSlicing:
        renameByDefault: false
        failRequestsGreaterThanOne: false
        resources:
          - name: nvidia.com/gpu
            replicas: 4    # each physical GPU appears as 4 virtual GPUs
```

#### 3. Multi-Process Service (MPS) — Process-Level Sharing

```
┌──────────────────── GPU with MPS ────────────────────────────────────┐
│                                                                       │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                           │
│  │ Process A│  │ Process B│  │ Process C│                           │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘                           │
│       │              │              │                                 │
│       └──────────────┼──────────────┘                                 │
│                      ▼                                                │
│              ┌──────────────┐                                         │
│              │  MPS Server  │    Multiplexes CUDA contexts            │
│              └──────┬───────┘                                         │
│                     ▼                                                 │
│              ┌──────────────┐                                         │
│              │   GPU        │                                         │
│              └──────────────┘                                         │
│                                                                       │
│  ✅ True concurrent execution (no time-slicing overhead)             │
│  ✅ Better utilization than time-slicing                             │
│  ❌ No error isolation (one crash affects all)                       │
│  ❌ Limited memory protection                                        │
└──────────────────────────────────────────────────────────────────────┘
```

### Sharing Comparison

| Feature | MIG | Time-Slicing | MPS |
|---|---|---|---|
| **Memory isolation** | ✅ Full | ❌ None | ❌ Partial |
| **Compute isolation** | ✅ Full | ✅ Time-divided | ❌ Shared |
| **Error isolation** | ✅ Full | ❌ None | ❌ None |
| **GPU support** | A100, H100 only | All NVIDIA GPUs | All NVIDIA GPUs |
| **Overhead** | None | 10–20 % | ~5 % |
| **Max partitions** | 7 (A100) | Configurable | Configurable |
| **Best for** | Multi-tenant prod | Dev/test, low-priority | Batch inference |

---

## GPU Monitoring

### nvidia-smi — Your Primary Tool

```bash
# Basic GPU status
nvidia-smi

# Sample output:
# +-----------------------------------------------------------------------------+
# | NVIDIA-SMI 535.104.05   Driver Version: 535.104.05   CUDA Version: 12.2     |
# |-------------------------------+----------------------+----------------------+
# | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
# | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
# |===============================+======================+======================|
# |   0  NVIDIA A100-SXM...  On  | 00000000:00:04.0 Off |                    0 |
# | N/A   42C    P0    62W / 400W |  12345MiB / 81920MiB |     67%      Default |
# +-------------------------------+----------------------+----------------------+
```

### Key nvidia-smi Commands

```bash
# Continuous monitoring (refresh every 1 second)
nvidia-smi dmon -s pucvmet -d 1

# Query specific metrics in CSV format
nvidia-smi --query-gpu=timestamp,name,temperature.gpu,utilization.gpu,\
utilization.memory,memory.used,memory.total,power.draw \
--format=csv -l 5

# Show running processes
nvidia-smi pmon -s um -d 1

# Check MIG status
nvidia-smi mig -lgip

# Reset GPU (use with caution)
nvidia-smi -r -i 0
```

### Critical Metrics to Monitor

```
┌──────────────────── GPU Monitoring Dashboard ────────────────────────┐
│                                                                       │
│  GPU Utilization (%)     Memory Usage (GB)      Temperature (°C)     │
│  ┌────────────────┐      ┌────────────────┐     ┌────────────────┐   │
│  │████████░░ 78%  │      │██████░░░ 48/80 │     │███░░░░░░ 52°C │   │
│  └────────────────┘      └────────────────┘     └────────────────┘   │
│                                                                       │
│  Power Draw (W)          SM Clock (MHz)         PCIe Throughput       │
│  ┌────────────────┐      ┌────────────────┐     ┌────────────────┐   │
│  │██████░░ 280/400│      │████████░ 1410  │     │████░░░░ 12GB/s │   │
│  └────────────────┘      └────────────────┘     └────────────────┘   │
│                                                                       │
│  Alert Thresholds:                                                    │
│  🔴 GPU Util < 30% for 10 min  → Under-utilized (waste)             │
│  🔴 Memory > 90%               → OOM risk                           │
│  🔴 Temperature > 85°C         → Thermal throttling                 │
│  🟡 Power > 90% TDP            → Near power limit                   │
└──────────────────────────────────────────────────────────────────────┘
```

| Metric | Healthy Range | Action If Out of Range |
|---|---|---|
| **GPU Utilization** | 60–95 % | < 30 %: downsize or share; > 95 %: scale out |
| **Memory Utilization** | 50–85 % | > 90 %: reduce batch size or upgrade GPU |
| **Temperature** | 30–80 °C | > 85 °C: check cooling, reduce workload |
| **Power Draw** | 50–90 % TDP | > 95 %: thermal throttling risk |
| **Memory Clock** | Near max | Low clock = power-saving mode (idle GPU) |

### Prometheus + Grafana Monitoring

```python
# gpu_monitor.py — export GPU metrics to Prometheus
import subprocess
import time
from prometheus_client import Gauge, start_http_server

GPU_UTIL = Gauge("gpu_utilization_percent", "GPU utilization", ["gpu_id"])
GPU_MEM_USED = Gauge("gpu_memory_used_mb", "GPU memory used", ["gpu_id"])
GPU_MEM_TOTAL = Gauge("gpu_memory_total_mb", "GPU memory total", ["gpu_id"])
GPU_TEMP = Gauge("gpu_temperature_celsius", "GPU temperature", ["gpu_id"])
GPU_POWER = Gauge("gpu_power_draw_watts", "GPU power draw", ["gpu_id"])

def collect_gpu_metrics():
    result = subprocess.run(
        ["nvidia-smi",
         "--query-gpu=index,utilization.gpu,memory.used,memory.total,"
         "temperature.gpu,power.draw",
         "--format=csv,noheader,nounits"],
        capture_output=True, text=True
    )
    for line in result.stdout.strip().split("\n"):
        idx, util, mem_used, mem_total, temp, power = line.split(", ")
        GPU_UTIL.labels(gpu_id=idx).set(float(util))
        GPU_MEM_USED.labels(gpu_id=idx).set(float(mem_used))
        GPU_MEM_TOTAL.labels(gpu_id=idx).set(float(mem_total))
        GPU_TEMP.labels(gpu_id=idx).set(float(temp))
        GPU_POWER.labels(gpu_id=idx).set(float(power))

if __name__ == "__main__":
    start_http_server(9400)
    while True:
        collect_gpu_metrics()
        time.sleep(15)
```

---

## Cost-Performance Tradeoffs

### Cost Efficiency Analysis

```
Cost per 1M Inferences (BERT-base, batch=1)

T4        ██████░░░░░░░░░░░░░░  $8.50   ← Best cost for small models
L4        ████░░░░░░░░░░░░░░░░  $5.80   ← Best efficiency for inference
A10G      █████░░░░░░░░░░░░░░░  $7.20
A100      ████████████░░░░░░░░  $16.40  ← Overkill for inference
CPU       ██████████████████░░  $24.00  ← Worst cost for DL inference

Cost per Training Hour (ResNet-50, ImageNet)

T4        ████████████████████  $12.00/epoch  ← Too slow
A10G      ██████████░░░░░░░░░░  $6.80/epoch
A100      ████░░░░░░░░░░░░░░░░  $3.20/epoch   ← Best for training
H100      ███░░░░░░░░░░░░░░░░░  $2.10/epoch   ← Fastest
```

### GPU Selection Guide

```
                    ┌──────────────────────┐
                    │  What's your task?   │
                    └──────────┬───────────┘
                               │
                 ┌─────────────┼─────────────┐
                 ▼                           ▼
          ┌──────────────┐            ┌──────────────┐
          │  Training    │            │  Inference   │
          └──────┬───────┘            └──────┬───────┘
                 │                           │
         ┌───────┼───────┐           ┌───────┼───────┐
         ▼               ▼           ▼               ▼
    ┌─────────┐    ┌─────────┐  ┌─────────┐   ┌──────────┐
    │ < 10B   │    │ > 10B   │  │ < 1B    │   │ > 1B     │
    │ params  │    │ params  │  │ params  │   │ params   │
    └────┬────┘    └────┬────┘  └────┬────┘   └────┬─────┘
         │              │           │              │
         ▼              ▼           ▼              ▼
    ┌─────────┐  ┌──────────┐  ┌─────────┐  ┌──────────┐
    │ A100    │  │ H100 ×8  │  │ T4 / L4 │  │ A100 /   │
    │ 40/80GB │  │ cluster  │  │         │  │ L40S     │
    └─────────┘  └──────────┘  └─────────┘  └──────────┘
```

### Optimization Strategies

| Strategy | Savings | Implementation Effort |
|---|---|---|
| **Right-size GPU** | 30–60 % | Low (benchmark different GPUs) |
| **Spot / preemptible** | 60–90 % | Medium (checkpointing required) |
| **GPU sharing (MIG)** | 40–70 % | Medium (reconfigure cluster) |
| **Time-slicing** | 30–50 % | Low (ConfigMap change) |
| **Model quantization** | 50–75 % | Medium (FP16/INT8 conversion) |
| **Batch inference** | 40–60 % | Low (increase batch size) |
| **Scale to zero** | Up to 100 % idle cost | Medium (serverless inference) |

---

## GPU Management in Kubernetes

### NVIDIA GPU Operator

```yaml
# Install via Helm
# helm install gpu-operator nvidia/gpu-operator \
#   --namespace gpu-operator --create-namespace \
#   --set driver.enabled=true \
#   --set toolkit.enabled=true \
#   --set devicePlugin.enabled=true \
#   --set migManager.enabled=true

# Verify GPU availability
# kubectl describe nodes | grep nvidia.com/gpu
```

### Node Labels for GPU Scheduling

```yaml
# Node labels (auto-applied by GPU feature discovery)
# nvidia.com/gpu.product=NVIDIA-A100-SXM4-80GB
# nvidia.com/gpu.memory=81920
# nvidia.com/mig.strategy=single
# nvidia.com/gpu.count=8

# Use in Pod spec
nodeSelector:
  nvidia.com/gpu.product: NVIDIA-A100-SXM4-80GB

# Or use node affinity for flexibility
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: nvidia.com/gpu.memory
              operator: Gt
              values: ["40000"]   # >= 40GB VRAM
```

### MIG in Kubernetes

```yaml
# Pod requesting a specific MIG slice
apiVersion: v1
kind: Pod
metadata:
  name: inference-small
spec:
  containers:
    - name: model
      image: my-model:latest
      resources:
        limits:
          nvidia.com/mig-2g.20gb: 1    # Request a 2g.20gb MIG slice
```

---

## Code Examples

### Check GPU Availability in PyTorch

```python
import torch

def print_gpu_info():
    if not torch.cuda.is_available():
        print("No GPU available, using CPU")
        return

    n_gpus = torch.cuda.device_count()
    print(f"Available GPUs: {n_gpus}")

    for i in range(n_gpus):
        props = torch.cuda.get_device_properties(i)
        total_mem = props.total_mem / 1e9
        print(f"  GPU {i}: {props.name}")
        print(f"    Memory: {total_mem:.1f} GB")
        print(f"    Compute Capability: {props.major}.{props.minor}")
        print(f"    Multi-Processors: {props.multi_processor_count}")

    # Current memory usage
    allocated = torch.cuda.memory_allocated() / 1e9
    reserved = torch.cuda.memory_reserved() / 1e9
    print(f"\nMemory Allocated: {allocated:.2f} GB")
    print(f"Memory Reserved:  {reserved:.2f} GB")

print_gpu_info()
```

### GPU Memory-Aware Model Loading

```python
import torch
import gc

def load_model_to_best_device(model_class, model_path, min_gpu_memory_gb=4):
    """Load model to GPU if sufficient memory, else CPU."""
    device = "cpu"

    if torch.cuda.is_available():
        gpu_mem = torch.cuda.get_device_properties(0).total_mem / 1e9
        allocated = torch.cuda.memory_allocated(0) / 1e9
        free = gpu_mem - allocated

        if free >= min_gpu_memory_gb:
            device = "cuda:0"
            print(f"Loading to GPU ({free:.1f} GB free)")
        else:
            print(f"Insufficient GPU memory ({free:.1f} GB free, "
                  f"need {min_gpu_memory_gb} GB). Using CPU.")

    model = model_class()
    state_dict = torch.load(model_path, map_location=device)
    model.load_state_dict(state_dict)
    model.to(device)
    model.eval()
    return model, device

def clear_gpu_memory():
    """Force GPU memory cleanup."""
    if torch.cuda.is_available():
        torch.cuda.empty_cache()
        gc.collect()
        print(f"GPU memory after cleanup: "
              f"{torch.cuda.memory_allocated()/1e9:.2f} GB allocated")
```

### GPU Utilization Tracker

```python
import threading
import time
import subprocess
from dataclasses import dataclass

@dataclass
class GPUSnapshot:
    timestamp: float
    gpu_util: float
    mem_used_mb: float
    mem_total_mb: float
    temperature: float

class GPUTracker:
    """Track GPU utilization during training/inference."""

    def __init__(self, interval_seconds=5):
        self.interval = interval_seconds
        self.snapshots: list[GPUSnapshot] = []
        self._running = False

    def _collect(self):
        while self._running:
            result = subprocess.run(
                ["nvidia-smi",
                 "--query-gpu=utilization.gpu,memory.used,memory.total,"
                 "temperature.gpu",
                 "--format=csv,noheader,nounits"],
                capture_output=True, text=True
            )
            if result.returncode == 0:
                parts = result.stdout.strip().split(", ")
                self.snapshots.append(GPUSnapshot(
                    timestamp=time.time(),
                    gpu_util=float(parts[0]),
                    mem_used_mb=float(parts[1]),
                    mem_total_mb=float(parts[2]),
                    temperature=float(parts[3]),
                ))
            time.sleep(self.interval)

    def start(self):
        self._running = True
        self._thread = threading.Thread(target=self._collect, daemon=True)
        self._thread.start()

    def stop(self):
        self._running = False
        self._thread.join()

    def summary(self):
        if not self.snapshots:
            return "No data collected"
        utils = [s.gpu_util for s in self.snapshots]
        mems = [s.mem_used_mb / s.mem_total_mb * 100 for s in self.snapshots]
        return {
            "avg_gpu_util": sum(utils) / len(utils),
            "max_gpu_util": max(utils),
            "avg_mem_util": sum(mems) / len(mems),
            "max_mem_util": max(mems),
            "samples": len(self.snapshots),
        }

# Usage:
# tracker = GPUTracker(interval_seconds=2)
# tracker.start()
# ... run training/inference ...
# tracker.stop()
# print(tracker.summary())
```

---

## Summary

| Topic | Key Takeaway |
|---|---|
| **GPU vs CPU** | Use GPU for deep learning; CPU for traditional ML and tiny models |
| **GPU selection** | Match VRAM to model size; don't over-provision |
| **MIG** | Hardware-level isolation on A100/H100; best for multi-tenant prod |
| **Time-slicing** | Software sharing on any GPU; simple but no isolation |
| **MPS** | Concurrent execution; lower overhead but no error isolation |
| **nvidia-smi** | Primary monitoring tool; track util, memory, temp, power |
| **Cost optimization** | Right-size → spot instances → sharing → quantization |
| **K8s GPU scheduling** | Use NVIDIA device plugin; GPUs are non-sharable by default |
| **Monitoring** | Alert on < 30 % util (waste) and > 90 % memory (OOM risk) |

> **Golden rule:** Monitor GPU utilization. If average util is below 50 %, you are either over-provisioned or should implement GPU sharing.

---

## Revision Questions

1. **When would you choose** CPU over GPU for model inference? Give three specific scenarios with justification.

2. **Compare and contrast** MIG, time-slicing, and MPS for GPU sharing. Which would you choose for a multi-tenant Kubernetes cluster running production inference?

3. **You have an A100 80GB** running a single model that uses 15 GB of VRAM and 20 % compute. How would you improve utilization? Propose a specific MIG configuration.

4. **Describe** the key metrics you would monitor for a GPU-based inference service. What alert thresholds would you set and why?

5. **A team** is choosing between T4 and A100 GPUs for serving a BERT-base model at 500 requests/second. Analyze the cost-performance tradeoff and make a recommendation.

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Kubernetes for ML](./02-kubernetes-for-ml.md) | [📂 Infrastructure](./README.md) | [Cost Optimization →](./04-cost-optimization.md) |

---

> **Next up:** [Chapter 4 — Cost Optimization](./04-cost-optimization.md) — Spot instances, right-sizing, auto-scaling, and model-level optimizations for cheaper inference.
