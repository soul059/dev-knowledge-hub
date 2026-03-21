# 📈 Scaling Strategies

> **Unit 9 · Infrastructure · Chapter 5**
>
> How to scale ML systems to handle growing traffic — horizontal vs vertical scaling, auto-scaling by queue depth, load balancing for inference, prediction caching, and batch processing optimization.

---

## Table of Contents

1. [Chapter Overview](#chapter-overview)
2. [Horizontal vs Vertical Scaling](#horizontal-vs-vertical-scaling)
3. [Auto-Scaling Based on Queue Depth](#auto-scaling-based-on-queue-depth)
4. [Load Balancing for ML](#load-balancing-for-ml)
5. [Caching Predictions](#caching-predictions)
6. [Batch Processing Optimization](#batch-processing-optimization)
7. [End-to-End Scaling Architecture](#end-to-end-scaling-architecture)
8. [Summary](#summary)
9. [Revision Questions](#revision-questions)
10. [Navigation](#navigation)

---

## Chapter Overview

An ML system that works in a notebook breaks at 10 requests/second, and one that works at 10 req/s fails at 10,000. **Scaling** is the discipline of making systems handle growing load gracefully. This chapter covers five core techniques: choosing between horizontal and vertical scaling, auto-scaling based on queue depth (not just CPU), intelligent load balancing for GPU-backed services, caching predictions to avoid redundant computation, and optimizing batch inference for throughput.

---

## Horizontal vs Vertical Scaling

### Comparison

```
Vertical Scaling (Scale Up)          Horizontal Scaling (Scale Out)
┌──────────────────────┐             ┌───────┐ ┌───────┐ ┌───────┐
│                      │             │ Pod 1 │ │ Pod 2 │ │ Pod 3 │
│    BIG SERVER        │             │ Small │ │ Small │ │ Small │
│    64 CPU, 512GB     │             │       │ │       │ │       │
│    8× A100 GPU       │             └───┬───┘ └───┬───┘ └───┬───┘
│                      │                 │         │         │
│    Single point      │                 └─────────┼─────────┘
│    of failure        │                       Load Balancer
└──────────────────────┘                 Fault tolerant, elastic
```

### Decision Matrix

| Factor | Vertical (Scale Up) | Horizontal (Scale Out) |
|---|---|---|
| **How it works** | Bigger machine | More machines |
| **Limit** | Hardware ceiling | Near-infinite |
| **Fault tolerance** | Single point of failure | Survives pod/node failures |
| **Cost curve** | Exponential (bigger = much more $) | Linear (proportional to replicas) |
| **Complexity** | Low (no coordination) | Higher (load balancing, state) |
| **Best for** | Single large model that won't shard | Stateless inference services |
| **Scaling speed** | Slow (new machine) | Fast (add pods in seconds) |

### When to Use Each

```
                      ┌────────────────────────┐
                      │  Can the model fit on   │
                      │  a single GPU/machine?  │
                      └──────────┬─────────────┘
                                 │
                    Yes ─────────┼───────── No
                    │                       │
                    ▼                       ▼
           ┌────────────────┐      ┌────────────────┐
           │  Horizontal    │      │  Vertical first │
           │  Scaling       │      │  (bigger GPU)   │
           │  (add replicas)│      │  then model     │
           │                │      │  parallelism    │
           └────────────────┘      └────────────────┘

Examples:
  Horizontal: BERT serving, recommendation APIs, sklearn models
  Vertical:   LLM 70B inference, large embedding models
  Both:       LLM 70B on multi-GPU nodes, scaled horizontally
```

### Horizontal Scaling Implementation

```yaml
# k8s/horizontal-scaling.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: model-server
spec:
  replicas: 3                         # Start with 3
  selector:
    matchLabels:
      app: model-server
  template:
    metadata:
      labels:
        app: model-server
    spec:
      containers:
        - name: model
          image: registry.example.com/model:v2
          ports:
            - containerPort: 8080
          resources:
            requests:
              cpu: "2"
              memory: "4Gi"
              nvidia.com/gpu: "1"
            limits:
              cpu: "2"
              memory: "4Gi"
              nvidia.com/gpu: "1"
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: model-server-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: model-server
  minReplicas: 2
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 60
```

### Vertical Scaling Implementation

```yaml
# k8s/vertical-scaling.yaml
# Vertical Pod Autoscaler recommends or auto-sets resource requests
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: model-server-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: model-server
  updatePolicy:
    updateMode: "Auto"          # or "Off" for recommendations only
  resourcePolicy:
    containerPolicies:
      - containerName: model
        minAllowed:
          cpu: "1"
          memory: "2Gi"
        maxAllowed:
          cpu: "8"
          memory: "32Gi"
        controlledResources: ["cpu", "memory"]
```

---

## Auto-Scaling Based on Queue Depth

### Why CPU-Based Scaling Fails for ML

```
Problem: GPU inference — CPU is idle, GPU is saturated

CPU Utilization:    ██░░░░░░░░░░  15%   ← HPA sees "low load"
GPU Utilization:    █████████░░░  85%   ← Actually overloaded!
Request Latency:    ████████████  500ms ← Users are suffering

CPU-based HPA won't scale up because CPU is low.
Queue-based scaling solves this.
```

### Queue-Based Architecture

```
┌──────────┐    ┌──────────────┐    ┌──────────────────────┐
│  Client  │───▶│  Message     │───▶│  Worker Pods         │
│  Request │    │  Queue       │    │  (GPU Inference)     │
└──────────┘    │              │    │                      │
                │  depth = 47  │    │  ┌────┐ ┌────┐      │
                │              │    │  │ W1 │ │ W2 │      │
                └──────┬───────┘    │  └────┘ └────┘      │
                       │            │        ... scale     │
                       │            └──────────────────────┘
                       │                      ▲
                       │                      │
                ┌──────▼───────┐              │
                │   KEDA       │──── scale ───┘
                │   Scaler     │
                │              │
                │   if depth   │
                │   > 10/pod:  │
                │   add pods   │
                └──────────────┘
```

### KEDA Configuration

```yaml
# k8s/keda-scaling.yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: inference-worker-scaler
spec:
  scaleTargetRef:
    name: inference-worker
  pollingInterval: 15                  # Check queue every 15s
  cooldownPeriod: 300                  # Wait 5 min before scaling down
  minReplicaCount: 1                   # Keep at least 1 warm
  maxReplicaCount: 50
  triggers:
    # Scale based on RabbitMQ queue depth
    - type: rabbitmq
      metadata:
        host: "amqp://user:pass@rabbitmq.default.svc:5672"
        queueName: inference-queue
        mode: QueueLength
        value: "10"                    # 1 pod per 10 pending messages

    # OR: Scale based on Redis list length
    # - type: redis
    #   metadata:
    #     address: redis.default.svc:6379
    #     listName: inference-queue
    #     listLength: "15"

    # OR: Scale based on Prometheus metric
    # - type: prometheus
    #   metadata:
    #     serverAddress: http://prometheus:9090
    #     metricName: inference_queue_depth
    #     query: sum(inference_pending_requests)
    #     threshold: "20"
---
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: gpu-worker-scaler
spec:
  scaleTargetRef:
    name: gpu-inference-worker
  minReplicaCount: 0                   # Scale to zero when idle
  maxReplicaCount: 20
  triggers:
    - type: prometheus
      metadata:
        serverAddress: http://prometheus.monitoring.svc:9090
        metricName: gpu_inference_queue
        query: |
          sum(rate(inference_requests_total[2m]))
          / sum(inference_capacity_per_pod)
        threshold: "1"                 # Scale when demand exceeds capacity
```

### Queue Worker Implementation

```python
# worker.py — GPU inference worker consuming from a queue
import json
import pika
import torch
from model import load_model, predict

model = load_model("model.pt", device="cuda")

def process_message(ch, method, properties, body):
    request = json.loads(body)

    result = predict(model, request["features"])

    # Publish result to response queue
    ch.basic_publish(
        exchange="",
        routing_key=request["reply_to"],
        body=json.dumps({
            "request_id": request["request_id"],
            "prediction": result,
        }),
        properties=pika.BasicProperties(
            correlation_id=properties.correlation_id,
        ),
    )
    ch.basic_ack(delivery_tag=method.delivery_tag)

connection = pika.BlockingConnection(
    pika.ConnectionParameters("rabbitmq")
)
channel = connection.channel()
channel.queue_declare(queue="inference-queue", durable=True)

# Prefetch 1 — process one at a time (GPU is the bottleneck)
channel.basic_qos(prefetch_count=1)
channel.basic_consume(queue="inference-queue", on_message_callback=process_message)

print("Worker ready. Waiting for messages...")
channel.start_consuming()
```

---

## Load Balancing for ML

### Why ML Load Balancing Is Different

```
Traditional Web App                    ML Inference Service
┌────────────────────┐                 ┌────────────────────┐
│ Request time: ~5ms │                 │ Request time: ~200ms│
│ Stateless          │                 │ GPU-bound           │
│ Uniform requests   │                 │ Variable request    │
│                    │                 │ sizes (1 vs 100     │
│ Round-robin works  │                 │ tokens)             │
│ perfectly          │                 │                     │
│                    │                 │ Round-robin causes  │
│                    │                 │ hot spots!          │
└────────────────────┘                 └────────────────────┘
```

### Load Balancing Strategies for ML

| Strategy | How It Works | Best For |
|---|---|---|
| **Round Robin** | Distribute evenly in order | Uniform request sizes |
| **Least Connections** | Send to pod with fewest active requests | Variable-cost requests |
| **Least Response Time** | Send to fastest-responding pod | Mixed model versions |
| **Weighted** | Route more traffic to bigger pods | Heterogeneous GPU fleet |
| **Queue-based** | Pods pull work when ready | GPU inference (best) |

### Optimal ML Load Balancing

```
     ❌ Round Robin (Bad for ML)        ✅ Least-Connections (Better)
     ┌────────────────────┐             ┌────────────────────┐
     │   Load Balancer    │             │   Load Balancer    │
     └──┬──────┬──────┬───┘             └──┬──────┬──────┬───┘
        │      │      │                    │      │      │
        ▼      ▼      ▼                    ▼      ▼      ▼
     ┌─────┐┌─────┐┌─────┐             ┌─────┐┌─────┐┌─────┐
     │ 5   ││ 5   ││ 5   │ reqs each   │ 8   ││ 4   ││ 3   │ by load
     │ req ││ req ││ req │             │ req ││ req ││ req │
     │     ││     ││     │             │(big)││(med)││(sml)│
     │ GPU ││ GPU ││ GPU │             │ GPU ││ GPU ││ GPU │
     │ 90% ││ 30% ││ 60% │             │ 75% ││ 70% ││ 65% │
     └─────┘└─────┘└─────┘             └─────┘└─────┘└─────┘
     Uneven GPU utilization             Balanced GPU utilization
```

### Nginx Configuration for ML

```nginx
# nginx.conf — load balancing for ML inference
upstream model_backend {
    least_conn;                        # Best for variable-cost ML requests

    server model-server-1:8080 weight=3;   # A100 GPU (3× capacity)
    server model-server-2:8080 weight=1;   # T4 GPU
    server model-server-3:8080 weight=1;   # T4 GPU

    keepalive 32;                      # Reuse connections
}

server {
    listen 80;

    location /predict {
        proxy_pass http://model_backend;
        proxy_http_version 1.1;
        proxy_set_header Connection "";

        # Timeouts tuned for ML inference
        proxy_connect_timeout 5s;
        proxy_read_timeout 30s;        # Allow up to 30s for inference
        proxy_send_timeout 10s;

        # Buffer settings for large responses
        proxy_buffering on;
        proxy_buffer_size 16k;
        proxy_buffers 4 32k;
    }

    location /health {
        proxy_pass http://model_backend;
        proxy_read_timeout 5s;
    }
}
```

### Kubernetes Ingress for ML

```yaml
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: model-ingress
  annotations:
    nginx.ingress.kubernetes.io/proxy-read-timeout: "60"
    nginx.ingress.kubernetes.io/proxy-body-size: "10m"
    nginx.ingress.kubernetes.io/load-balance: "ewma"    # least response time
    nginx.ingress.kubernetes.io/upstream-hash-by: "$request_uri"  # sticky routing
spec:
  rules:
    - host: ml.example.com
      http:
        paths:
          - path: /v1/predict
            pathType: Prefix
            backend:
              service:
                name: model-server-v1
                port:
                  number: 80
          - path: /v2/predict
            pathType: Prefix
            backend:
              service:
                name: model-server-v2
                port:
                  number: 80
```

---

## Caching Predictions

### When to Cache

```
┌────────────────────────────────────────────────────────┐
│                  Cache Decision Tree                   │
└──────────────────────┬─────────────────────────────────┘
                       │
              ┌────────▼────────┐
              │ Are inputs      │
              │ repeated often? │
              └────┬───────┬────┘
                   │       │
               Yes │       │ No
                   ▼       ▼
              ┌─────────┐ ┌──────────────┐
              │ Is the  │ │ Don't cache  │
              │ model   │ │ (unique      │
              │ static? │ │  inputs)     │
              └──┬───┬──┘ └──────────────┘
                 │   │
             Yes │   │ No (retrained often)
                 ▼   ▼
           ┌──────┐ ┌───────────────┐
           │Cache!│ │ Cache with    │
           │      │ │ short TTL     │
           │Long  │ │ (invalidate   │
           │TTL   │ │ on retrain)   │
           └──────┘ └───────────────┘
```

### Caching Architecture

```
┌──────────┐    ┌───────────────┐    ┌──────────────────┐
│  Client  │───▶│  API Gateway  │───▶│  Redis Cache     │
│          │    │               │    │                  │
│          │    │  1. Hash      │    │  Hit? Return     │
│          │    │     input     │    │  cached result   │
│          │◀───│               │◀───│                  │
│          │    │  3. Return    │    └──────────────────┘
│          │    │     result    │             │
│          │    └───────────────┘             │ Miss
│          │                                 ▼
│          │                        ┌──────────────────┐
│          │                        │  Model Server    │
│          │                        │  (GPU Inference) │
│          │                        │                  │
│          │                        │  2. Predict +    │
│          │                        │     store in     │
│          │                        │     cache        │
│          │                        └──────────────────┘
```

### Redis-Based Prediction Cache

```python
# prediction_cache.py
import hashlib
import json
import redis
import torch
import numpy as np
from functools import wraps

redis_client = redis.Redis(host="redis", port=6379, db=0)

def cache_key(model_version: str, features) -> str:
    """Create a deterministic cache key from model version + input features."""
    feature_bytes = json.dumps(features, sort_keys=True).encode()
    feature_hash = hashlib.sha256(feature_bytes).hexdigest()[:16]
    return f"pred:{model_version}:{feature_hash}"

class PredictionCache:
    def __init__(self, redis_client, model_version: str, ttl_seconds: int = 3600):
        self.redis = redis_client
        self.model_version = model_version
        self.ttl = ttl_seconds
        self.hits = 0
        self.misses = 0

    def get(self, features) -> dict | None:
        key = cache_key(self.model_version, features)
        cached = self.redis.get(key)
        if cached:
            self.hits += 1
            return json.loads(cached)
        self.misses += 1
        return None

    def set(self, features, prediction: dict):
        key = cache_key(self.model_version, features)
        self.redis.setex(key, self.ttl, json.dumps(prediction))

    def invalidate_all(self):
        """Invalidate all predictions for this model version."""
        pattern = f"pred:{self.model_version}:*"
        cursor = 0
        while True:
            cursor, keys = self.redis.scan(cursor, match=pattern, count=1000)
            if keys:
                self.redis.delete(*keys)
            if cursor == 0:
                break

    @property
    def hit_rate(self) -> float:
        total = self.hits + self.misses
        return self.hits / total if total > 0 else 0.0

    def stats(self) -> dict:
        return {
            "hits": self.hits,
            "misses": self.misses,
            "hit_rate": f"{self.hit_rate:.1%}",
        }

# Usage in FastAPI
from fastapi import FastAPI
app = FastAPI()
cache = PredictionCache(redis_client, model_version="v2.1.0", ttl_seconds=3600)

@app.post("/predict")
def predict(request: dict):
    features = request["features"]

    cached = cache.get(features)
    if cached:
        cached["cache"] = "hit"
        return cached

    prediction = model_predict(features)  # expensive GPU inference
    result = {"prediction": prediction, "cache": "miss"}
    cache.set(features, result)
    return result
```

### Cache Performance Impact

```
Without Cache                       With Cache (80% hit rate)
┌────────────────────────┐          ┌────────────────────────┐
│ 1000 requests/sec      │          │ 1000 requests/sec      │
│ × 50ms each (GPU)      │          │ 800 cache hits (1ms)   │
│ = 50 GPU-seconds/sec   │          │ 200 cache misses (50ms)│
│                        │          │ = 10 GPU-seconds/sec   │
│ Need: 50 GPU pods      │          │ Need: 10 GPU pods      │
│ Cost: $2,500/day       │          │ Cost: $500/day + Redis │
└────────────────────────┘          └────────────────────────┘
                                    Savings: ~$2,000/day (80%)
```

### Multi-Level Caching

| Cache Level | Latency | Capacity | Use Case |
|---|---|---|---|
| **In-process (dict)** | ~0.01 ms | MBs | Top-100 most frequent |
| **Redis/Memcached** | ~1 ms | GBs | All cacheable predictions |
| **CDN** | ~5 ms | TBs | Static model outputs (e.g., embeddings) |
| **Precomputed table** | ~0.1 ms | Depends | Finite input space (e.g., zip codes) |

---

## Batch Processing Optimization

### Why Batching Matters for GPUs

```
Sequential Inference                 Batched Inference
┌────────┐                          ┌────────────────────┐
│Input 1 │──▶ GPU ──▶ 10ms         │Input 1│Input 2│... │──▶ GPU ──▶ 15ms
├────────┤                          │Input 3│Input 4│    │    (for all 8)
│Input 2 │──▶ GPU ──▶ 10ms         └────────────────────┘
├────────┤                          
│  ...   │                           8 inputs × 10ms = 80ms (sequential)
├────────┤                           8 inputs batched  = 15ms (batched)
│Input 8 │──▶ GPU ──▶ 10ms         
└────────┘                           Throughput: 5.3× improvement
Total: 80ms                          GPU Utilization: 90% vs 15%
```

### Dynamic Batching

```
                    ┌───────────────────────┐
  Request 1 ───────▶│                       │
  Request 2 ───────▶│    Batch Collector    │
  Request 3 ───────▶│                       │
                    │  Wait up to 50ms OR   │──▶ GPU Inference
  Request 4 ───────▶│  until batch_size=32  │    (single batch)
  Request 5 ───────▶│                       │
                    │  Whichever comes first│
                    └───────────────────────┘
```

### Dynamic Batching Implementation

```python
# dynamic_batcher.py
import asyncio
import time
import torch
from dataclasses import dataclass, field
from typing import Any

@dataclass
class InferenceRequest:
    input_data: torch.Tensor
    future: asyncio.Future = field(default_factory=lambda: asyncio.get_event_loop().create_future())
    timestamp: float = field(default_factory=time.time)

class DynamicBatcher:
    """Collects individual requests and batches them for GPU efficiency."""

    def __init__(self, model, max_batch_size=32, max_wait_ms=50, device="cuda"):
        self.model = model
        self.max_batch_size = max_batch_size
        self.max_wait_ms = max_wait_ms
        self.device = device
        self.queue: asyncio.Queue = asyncio.Queue()
        self._running = False

    async def start(self):
        self._running = True
        asyncio.create_task(self._batch_loop())

    async def predict(self, input_data: torch.Tensor):
        """Submit a single request and wait for the batched result."""
        request = InferenceRequest(input_data=input_data)
        await self.queue.put(request)
        return await request.future

    async def _batch_loop(self):
        while self._running:
            batch: list[InferenceRequest] = []
            deadline = time.time() + self.max_wait_ms / 1000

            # Collect requests up to max_batch_size or max_wait_ms
            try:
                first = await asyncio.wait_for(
                    self.queue.get(),
                    timeout=self.max_wait_ms / 1000
                )
                batch.append(first)
            except asyncio.TimeoutError:
                continue

            while len(batch) < self.max_batch_size and time.time() < deadline:
                try:
                    remaining = deadline - time.time()
                    req = await asyncio.wait_for(
                        self.queue.get(), timeout=max(0.001, remaining)
                    )
                    batch.append(req)
                except asyncio.TimeoutError:
                    break

            # Process batch
            await self._process_batch(batch)

    async def _process_batch(self, batch: list[InferenceRequest]):
        inputs = torch.stack([r.input_data for r in batch]).to(self.device)

        with torch.no_grad():
            outputs = self.model(inputs)

        for i, request in enumerate(batch):
            request.future.set_result(outputs[i].cpu())

# Usage with FastAPI
from fastapi import FastAPI

app = FastAPI()
batcher = DynamicBatcher(model, max_batch_size=32, max_wait_ms=50)

@app.on_event("startup")
async def startup():
    await batcher.start()

@app.post("/predict")
async def predict(request: dict):
    tensor = torch.tensor(request["features"])
    result = await batcher.predict(tensor)
    return {"prediction": result.tolist()}
```

### Batch Inference Pipeline (Offline)

```python
# batch_inference.py — Process large datasets efficiently
import torch
from torch.utils.data import DataLoader, Dataset

class InferenceDataset(Dataset):
    def __init__(self, data):
        self.data = data

    def __len__(self):
        return len(self.data)

    def __getitem__(self, idx):
        return torch.tensor(self.data[idx], dtype=torch.float32)

def batch_predict(model, data, batch_size=256, device="cuda", num_workers=4):
    """Run inference on a large dataset with optimal batching."""
    dataset = InferenceDataset(data)
    loader = DataLoader(
        dataset,
        batch_size=batch_size,
        num_workers=num_workers,
        pin_memory=True,              # Faster CPU→GPU transfer
        prefetch_factor=2,            # Prefetch 2 batches
    )

    model = model.to(device).eval()
    all_predictions = []

    with torch.no_grad():
        for batch in loader:
            batch = batch.to(device, non_blocking=True)
            outputs = model(batch)
            all_predictions.append(outputs.cpu())

    return torch.cat(all_predictions)

# Throughput comparison
# batch_size=1:   ~100 predictions/sec
# batch_size=32:  ~2,500 predictions/sec
# batch_size=256: ~8,000 predictions/sec
# batch_size=512: ~9,000 predictions/sec (diminishing returns)
```

### Batching Configuration Guide

| Parameter | Low Latency | High Throughput | Balanced |
|---|---|---|---|
| **max_batch_size** | 4–8 | 64–256 | 16–32 |
| **max_wait_ms** | 5–10 ms | 100–500 ms | 20–50 ms |
| **Latency** | +5 ms (small overhead) | +100 ms (wait for full batch) | +20 ms |
| **Throughput** | 2–3× baseline | 10–20× baseline | 5–8× baseline |
| **GPU utilization** | 40–60 % | 85–95 % | 70–85 % |

---

## End-to-End Scaling Architecture

### Production ML Serving at Scale

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        Production ML Scaling Architecture                  │
│                                                                             │
│  ┌──────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │  Clients │───▶│  CDN / Edge  │───▶│  API Gateway │───▶│  Load        │  │
│  │          │    │  Cache       │    │  (Rate limit,│    │  Balancer    │  │
│  │          │    │              │    │   auth)      │    │  (L7, least  │  │
│  │          │    │  Static      │    │              │    │   conn)      │  │
│  └──────────┘    │  responses   │    └──────────────┘    └──────┬───────┘  │
│                  └──────────────┘                               │          │
│                                                                 │          │
│                  ┌──────────────────────────────────────────────┤          │
│                  │                                              │          │
│                  ▼                                              ▼          │
│  ┌──────────────────────────┐              ┌──────────────────────────┐    │
│  │    Redis Prediction      │              │    Model Server Pods     │    │
│  │    Cache                 │              │    (GPU)                 │    │
│  │                          │              │                          │    │
│  │    Cache Hit? ──▶ Return │   Miss ──▶   │  ┌────┐┌────┐┌────┐    │    │
│  │                          │              │  │Pod1││Pod2││Pod3│    │    │
│  │    Hit rate: ~70-90%     │              │  │GPU ││GPU ││GPU │    │    │
│  │                          │              │  └────┘└────┘└────┘    │    │
│  └──────────────────────────┘              │                          │    │
│                                            │  Dynamic Batching       │    │
│                                            │  HPA (queue depth)      │    │
│                                            └──────────────────────────┘    │
│                                                                             │
│  ┌──────────────────────────┐              ┌──────────────────────────┐    │
│  │    Async / Batch Path    │              │    Monitoring            │    │
│  │                          │              │                          │    │
│  │  Queue ──▶ Workers ──▶   │              │  Prometheus + Grafana   │    │
│  │              Results     │              │  ┌─────────────────┐    │    │
│  │                          │              │  │ Latency p99     │    │    │
│  │  KEDA auto-scales        │              │  │ Throughput      │    │    │
│  │  workers by queue depth  │              │  │ GPU utilization │    │    │
│  └──────────────────────────┘              │  │ Cache hit rate  │    │    │
│                                            │  │ Queue depth     │    │    │
│                                            │  └─────────────────┘    │    │
│                                            └──────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Scaling Metrics Decision Table

| Metric | Scale-Up Trigger | Scale-Down Trigger | Best For |
|---|---|---|---|
| **CPU utilization** | > 70 % | < 30 % | CPU-bound models |
| **GPU utilization** | > 80 % | < 30 % | GPU inference |
| **Queue depth** | > 10 per pod | 0 for 5 min | Async inference |
| **Request latency (p99)** | > SLA threshold | < 50 % of SLA | Latency-sensitive APIs |
| **Requests per second** | > capacity/pod | < 20 % capacity | Predictable workloads |
| **Custom business metric** | Varies | Varies | Domain-specific |

### Capacity Planning Formula

```
Required Pods = ceil(
    (Peak RPS × Average Latency per Request)
    ÷ (Target Utilization × Concurrency per Pod)
)

Example:
  Peak RPS:              500
  Avg Latency:           100ms = 0.1s
  Target Utilization:    70%
  Concurrency per Pod:   4 (batch size)

  Required Pods = ceil(500 × 0.1 / (0.7 × 4)) = ceil(17.9) = 18 pods

  With 20% headroom: 18 × 1.2 = 22 pods (set as maxReplicas)
  Minimum (off-peak): 3-5 pods (set as minReplicas)
```

---

## Summary

| Strategy | Key Takeaway |
|---|---|
| **Horizontal scaling** | Add more pods; best for stateless inference (preferred default) |
| **Vertical scaling** | Bigger machine; only when model can't fit on smaller hardware |
| **Queue-based auto-scaling** | Scale on queue depth, not CPU; essential for GPU workloads |
| **KEDA** | Kubernetes event-driven autoscaler; supports queues, Prometheus, etc. |
| **Load balancing** | Use least-connections or EWMA for ML; avoid round-robin |
| **Prediction caching** | 70–90 % hit rate saves proportional GPU costs |
| **Dynamic batching** | Collect requests into batches; 5–20× throughput improvement |
| **Offline batch inference** | Use large batch sizes + DataLoader with `pin_memory` |
| **Capacity planning** | Formula: Peak RPS × Latency ÷ (Utilization × Concurrency) |

> **Scaling priority order:** Cache first → Batch next → Scale horizontally → Scale vertically (last resort). Each layer multiplies the effectiveness of the next.

---

## Revision Questions

1. **Compare** horizontal and vertical scaling for ML inference. When is vertical scaling unavoidable, and what are its risks?

2. **Explain** why CPU-based auto-scaling is insufficient for GPU inference workloads. Design a KEDA-based scaling solution using queue depth as the primary metric.

3. **A recommendation API** receives 10,000 requests/second with 60 % of requests being for the same 1,000 products. Design a multi-level caching strategy and estimate the GPU cost savings.

4. **Explain** dynamic batching for real-time inference. What are the tradeoffs between `max_batch_size` and `max_wait_ms`? How would you tune these for a chatbot vs a batch analytics pipeline?

5. **You are tasked** with designing a system that serves a 7B-parameter LLM at 1,000 requests/minute with p99 latency under 2 seconds. Outline your scaling strategy, including hardware selection, batching, caching, and auto-scaling configuration.

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Cost Optimization](./04-cost-optimization.md) | [📂 Infrastructure](./README.md) | [ML Testing: Unit Testing →](../10-ML-Testing/01-unit-testing-ml-code.md) |

---

> **End of Unit 9 — Infrastructure.** Continue to [Unit 10 — ML Testing](../10-ML-Testing/01-unit-testing-ml-code.md) for testing strategies specific to ML systems.
