# ⚙️ Kubernetes for ML

> **Unit 9 · Infrastructure · Chapter 2**
>
> How to leverage Kubernetes (K8s) for scalable model training and serving — covering core concepts, Kubeflow, resource management, autoscaling, and production-ready YAML configurations.

---

## Table of Contents

1. [Chapter Overview](#chapter-overview)
2. [Why Kubernetes for ML?](#why-kubernetes-for-ml)
3. [Core K8s Concepts for ML Engineers](#core-k8s-concepts-for-ml-engineers)
4. [Deploying Models on Kubernetes](#deploying-models-on-kubernetes)
5. [Kubeflow — ML Toolkit for K8s](#kubeflow--ml-toolkit-for-k8s)
6. [Resource Requests and Limits](#resource-requests-and-limits)
7. [Horizontal Pod Autoscaling](#horizontal-pod-autoscaling)
8. [Production YAML Configs](#production-yaml-configs)
9. [Monitoring and Observability](#monitoring-and-observability)
10. [Summary](#summary)
11. [Revision Questions](#revision-questions)
12. [Navigation](#navigation)

---

## Chapter Overview

Kubernetes has become the de facto orchestration layer for production ML workloads. It provides **reproducible environments** via containers, **elastic scaling** via autoscalers, and **declarative infrastructure** via YAML manifests. This chapter teaches ML engineers the K8s primitives they actually need — pods, deployments, services, GPU scheduling — and introduces **Kubeflow** as the ML-specific toolkit built on top of Kubernetes.

---

## Why Kubernetes for ML?

```
Traditional ML Deployment            Kubernetes ML Deployment
┌─────────────────────┐              ┌─────────────────────────────────┐
│  SSH into VM        │              │  kubectl apply -f deploy.yaml   │
│  pip install ...    │              │                                 │
│  python serve.py    │              │  ┌─────┐ ┌─────┐ ┌─────┐      │
│  (single process)   │              │  │Pod 1│ │Pod 2│ │Pod 3│      │
│  No auto-recovery   │              │  │ GPU │ │ GPU │ │ GPU │      │
│  Manual scaling     │              │  └──┬──┘ └──┬──┘ └──┬──┘      │
└─────────────────────┘              │     └───────┼───────┘          │
                                     │         Service LB             │
                                     │    Auto-heal · Auto-scale      │
                                     └─────────────────────────────────┘
```

| Benefit | Description |
|---|---|
| **Reproducibility** | Containers ensure identical environments everywhere |
| **Self-healing** | Failed pods are automatically restarted |
| **Scaling** | HPA scales replicas based on CPU, GPU, or custom metrics |
| **Resource isolation** | Requests / limits prevent noisy-neighbor problems |
| **Declarative** | Infrastructure-as-code via YAML, GitOps-friendly |
| **Multi-cloud** | Same manifests work on EKS, GKE, AKS, or bare metal |

---

## Core K8s Concepts for ML Engineers

### Architecture

```
┌──────────────────────────── Kubernetes Cluster ─────────────────────────────┐
│                                                                             │
│  ┌─── Control Plane ──────────────────────────────────────────────────────┐ │
│  │  API Server ─── Scheduler ─── Controller Manager ─── etcd             │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  ┌─── Worker Node 1 (GPU) ─────────┐  ┌─── Worker Node 2 (GPU) ─────────┐ │
│  │  ┌───────┐  ┌───────┐           │  │  ┌───────┐  ┌───────┐           │ │
│  │  │ Pod A │  │ Pod B │  kubelet  │  │  │ Pod C │  │ Pod D │  kubelet  │ │
│  │  │(train)│  │(serve)│  kube-    │  │  │(serve)│  │(serve)│  kube-    │ │
│  │  └───────┘  └───────┘  proxy    │  │  └───────┘  └───────┘  proxy    │ │
│  │  NVIDIA Device Plugin           │  │  NVIDIA Device Plugin           │ │
│  └─────────────────────────────────┘  └─────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Essential Resources

| Resource | Purpose | ML Use Case |
|---|---|---|
| **Pod** | Smallest deployable unit (1+ containers) | Run a model server |
| **Deployment** | Manages replica sets, rolling updates | Scale inference pods |
| **Service** | Stable network endpoint (ClusterIP / LoadBalancer) | Route traffic to model pods |
| **Job** | Run-to-completion task | Training job |
| **CronJob** | Scheduled Job | Nightly retraining |
| **ConfigMap** | Key-value config | Hyperparameters, feature flags |
| **Secret** | Encrypted key-value | API keys, model registry creds |
| **PersistentVolumeClaim** | Storage | Training data, model artifacts |
| **Namespace** | Logical isolation | Separate dev / staging / prod |
| **Node** | Physical / virtual machine | GPU node pool |

### Pod Lifecycle

```
Pending ──▶ Running ──▶ Succeeded
                │
                └──▶ Failed ──▶ (Restarted by Controller)
                        │
                        └──▶ CrashLoopBackOff (repeated failures)
```

---

## Deploying Models on Kubernetes

### Step-by-Step Workflow

```
┌─────────┐    ┌──────────┐    ┌────────────┐    ┌────────────┐
│  Train   │───▶│  Package  │───▶│  Push to   │───▶│  Deploy    │
│  Model   │    │  Docker   │    │  Registry  │    │  to K8s    │
└─────────┘    └──────────┘    └────────────┘    └────────────┘
```

### 1. Dockerfile for Model Serving

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY model/ ./model/
COPY serve.py .

EXPOSE 8080

# Health check for K8s readiness probe
HEALTHCHECK --interval=30s --timeout=5s \
    CMD curl -f http://localhost:8080/health || exit 1

CMD ["uvicorn", "serve:app", "--host", "0.0.0.0", "--port", "8080"]
```

### 2. Model Server (FastAPI)

```python
# serve.py
import torch
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()
model = torch.jit.load("model/model.pt")
model.eval()

class PredictRequest(BaseModel):
    features: list[float]

class PredictResponse(BaseModel):
    prediction: float
    model_version: str = "v1.2.0"

@app.get("/health")
def health():
    return {"status": "healthy"}

@app.post("/predict", response_model=PredictResponse)
def predict(req: PredictRequest):
    tensor = torch.tensor([req.features])
    with torch.no_grad():
        output = model(tensor)
    return PredictResponse(prediction=output.item())
```

### 3. Kubernetes Deployment YAML

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: model-server
  labels:
    app: model-server
    version: v1.2.0
spec:
  replicas: 3
  selector:
    matchLabels:
      app: model-server
  template:
    metadata:
      labels:
        app: model-server
        version: v1.2.0
    spec:
      containers:
        - name: model-server
          image: registry.example.com/ml/model-server:v1.2.0
          ports:
            - containerPort: 8080
          resources:
            requests:
              cpu: "1"
              memory: "2Gi"
              nvidia.com/gpu: "1"
            limits:
              cpu: "2"
              memory: "4Gi"
              nvidia.com/gpu: "1"
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 60
            periodSeconds: 15
          env:
            - name: MODEL_VERSION
              value: "v1.2.0"
            - name: LOG_LEVEL
              value: "info"
      tolerations:
        - key: "nvidia.com/gpu"
          operator: "Exists"
          effect: "NoSchedule"
      nodeSelector:
        cloud.google.com/gke-accelerator: nvidia-tesla-t4
---
apiVersion: v1
kind: Service
metadata:
  name: model-server
spec:
  type: LoadBalancer
  selector:
    app: model-server
  ports:
    - port: 80
      targetPort: 8080
      protocol: TCP
```

---

## Kubeflow — ML Toolkit for K8s

### Architecture

```
┌──────────────────────────── Kubeflow on K8s ────────────────────────────────┐
│                                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌───────────────┐   │
│  │  Kubeflow    │  │  KFServing   │  │  Katib       │  │  Training     │   │
│  │  Pipelines   │  │  (KServe)    │  │  (AutoML /   │  │  Operators    │   │
│  │              │  │              │  │   HPO)       │  │  (TF/PyTorch) │   │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └───────┬───────┘   │
│         │                 │                 │                   │           │
│         ▼                 ▼                 ▼                   ▼           │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                    Kubernetes (EKS / GKE / AKS)                     │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌───────────────┐   │
│  │  Notebooks   │  │  Metadata    │  │  Feature     │  │  Volumes      │   │
│  │  (Jupyter)   │  │  Store       │  │  Store       │  │  (PVC)        │   │
│  └──────────────┘  └──────────────┘  └──────────────┘  └───────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Key Components

| Component | Purpose |
|---|---|
| **Pipelines** | DAG-based ML workflow orchestration (compile → upload → run) |
| **KServe** | Serverless inference with autoscaling, canary rollouts, explainability |
| **Katib** | Hyperparameter tuning (grid, random, Bayesian, NAS) |
| **Training Operators** | Distributed training for TensorFlow, PyTorch, MPI, XGBoost |
| **Notebooks** | Managed Jupyter notebooks inside the cluster |
| **Metadata** | Artifact and lineage tracking |

### Kubeflow Pipeline Example

```python
# pipeline.py
from kfp import dsl, compiler

@dsl.component(base_image="python:3.11-slim", packages_to_install=["pandas", "scikit-learn"])
def preprocess(data_path: str, output_path: dsl.OutputPath("Dataset")):
    import pandas as pd
    df = pd.read_csv(data_path)
    df_clean = df.dropna()
    df_clean.to_csv(output_path, index=False)

@dsl.component(base_image="python:3.11-slim", packages_to_install=["scikit-learn", "joblib"])
def train(dataset: dsl.InputPath("Dataset"), model_path: dsl.OutputPath("Model")):
    import pandas as pd
    from sklearn.ensemble import RandomForestClassifier
    import joblib

    df = pd.read_csv(dataset)
    X, y = df.drop("target", axis=1), df["target"]
    clf = RandomForestClassifier(n_estimators=100)
    clf.fit(X, y)
    joblib.dump(clf, model_path)

@dsl.pipeline(name="training-pipeline", description="End-to-end training")
def ml_pipeline(data_path: str = "gs://bucket/data.csv"):
    preprocess_task = preprocess(data_path=data_path)
    train_task = train(dataset=preprocess_task.outputs["output_path"])

compiler.Compiler().compile(ml_pipeline, "pipeline.yaml")
```

### KServe InferenceService

```yaml
# kserve/inference-service.yaml
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: sklearn-model
spec:
  predictor:
    model:
      modelFormat:
        name: sklearn
      storageUri: "gs://my-bucket/models/sklearn/v1"
      resources:
        requests:
          cpu: "1"
          memory: "2Gi"
        limits:
          cpu: "2"
          memory: "4Gi"
    minReplicas: 1
    maxReplicas: 5
    scaleTarget: 10           # concurrent requests per replica
    scaleMetric: concurrency
```

---

## Resource Requests and Limits

### How the Scheduler Uses Resources

```
Pod Spec                          Scheduler Decision
┌────────────────────┐            ┌─────────────────────────────┐
│ requests:          │            │  "Guarantee this minimum    │
│   cpu: "500m"      │──────────▶│   to schedule the pod"      │
│   memory: "1Gi"    │            └─────────────────────────────┘
│   nvidia.com/gpu: 1│
│                    │            ┌─────────────────────────────┐
│ limits:            │            │  "Kill if it exceeds these  │
│   cpu: "2"         │──────────▶│   (OOMKilled for memory)"   │
│   memory: "4Gi"    │            └─────────────────────────────┘
│   nvidia.com/gpu: 1│
└────────────────────┘
```

### GPU Resource Rules

> GPUs in Kubernetes are **non-sharable by default** — each GPU is allocated to exactly one pod. One pod cannot request a fraction of a GPU.

```yaml
# Correct — request exactly 1 GPU
resources:
  requests:
    nvidia.com/gpu: "1"
  limits:
    nvidia.com/gpu: "1"

# Invalid — fractional GPU requests are NOT supported natively
# resources:
#   requests:
#     nvidia.com/gpu: "0.5"   # ❌ Not allowed
```

### QoS Classes

| QoS Class | Condition | Eviction Priority |
|---|---|---|
| **Guaranteed** | requests == limits for all containers | Last to be evicted |
| **Burstable** | At least one request set, requests < limits | Medium priority |
| **BestEffort** | No requests or limits set | First to be evicted |

> **Best practice for ML inference:** Use **Guaranteed** QoS (requests == limits) to prevent eviction under memory pressure.

### Resource Sizing Guide

| Workload | CPU Request | Memory Request | GPU |
|---|---|---|---|
| Small model (sklearn) | 0.5–1 | 512Mi–1Gi | None |
| Medium model (BERT) | 2–4 | 4–8Gi | 1× T4 |
| Large model (LLM 7B) | 4–8 | 16–32Gi | 1× A100 |
| Very large (LLM 70B) | 8–16 | 64–128Gi | 4–8× A100 |

---

## Horizontal Pod Autoscaling

### How HPA Works

```
                   ┌───────────────┐
                   │  Metrics      │
                   │  Server       │
                   └───────┬───────┘
                           │ current metrics
                           ▼
                   ┌───────────────┐
         ┌────────│      HPA      │────────┐
         │        │  Controller   │        │
         │        └───────────────┘        │
         │                                 │
    Scale Up                          Scale Down
    (add pods)                       (remove pods)
         │                                 │
         ▼                                 ▼
   ┌─────────────────────────────────────────┐
   │  Deployment: model-server               │
   │  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐      │
   │  │Pod 1│ │Pod 2│ │Pod 3│ │Pod 4│ ...   │
   │  └─────┘ └─────┘ └─────┘ └─────┘      │
   └─────────────────────────────────────────┘
```

### HPA Formula

```
desiredReplicas = ceil(
    currentReplicas × (currentMetricValue / targetMetricValue)
)

Example:
  currentReplicas = 3
  currentCPU = 80%
  targetCPU = 50%
  desiredReplicas = ceil(3 × 80/50) = ceil(4.8) = 5
```

### HPA YAML

```yaml
# k8s/hpa.yaml
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
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
        - type: Pods
          value: 4
          periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300    # wait 5 min before scaling down
      policies:
        - type: Percent
          value: 25
          periodSeconds: 60
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 60
    - type: Pods
      pods:
        metric:
          name: inference_requests_per_second
        target:
          type: AverageValue
          averageValue: "50"
```

### Custom Metrics for ML Autoscaling

```python
# expose custom metrics via Prometheus
from prometheus_client import Counter, Histogram, start_http_server

REQUEST_COUNT = Counter(
    "inference_requests_total",
    "Total inference requests",
    ["model", "version"]
)

LATENCY = Histogram(
    "inference_latency_seconds",
    "Inference latency in seconds",
    ["model"],
    buckets=[0.01, 0.05, 0.1, 0.25, 0.5, 1.0, 2.5]
)

# Start metrics server on port 9090
start_http_server(9090)
```

---

## Production YAML Configs

### Complete Production Deployment

```yaml
# k8s/production/model-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: model-server
  namespace: ml-production
  labels:
    app: model-server
    team: ml-platform
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  selector:
    matchLabels:
      app: model-server
  template:
    metadata:
      labels:
        app: model-server
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "9090"
    spec:
      terminationGracePeriodSeconds: 60
      containers:
        - name: model-server
          image: registry.example.com/ml/model-server:v1.2.0
          ports:
            - containerPort: 8080
              name: http
            - containerPort: 9090
              name: metrics
          resources:
            requests:
              cpu: "2"
              memory: "4Gi"
              nvidia.com/gpu: "1"
            limits:
              cpu: "2"
              memory: "4Gi"
              nvidia.com/gpu: "1"
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
            failureThreshold: 3
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 60
            periodSeconds: 15
            failureThreshold: 5
          startupProbe:
            httpGet:
              path: /health
              port: 8080
            failureThreshold: 30
            periodSeconds: 10
          volumeMounts:
            - name: model-cache
              mountPath: /models
      volumes:
        - name: model-cache
          persistentVolumeClaim:
            claimName: model-pvc
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: topology.kubernetes.io/zone
          whenUnsatisfiable: DoNotSchedule
          labelSelector:
            matchLabels:
              app: model-server
---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: model-server-pdb
  namespace: ml-production
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: model-server
```

### Training Job YAML

```yaml
# k8s/training/training-job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: model-training-v1-2-0
  namespace: ml-training
spec:
  backoffLimit: 3
  activeDeadlineSeconds: 86400   # 24 hour timeout
  template:
    spec:
      restartPolicy: OnFailure
      containers:
        - name: trainer
          image: registry.example.com/ml/trainer:v1.2.0
          command: ["python", "train.py"]
          args:
            - "--epochs=50"
            - "--batch-size=128"
            - "--lr=0.001"
            - "--output-dir=/output"
          resources:
            requests:
              cpu: "8"
              memory: "32Gi"
              nvidia.com/gpu: "4"
            limits:
              cpu: "8"
              memory: "32Gi"
              nvidia.com/gpu: "4"
          volumeMounts:
            - name: training-data
              mountPath: /data
              readOnly: true
            - name: output
              mountPath: /output
      volumes:
        - name: training-data
          persistentVolumeClaim:
            claimName: training-data-pvc
        - name: output
          persistentVolumeClaim:
            claimName: training-output-pvc
      nodeSelector:
        gpu-type: a100
      tolerations:
        - key: "nvidia.com/gpu"
          operator: "Exists"
          effect: "NoSchedule"
```

---

## Monitoring and Observability

```
┌───────────────┐     ┌───────────────┐     ┌───────────────┐
│   Prometheus  │────▶│   Grafana     │     │   Alertmanager│
│   (Metrics)   │     │  (Dashboards) │     │   (Alerts)    │
└───────┬───────┘     └───────────────┘     └───────┬───────┘
        │                                           │
        │  scrape /metrics                          │ alert
        ▼                                           ▼
┌───────────────────────────────────────────────────────────┐
│                    Model Server Pods                      │
│  Metrics: latency, throughput, GPU util, error rate       │
└───────────────────────────────────────────────────────────┘
```

### Key Kubectl Commands

```bash
# Check pod status
kubectl get pods -n ml-production -l app=model-server

# View pod logs
kubectl logs -f deployment/model-server -n ml-production

# Check resource usage
kubectl top pods -n ml-production

# Describe pod (see events, scheduling decisions)
kubectl describe pod model-server-abc123 -n ml-production

# Check HPA status
kubectl get hpa -n ml-production

# Port-forward for local testing
kubectl port-forward svc/model-server 8080:80 -n ml-production

# Check GPU allocation
kubectl describe nodes | grep -A 5 "nvidia.com/gpu"
```

---

## Summary

| Concept | Key Takeaway |
|---|---|
| **Why K8s for ML** | Reproducible, self-healing, scalable, cloud-agnostic |
| **Pod** | Smallest unit; one model server = one pod |
| **Deployment** | Manages replicas with rolling updates |
| **Service** | Stable endpoint for load-balanced traffic |
| **Kubeflow** | ML-specific layer: pipelines, KServe, Katib, training operators |
| **Requests** | Minimum resources guaranteed; used by scheduler |
| **Limits** | Maximum resources allowed; OOMKill if exceeded |
| **QoS** | Guaranteed (requests==limits) is best for inference |
| **HPA** | Auto-scale pods on CPU, memory, or custom metrics |
| **GPU scheduling** | Non-sharable by default; 1 GPU = 1 pod |
| **Probes** | Readiness, liveness, and startup probes prevent bad traffic |
| **PDB** | Pod Disruption Budget ensures minimum availability |

> **Rule of thumb:** Start with a simple Deployment + Service + HPA. Add Kubeflow components only when you outgrow manual workflows.

---

## Revision Questions

1. **Explain** the difference between resource `requests` and `limits` in Kubernetes. What happens when a pod exceeds its memory limit?

2. **Design** a Kubernetes deployment for a real-time image classification model that needs GPU access, health checks, and auto-scaling based on request latency. What YAML resources would you need?

3. **Compare** KServe (Kubeflow Serving) with a plain Kubernetes Deployment + Service for model serving. What advantages does KServe provide?

4. **A model server** takes 45 seconds to load a large model into GPU memory. How would you configure Kubernetes probes to handle this gracefully?

5. **Your HPA** is scaling up and down rapidly ("flapping"). What configuration changes would you make to stabilize it?

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Cloud Platforms for ML](./01-cloud-platforms-for-ml.md) | [📂 Infrastructure](./README.md) | [GPU Management →](./03-gpu-management.md) |

---

> **Next up:** [Chapter 3 — GPU Management](./03-gpu-management.md) — GPU vs CPU decisions, NVIDIA GPU sharing, monitoring, and cost-performance tradeoffs.
