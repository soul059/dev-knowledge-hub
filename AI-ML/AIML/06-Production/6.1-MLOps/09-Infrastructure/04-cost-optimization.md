# 💰 Cost Optimization

> **Unit 9 · Infrastructure · Chapter 4**
>
> Strategies for reducing ML infrastructure costs without sacrificing model quality — covering spot instances, right-sizing, auto-scaling, model-level optimizations (quantization, distillation), and cloud cost calculators.

---

## Table of Contents

1. [Chapter Overview](#chapter-overview)
2. [The ML Cost Problem](#the-ml-cost-problem)
3. [Spot & Preemptible Instances](#spot--preemptible-instances)
4. [Right-Sizing Compute](#right-sizing-compute)
5. [Auto-Scaling for Cost Efficiency](#auto-scaling-for-cost-efficiency)
6. [Model Optimization for Cheaper Inference](#model-optimization-for-cheaper-inference)
7. [Cloud Cost Calculators & Budgeting](#cloud-cost-calculators--budgeting)
8. [Cost Optimization Checklist](#cost-optimization-checklist)
9. [Summary](#summary)
10. [Revision Questions](#revision-questions)
11. [Navigation](#navigation)

---

## Chapter Overview

ML workloads are among the most expensive cloud expenditures — a single A100 training cluster can cost **$50,000+/month**. Yet studies consistently show that **30–60 % of ML compute spend is wasted** through over-provisioning, idle resources, and suboptimal model architectures. This chapter provides a systematic framework for reducing costs at every layer: infrastructure (spot, right-sizing, auto-scaling), model (quantization, distillation, pruning), and operational (budgets, alerts, calculators).

---

## The ML Cost Problem

### Where the Money Goes

```
Typical ML Infrastructure Cost Breakdown

Training (40%)     ████████████████░░░░░░░░░░░░░░░░░░░░░░░░
Inference (35%)    ██████████████░░░░░░░░░░░░░░░░░░░░░░░░░░
Storage (10%)      ████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░
Data Transfer (8%) ███░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░
Other (7%)         ██░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░

Note: As models move to production, inference cost often exceeds
      training cost over the model's lifetime.
```

### Common Waste Sources

| Waste Source | Typical Impact | Solution |
|---|---|---|
| Over-provisioned GPUs | 30–50 % waste | Right-size with benchmarks |
| Idle endpoints (nights/weekends) | 60 % of hours | Auto-scale to zero |
| On-demand instead of spot | 60–90 % overpay | Use spot for training |
| Full-precision inference (FP32) | 2–4× excess compute | Quantize to FP16/INT8 |
| No model caching | Redundant computation | Cache frequent predictions |
| Storing all experiment artifacts | Storage bloat | Lifecycle policies |

---

## Spot & Preemptible Instances

### How Spot Instances Work

```
On-Demand Pricing                   Spot Pricing
┌────────────────────┐              ┌────────────────────┐
│                    │              │                    │
│  $3.06/hr (p3.2xl)│              │  $0.92/hr (70% off)│
│                    │              │                    │
│  ✅ Always avail.  │              │  ⚠️  Can be revoked│
│  ✅ No interruption│              │     with 2-min     │
│  ❌ Full price     │              │     warning        │
│                    │              │  ✅ Same hardware   │
└────────────────────┘              └────────────────────┘

Spot Interruption Flow:
                                  ┌──────────────┐
  Training ──▶ ⚠️ 2-min warning ──▶│  Save        │──▶ Resume on
  in progress                     │  Checkpoint  │    new spot
                                  └──────────────┘    instance
```

### Cloud Provider Comparison

| Feature | AWS Spot | GCP Preemptible/Spot | Azure Low-Priority |
|---|---|---|---|
| **Discount** | Up to 90 % | Up to 91 % | Up to 80 % |
| **Warning time** | 2 minutes | 30 seconds | 30 seconds |
| **Max duration** | No limit | 24 hours (preemptible) | No limit |
| **Best for** | Training, batch inference | Training, data processing | Training, batch jobs |

### Implementing Fault-Tolerant Training

```python
# checkpointed_training.py
import os
import torch
import signal
import sys

CHECKPOINT_DIR = "/checkpoints"
CHECKPOINT_PATH = os.path.join(CHECKPOINT_DIR, "latest.pt")

def save_checkpoint(model, optimizer, epoch, step, loss):
    """Save training state for spot instance recovery."""
    torch.save({
        "epoch": epoch,
        "step": step,
        "model_state_dict": model.state_dict(),
        "optimizer_state_dict": optimizer.state_dict(),
        "loss": loss,
    }, CHECKPOINT_PATH)
    print(f"Checkpoint saved: epoch={epoch}, step={step}")

def load_checkpoint(model, optimizer):
    """Resume from latest checkpoint if it exists."""
    if os.path.exists(CHECKPOINT_PATH):
        checkpoint = torch.load(CHECKPOINT_PATH)
        model.load_state_dict(checkpoint["model_state_dict"])
        optimizer.load_state_dict(checkpoint["optimizer_state_dict"])
        print(f"Resumed from epoch={checkpoint['epoch']}, "
              f"step={checkpoint['step']}")
        return checkpoint["epoch"], checkpoint["step"]
    return 0, 0

def handle_spot_termination(signum, frame):
    """Handle spot instance termination signal."""
    print("SPOT TERMINATION SIGNAL — saving checkpoint...")
    save_checkpoint(model, optimizer, current_epoch, current_step, current_loss)
    sys.exit(0)

# Register signal handler for graceful shutdown
signal.signal(signal.SIGTERM, handle_spot_termination)

# Training loop with periodic checkpointing
model = MyModel().cuda()
optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)
start_epoch, start_step = load_checkpoint(model, optimizer)

for epoch in range(start_epoch, num_epochs):
    current_epoch = epoch
    for step, batch in enumerate(dataloader):
        if step < start_step and epoch == start_epoch:
            continue
        current_step = step

        loss = train_step(model, batch)
        current_loss = loss.item()

        # Checkpoint every 500 steps
        if step % 500 == 0:
            save_checkpoint(model, optimizer, epoch, step, current_loss)

    start_step = 0  # reset for next epoch
```

### SageMaker Managed Spot Training

```python
from sagemaker.pytorch import PyTorch

estimator = PyTorch(
    entry_point="train.py",
    instance_type="ml.p3.2xlarge",
    instance_count=1,
    use_spot_instances=True,              # Enable spot
    max_wait=7200,                        # Max time to wait for spot + train
    max_run=3600,                         # Max training time
    checkpoint_s3_uri="s3://bucket/checkpoints/",  # Auto-resume
    checkpoint_local_path="/opt/ml/checkpoints",
)
```

---

## Right-Sizing Compute

### The Right-Sizing Process

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  1. Profile  │───▶│  2. Benchmark│───▶│  3. Select   │───▶│  4. Monitor  │
│  your model  │    │  on multiple │    │  smallest    │    │  & iterate   │
│  (mem, flops)│    │  instances   │    │  that meets  │    │              │
│              │    │              │    │  SLA         │    │              │
└──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
```

### Profiling Script

```python
# profile_model.py
import torch
import time
import numpy as np

def profile_inference(model, input_shape, n_warmup=10, n_runs=100, device="cuda"):
    model = model.to(device).eval()
    dummy = torch.randn(*input_shape).to(device)

    # Warmup
    for _ in range(n_warmup):
        with torch.no_grad():
            model(dummy)
    if device == "cuda":
        torch.cuda.synchronize()

    # Benchmark
    latencies = []
    for _ in range(n_runs):
        start = time.perf_counter()
        with torch.no_grad():
            model(dummy)
        if device == "cuda":
            torch.cuda.synchronize()
        latencies.append((time.perf_counter() - start) * 1000)

    # Memory usage
    if device == "cuda":
        mem_allocated = torch.cuda.max_memory_allocated() / 1e6
    else:
        mem_allocated = 0

    # Model size
    param_count = sum(p.numel() for p in model.parameters())
    model_size_mb = sum(p.numel() * p.element_size() for p in model.parameters()) / 1e6

    return {
        "p50_ms": np.percentile(latencies, 50),
        "p95_ms": np.percentile(latencies, 95),
        "p99_ms": np.percentile(latencies, 99),
        "gpu_memory_mb": mem_allocated,
        "model_size_mb": model_size_mb,
        "param_count": param_count,
    }

# Usage:
# stats = profile_inference(model, input_shape=(1, 3, 224, 224))
# print(stats)
```

### Instance Selection Heuristics

| Model Size | Recommended Instance (AWS) | Approx Cost/hr |
|---|---|---|
| < 100 MB (sklearn, small NN) | `c5.xlarge` (CPU) | $0.17 |
| 100 MB – 1 GB (BERT-base) | `g4dn.xlarge` (T4) | $0.53 |
| 1 – 5 GB (BERT-large, ViT) | `g5.xlarge` (A10G) | $1.01 |
| 5 – 20 GB (LLM 7B quantized) | `g5.2xlarge` (A10G) | $1.21 |
| 20 – 40 GB (LLM 13B) | `p4d.24xlarge` (A100) | $3.91 |
| 40 – 80 GB (LLM 70B) | `p4de.24xlarge` (A100 80GB) | $5.12 |

> **Rule of thumb:** GPU memory should be 1.5–2× the model size (to accommodate activations, KV cache, and framework overhead).

---

## Auto-Scaling for Cost Efficiency

### Scale-to-Zero Architecture

```
                    Traffic Pattern (24 hours)
Requests ▲
         │         ┌──┐
    100  │    ┌────┤  ├────┐
         │   ┌┤    │  │    ├┐
     50  │  ┌┤│    │  │    │├┐
         │ ┌┤││    │  │    │││├┐
      0  │─┤│││    │  │    │││├│──────────────
         └─┴┴┴┴────┴──┴────┴┴┴┴─────────────▶ Time
         12am  6am  12pm  6pm  12am

Without Auto-Scaling:     With Auto-Scaling:
┌───────────────────┐     ┌───────────────────┐
│ 3 pods × 24 hrs   │     │ 0-5 pods dynamic  │
│ Cost: $72/day     │     │ Cost: $28/day     │
│ Night util: ~5%   │     │ Night util: N/A   │
└───────────────────┘     └───────────────────┘
                          Savings: 61%
```

### Kubernetes Auto-Scaling Stack

```yaml
# k8s/autoscaling.yaml

# 1. Horizontal Pod Autoscaler — scale pods
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: model-server-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: model-server
  minReplicas: 0                         # Scale to zero!
  maxReplicas: 10
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 50
          periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0      # Scale up immediately
      policies:
        - type: Pods
          value: 4
          periodSeconds: 60
  metrics:
    - type: External
      external:
        metric:
          name: queue_depth
        target:
          type: AverageValue
          averageValue: "5"
---
# 2. KEDA ScaledObject — for queue-based scaling
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: model-server-scaler
spec:
  scaleTargetRef:
    name: model-server
  minReplicaCount: 0
  maxReplicaCount: 20
  cooldownPeriod: 300
  triggers:
    - type: rabbitmq
      metadata:
        host: "amqp://rabbitmq:5672"
        queueName: inference-queue
        queueLength: "10"              # 1 pod per 10 queued messages
```

### SageMaker Auto-Scaling Config

```python
import boto3

client = boto3.client("application-autoscaling")

# Register scalable target
client.register_scalable_target(
    ServiceNamespace="sagemaker",
    ResourceId="endpoint/my-model-endpoint/variant/AllTraffic",
    ScalableDimension="sagemaker:variant:DesiredInstanceCount",
    MinCapacity=0,       # Scale to zero
    MaxCapacity=10,
)

# Create scaling policy
client.put_scaling_policy(
    PolicyName="inference-scaling",
    ServiceNamespace="sagemaker",
    ResourceId="endpoint/my-model-endpoint/variant/AllTraffic",
    ScalableDimension="sagemaker:variant:DesiredInstanceCount",
    PolicyType="TargetTrackingScaling",
    TargetTrackingScalingPolicyConfiguration={
        "TargetValue": 70.0,                  # target GPU utilization
        "PredefinedMetricSpecification": {
            "PredefinedMetricType": "SageMakerVariantInvocationsPerInstance",
        },
        "ScaleInCooldown": 300,
        "ScaleOutCooldown": 60,
    },
)
```

---

## Model Optimization for Cheaper Inference

### Optimization Techniques Overview

```
Original Model (FP32)
┌────────────────────────────────────────┐
│  Size: 440 MB    Latency: 50 ms       │
│  Cost: $1.00/hr  GPU: A10G required   │
└──────────────────┬─────────────────────┘
                   │
    ┌──────────────┼──────────────┬──────────────────┐
    ▼              ▼              ▼                   ▼
┌──────────┐ ┌──────────┐ ┌──────────┐       ┌──────────────┐
│Quantize  │ │ Distill  │ │  Prune   │       │  ONNX / TRT  │
│(FP16)    │ │(smaller  │ │(remove   │       │  Runtime Opt │
│          │ │ student) │ │ weights) │       │              │
│220 MB    │ │110 MB    │ │330 MB    │       │440→220 MB    │
│25 ms     │ │15 ms     │ │35 ms    │       │20 ms         │
│$0.50/hr  │ │$0.30/hr  │ │$0.70/hr │       │$0.50/hr      │
│<1% acc ↓ │ │2-3% acc↓ │ │1-2% acc↓│       │~0% acc ↓     │
└──────────┘ └──────────┘ └──────────┘       └──────────────┘
```

### 1. Quantization

```python
# quantization.py — Post-training quantization with PyTorch
import torch

# FP16 (Half Precision) — simplest, minimal accuracy loss
model_fp16 = model.half().cuda()

# Dynamic Quantization (CPU) — for linear layers
model_int8 = torch.quantization.quantize_dynamic(
    model.cpu(),
    {torch.nn.Linear},
    dtype=torch.qint8
)

# ONNX Runtime with INT8 quantization
from onnxruntime.quantization import quantize_dynamic, QuantType

quantize_dynamic(
    model_input="model.onnx",
    model_output="model_int8.onnx",
    weight_type=QuantType.QInt8,
)

# Compare sizes
import os
original_size = os.path.getsize("model.onnx") / 1e6
quantized_size = os.path.getsize("model_int8.onnx") / 1e6
print(f"Original:  {original_size:.1f} MB")
print(f"Quantized: {quantized_size:.1f} MB")
print(f"Reduction: {(1 - quantized_size/original_size)*100:.1f}%")
```

### 2. Knowledge Distillation

```python
# distillation.py — Train a smaller student model
import torch
import torch.nn.functional as F

def distillation_loss(student_logits, teacher_logits, labels,
                      temperature=4.0, alpha=0.7):
    """
    Combine soft targets (from teacher) with hard targets (ground truth).
    
    - temperature: higher = softer probability distribution
    - alpha: weight of distillation loss vs task loss
    """
    # Soft target loss (KL divergence between teacher and student)
    soft_loss = F.kl_div(
        F.log_softmax(student_logits / temperature, dim=1),
        F.softmax(teacher_logits / temperature, dim=1),
        reduction="batchmean"
    ) * (temperature ** 2)

    # Hard target loss (standard cross-entropy)
    hard_loss = F.cross_entropy(student_logits, labels)

    return alpha * soft_loss + (1 - alpha) * hard_loss

# Training loop
teacher_model.eval()
student_model.train()

for batch in dataloader:
    inputs, labels = batch

    with torch.no_grad():
        teacher_logits = teacher_model(inputs)

    student_logits = student_model(inputs)
    loss = distillation_loss(student_logits, teacher_logits, labels)

    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
```

### 3. Model Pruning

```python
# pruning.py — Structured pruning with PyTorch
import torch.nn.utils.prune as prune

# Unstructured pruning — remove individual weights
prune.l1_unstructured(model.fc1, name="weight", amount=0.3)  # 30% sparsity

# Structured pruning — remove entire channels/neurons
prune.ln_structured(model.conv1, name="weight", amount=0.2, n=2, dim=0)

# Global pruning — prune across all layers
parameters_to_prune = [
    (model.fc1, "weight"),
    (model.fc2, "weight"),
    (model.fc3, "weight"),
]
prune.global_unstructured(
    parameters_to_prune, pruning_method=prune.L1Unstructured, amount=0.4
)

# Check sparsity
def model_sparsity(model):
    total, zeros = 0, 0
    for name, param in model.named_parameters():
        total += param.numel()
        zeros += (param == 0).sum().item()
    return zeros / total * 100

print(f"Model sparsity: {model_sparsity(model):.1f}%")
```

### Optimization Comparison Table

| Technique | Size Reduction | Speedup | Accuracy Impact | Implementation Effort |
|---|---|---|---|---|
| **FP16 quantization** | 2× | 1.5–2× | < 1 % drop | Very low |
| **INT8 quantization** | 4× | 2–4× | 1–3 % drop | Low |
| **Knowledge distillation** | 4–10× | 3–8× | 2–5 % drop | High (need retraining) |
| **Pruning (30 %)** | 1.3× | 1.1–1.3× | < 1 % drop | Medium |
| **Pruning (70 %)** | 3× | 1.5–2× | 2–5 % drop | Medium |
| **ONNX Runtime** | 1× | 1.5–3× | ~0 % | Low |
| **TensorRT** | 1–4× | 2–5× | < 1 % | Medium |

---

## Cloud Cost Calculators & Budgeting

### Cloud Cost Calculator Links

| Provider | Tool | URL |
|---|---|---|
| **AWS** | Pricing Calculator | `calculator.aws` |
| **GCP** | Pricing Calculator | `cloud.google.com/products/calculator` |
| **Azure** | Pricing Calculator | `azure.microsoft.com/pricing/calculator` |
| **Multi-cloud** | Infracost (Terraform) | `infracost.io` |

### Cost Estimation Script

```python
# cost_estimator.py — Estimate ML workload costs
from dataclasses import dataclass

@dataclass
class InstancePricing:
    name: str
    gpu: str
    on_demand_hourly: float
    spot_hourly: float
    gpu_memory_gb: int

INSTANCES = {
    "g4dn.xlarge":  InstancePricing("g4dn.xlarge",  "T4",   0.526, 0.158, 16),
    "g5.xlarge":    InstancePricing("g5.xlarge",     "A10G", 1.006, 0.302, 24),
    "g5.2xlarge":   InstancePricing("g5.2xlarge",    "A10G", 1.212, 0.364, 24),
    "p4d.24xlarge": InstancePricing("p4d.24xlarge",  "A100", 32.77, 9.83,  320),
}

def estimate_training_cost(instance_type: str, hours: float, use_spot: bool = True):
    inst = INSTANCES[instance_type]
    rate = inst.spot_hourly if use_spot else inst.on_demand_hourly
    cost = rate * hours
    savings = (1 - inst.spot_hourly / inst.on_demand_hourly) * 100 if use_spot else 0
    return {
        "instance": instance_type,
        "gpu": inst.gpu,
        "hours": hours,
        "pricing": "spot" if use_spot else "on-demand",
        "cost": f"${cost:.2f}",
        "savings": f"{savings:.0f}%" if use_spot else "N/A",
    }

def estimate_inference_cost(instance_type: str, hours_per_day: float = 24,
                            days: int = 30):
    inst = INSTANCES[instance_type]
    total_hours = hours_per_day * days
    monthly_cost = inst.on_demand_hourly * total_hours
    return {
        "instance": instance_type,
        "gpu": inst.gpu,
        "hours_per_day": hours_per_day,
        "monthly_cost": f"${monthly_cost:.2f}",
        "annual_cost": f"${monthly_cost * 12:.2f}",
    }

# Examples
print(estimate_training_cost("p4d.24xlarge", hours=100, use_spot=True))
print(estimate_inference_cost("g4dn.xlarge", hours_per_day=16, days=30))
```

### Terraform + Infracost

```hcl
# main.tf
resource "aws_sagemaker_endpoint_configuration" "model" {
  name = "model-endpoint"

  production_variants {
    variant_name           = "primary"
    model_name             = aws_sagemaker_model.model.name
    instance_type          = "ml.g4dn.xlarge"
    initial_instance_count = 2
  }
}

# Run: infracost breakdown --path .
# Output:
# ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━┓
# ┃ Resource                             ┃ Monthly    ┃
# ┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━┫
# ┃ aws_sagemaker_endpoint_config.model  ┃ $768.00    ┃
# ┃   └─ ml.g4dn.xlarge × 2 instances   ┃            ┃
# ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┻━━━━━━━━━━━━┛
```

### Budget Alerts (AWS Example)

```yaml
# cloudformation/budget-alert.yaml
AWSTemplateFormatVersion: "2010-09-09"
Resources:
  MLBudget:
    Type: AWS::Budgets::Budget
    Properties:
      Budget:
        BudgetName: ml-infrastructure-budget
        BudgetLimit:
          Amount: 5000
          Unit: USD
        TimeUnit: MONTHLY
        BudgetType: COST
        CostFilters:
          Service:
            - Amazon SageMaker
            - Amazon EC2
      NotificationsWithSubscribers:
        - Notification:
            NotificationType: ACTUAL
            ComparisonOperator: GREATER_THAN
            Threshold: 80              # Alert at 80% of budget
          Subscribers:
            - SubscriptionType: EMAIL
              Address: ml-team@company.com
        - Notification:
            NotificationType: FORECASTED
            ComparisonOperator: GREATER_THAN
            Threshold: 100             # Alert if forecasted to exceed
          Subscribers:
            - SubscriptionType: EMAIL
              Address: ml-team@company.com
```

---

## Cost Optimization Checklist

```
┌─────────────────────── Cost Optimization Layers ─────────────────────────┐
│                                                                           │
│  Layer 1: Infrastructure                                                 │
│  ┌─────────────────────────────────────────────────────────────────┐      │
│  │ □ Use spot instances for all training jobs                     │      │
│  │ □ Right-size GPU instances (benchmark, don't guess)            │      │
│  │ □ Auto-scale inference endpoints (including scale-to-zero)     │      │
│  │ □ Use reserved instances / committed use for steady workloads  │      │
│  │ □ Clean up idle endpoints, notebooks, and clusters             │      │
│  └─────────────────────────────────────────────────────────────────┘      │
│                                                                           │
│  Layer 2: Model                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐      │
│  │ □ Quantize models (FP16 at minimum, INT8 if accuracy allows)  │      │
│  │ □ Distill large models into smaller student models             │      │
│  │ □ Use ONNX Runtime or TensorRT for inference optimization     │      │
│  │ □ Batch inference requests where latency permits               │      │
│  │ □ Cache frequent predictions                                   │      │
│  └─────────────────────────────────────────────────────────────────┘      │
│                                                                           │
│  Layer 3: Operational                                                    │
│  ┌─────────────────────────────────────────────────────────────────┐      │
│  │ □ Set up budget alerts (80%, 100% thresholds)                  │      │
│  │ □ Tag all resources for cost attribution                       │      │
│  │ □ Review cost reports weekly                                   │      │
│  │ □ Use cost calculators before provisioning                     │      │
│  │ □ Implement S3/GCS lifecycle policies for old artifacts        │      │
│  └─────────────────────────────────────────────────────────────────┘      │
└───────────────────────────────────────────────────────────────────────────┘
```

### Impact Summary

| Strategy | Typical Savings | Risk Level | Effort |
|---|---|---|---|
| Spot instances (training) | 60–90 % | Low (with checkpoints) | Low |
| Scale-to-zero (inference) | 40–70 % | Medium (cold-start latency) | Medium |
| Right-sizing GPUs | 30–60 % | Low | Low |
| FP16 quantization | 30–50 % | Low | Very Low |
| INT8 quantization | 50–75 % | Medium (accuracy) | Low |
| Knowledge distillation | 60–80 % | Medium (retraining) | High |
| Prediction caching | 20–50 % | Low | Low |
| Reserved instances | 30–40 % | Low (commitment) | Very Low |

---

## Summary

| Topic | Key Takeaway |
|---|---|
| **Cost breakdown** | Inference often exceeds training cost over a model's lifetime |
| **Spot instances** | 60–90 % savings; implement checkpointing for fault tolerance |
| **Right-sizing** | Profile first, benchmark on multiple instances, pick the smallest that meets SLA |
| **Auto-scaling** | Scale to zero during idle; use KEDA for queue-based scaling |
| **Quantization** | FP16 is nearly free; INT8 saves 50–75 % with minimal accuracy loss |
| **Distillation** | Highest savings but requires retraining a student model |
| **Pruning** | 30–70 % weight removal; combine with other techniques |
| **Budget alerts** | Set 80 % and 100 % threshold alerts; review weekly |
| **Cost calculators** | Use before provisioning; automate with Infracost for Terraform |

> **The #1 rule of ML cost optimization:** Measure before you optimize. Profile your workload, identify the largest waste source, and fix that first.

---

## Revision Questions

1. **Explain** how spot instances work for ML training. What safeguards must you implement, and what is the expected cost savings?

2. **A model serving endpoint** costs $2,400/month running 24/7 but only receives traffic 10 hours/day. Calculate the potential savings with scale-to-zero auto-scaling, accounting for cold-start considerations.

3. **Compare** FP16 quantization, INT8 quantization, and knowledge distillation in terms of size reduction, speedup, accuracy impact, and implementation effort.

4. **Design** a cost monitoring and alerting strategy for an ML team running 5 training jobs/week and 3 always-on inference endpoints. What metrics, thresholds, and tools would you use?

5. **You discover** that your team's GPU utilization averages 25 % across all inference endpoints. List four specific actions you would take to reduce costs, ordered by impact and implementation effort.

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← GPU Management](./03-gpu-management.md) | [📂 Infrastructure](./README.md) | [Scaling Strategies →](./05-scaling-strategies.md) |

---

> **Next up:** [Chapter 5 — Scaling Strategies](./05-scaling-strategies.md) — Horizontal vs vertical scaling, load balancing, prediction caching, and batch processing optimization.
