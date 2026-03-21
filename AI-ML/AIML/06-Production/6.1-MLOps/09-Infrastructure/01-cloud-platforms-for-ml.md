# ☁️ Cloud Platforms for ML

> **Unit 9 · Infrastructure · Chapter 1**
>
> A practical comparison of the three major cloud providers for machine learning — AWS SageMaker, GCP Vertex AI, and Azure ML — covering key services, pricing, and decision frameworks.

---

## Table of Contents

1. [Chapter Overview](#chapter-overview)
2. [Why Cloud for ML?](#why-cloud-for-ml)
3. [AWS SageMaker](#aws-sagemaker)
4. [GCP Vertex AI](#gcp-vertex-ai)
5. [Azure Machine Learning](#azure-machine-learning)
6. [Head-to-Head Comparison](#head-to-head-comparison)
7. [Pricing Considerations](#pricing-considerations)
8. [Decision Framework](#decision-framework)
9. [Code Examples](#code-examples)
10. [Summary](#summary)
11. [Revision Questions](#revision-questions)
12. [Navigation](#navigation)

---

## Chapter Overview

Choosing the right cloud platform is one of the most impactful infrastructure decisions for any ML team. This chapter compares the three dominant platforms — **AWS SageMaker**, **GCP Vertex AI**, and **Azure ML** — across training, serving, experiment tracking, pipeline orchestration, pricing, and ecosystem integration. By the end you will be able to evaluate trade-offs and select the best fit for your workload.

---

## Why Cloud for ML?

```
On-Prem Challenges              Cloud Benefits
┌──────────────────────┐        ┌──────────────────────────┐
│ Fixed GPU capacity   │───────▶│ Elastic GPU/TPU scaling  │
│ Long procurement     │───────▶│ Instant provisioning     │
│ Ops burden           │───────▶│ Managed services         │
│ Under-utilization    │───────▶│ Pay-per-use pricing      │
│ Single region        │───────▶│ Global deployment        │
└──────────────────────┘        └──────────────────────────┘
```

**Key motivations:**

| Factor | On-Premises | Cloud |
|---|---|---|
| Capital expense | High upfront | OpEx (pay-as-you-go) |
| Time to first experiment | Weeks–months | Minutes |
| Scaling | Manual hardware | API call / auto-scale |
| Maintenance | In-house team | Managed by provider |
| Compliance | Full control | Shared responsibility |

---

## AWS SageMaker

### Architecture Overview

```
┌─────────────────────────────── AWS SageMaker ───────────────────────────────┐
│                                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌───────────────┐   │
│  │   SageMaker  │  │  SageMaker   │  │  SageMaker   │  │  SageMaker    │   │
│  │   Studio     │  │  Training    │  │  Endpoints   │  │  Pipelines    │   │
│  │  (IDE)       │  │  (Jobs)      │  │  (Serving)   │  │  (Orchestr.)  │   │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └───────┬───────┘   │
│         │                 │                 │                   │           │
│         ▼                 ▼                 ▼                   ▼           │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                         S3  ·  ECR  ·  IAM                          │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌───────────────┐   │
│  │  Feature     │  │  Model       │  │  Ground      │  │  Clarify      │   │
│  │  Store       │  │  Registry    │  │  Truth        │  │  (Explain)    │   │
│  └──────────────┘  └──────────────┘  └──────────────┘  └───────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Key Services

| Service | Purpose |
|---|---|
| **SageMaker Studio** | Full IDE with notebooks, experiments, debugger |
| **Training Jobs** | Managed distributed training on any instance type |
| **Endpoints** | Real-time, serverless, or async inference |
| **Pipelines** | CI/CD-style ML workflow orchestration |
| **Feature Store** | Centralized feature management (online + offline) |
| **Model Registry** | Version, approve, and deploy models |
| **Ground Truth** | Data labeling with human + active learning |
| **Clarify** | Bias detection and model explainability |
| **Canvas** | No-code ML for business analysts |
| **JumpStart** | Pre-trained model hub (LLMs, CV, etc.) |

### Strengths

- Deepest integration with the AWS ecosystem (S3, Lambda, Step Functions, ECS)
- Largest selection of instance types (including `ml.p5.48xlarge` with H100 GPUs)
- Mature spot-instance support for training (up to 90 % savings)
- SageMaker Inference Recommender for right-sizing endpoints

---

## GCP Vertex AI

### Architecture Overview

```
┌────────────────────────────── GCP Vertex AI ────────────────────────────────┐
│                                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌───────────────┐   │
│  │  Workbench   │  │  Training    │  │  Prediction   │  │  Vertex AI    │   │
│  │  (Notebooks) │  │  (Custom +   │  │  (Online /    │  │  Pipelines    │   │
│  │              │  │   AutoML)    │  │   Batch)      │  │  (KFP-based)  │   │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └───────┬───────┘   │
│         │                 │                 │                   │           │
│         ▼                 ▼                 ▼                   ▼           │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                    GCS  ·  Artifact Registry  ·  IAM                │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌───────────────┐   │
│  │  Feature     │  │  Model       │  │  Experiments  │  │  Matching     │   │
│  │  Store       │  │  Registry    │  │  (Tracking)   │  │  Engine (ANN) │   │
│  └──────────────┘  └──────────────┘  └──────────────┘  └───────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Key Services

| Service | Purpose |
|---|---|
| **Workbench** | Managed JupyterLab notebooks |
| **Custom Training** | Bring-your-own-code training on GPUs / TPUs |
| **AutoML** | No-code training for tabular, vision, text, video |
| **Prediction** | Online endpoints + batch prediction |
| **Pipelines** | Kubeflow Pipelines–based orchestration |
| **Feature Store** | Managed feature serving (BigTable-backed) |
| **Model Registry** | Versioning and lineage |
| **Experiments** | MLflow-compatible experiment tracking |
| **Matching Engine** | Billion-scale approximate nearest neighbor search |
| **Model Garden** | Access to PaLM, Gemini, open-source LLMs |

### Strengths

- **TPU access** — custom ASICs purpose-built for deep learning
- Best-in-class BigQuery integration for feature engineering at scale
- Kubeflow-native pipelines provide portability
- Strong AutoML offering for quick baseline models
- Tight integration with Google's generative AI (Gemini, PaLM API)

---

## Azure Machine Learning

### Architecture Overview

```
┌───────────────────────────── Azure ML ──────────────────────────────────────┐
│                                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌───────────────┐   │
│  │  Azure ML    │  │  Compute     │  │  Managed      │  │  Designer     │   │
│  │  Studio      │  │  Clusters    │  │  Endpoints     │  │  (Drag-drop)  │   │
│  │  (Web UI)    │  │  (Training)  │  │  (Serving)     │  │               │   │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └───────┬───────┘   │
│         │                 │                 │                   │           │
│         ▼                 ▼                 ▼                   ▼           │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │              Blob Storage  ·  ACR  ·  Azure AD  ·  Key Vault       │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌───────────────┐   │
│  │  Responsible │  │  Model       │  │  Data         │  │  Prompt       │   │
│  │  AI Toolbox  │  │  Registry    │  │  Labeling     │  │  Flow         │   │
│  └──────────────┘  └──────────────┘  └──────────────┘  └───────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Key Services

| Service | Purpose |
|---|---|
| **Azure ML Studio** | Web-based workspace for the full ML lifecycle |
| **Compute Clusters** | Auto-scaling training clusters (CPU / GPU) |
| **Managed Endpoints** | Blue/green deployment with traffic splitting |
| **Designer** | Drag-and-drop pipeline builder |
| **Responsible AI** | Fairness, interpretability, error analysis |
| **Model Registry** | MLflow-compatible model versioning |
| **Data Labeling** | Human-in-the-loop annotation |
| **Prompt Flow** | LLM app development and evaluation |
| **AutoML** | Automated featurization, model selection, tuning |

### Strengths

- Best enterprise integration (Azure AD, RBAC, private endpoints, VNets)
- Native MLflow support as the tracking backend
- Strong hybrid-cloud story with Azure Arc
- Responsible AI tooling is the most comprehensive of the three
- Deep integration with Microsoft Fabric for analytics

---

## Head-to-Head Comparison

### Service Matrix

| Capability | AWS SageMaker | GCP Vertex AI | Azure ML |
|---|---|---|---|
| **IDE / Notebooks** | Studio (JupyterLab) | Workbench | Studio (Web) |
| **Training** | Training Jobs | Custom Training | Compute Clusters |
| **AutoML** | Autopilot | AutoML (strong) | AutoML |
| **Serving** | Endpoints (RT/Async/Serverless) | Prediction | Managed Endpoints |
| **Pipelines** | SageMaker Pipelines | KFP-based Pipelines | Designer + SDK Pipelines |
| **Feature Store** | ✅ | ✅ | ✅ (via Feast) |
| **Experiment Tracking** | Experiments | Experiments | MLflow-native |
| **Model Registry** | ✅ | ✅ | ✅ |
| **Explainability** | Clarify | Explainable AI | Responsible AI |
| **LLM Support** | JumpStart / Bedrock | Model Garden / Gemini | Prompt Flow / Azure OpenAI |
| **TPU Support** | ❌ | ✅ | ❌ |
| **Spot Training** | ✅ (Managed Spot) | ✅ (Preemptible VMs) | ✅ (Low-priority VMs) |

### Ecosystem Integration

```
     AWS SageMaker            GCP Vertex AI            Azure ML
    ┌─────────────┐          ┌─────────────┐          ┌─────────────┐
    │  S3         │          │  BigQuery    │          │  Azure AD   │
    │  Lambda     │          │  Dataflow    │          │  Synapse    │
    │  Step Fns   │          │  Pub/Sub     │          │  Fabric     │
    │  ECS / EKS  │          │  GKE         │          │  AKS        │
    │  Bedrock    │          │  Gemini API  │          │  OpenAI Svc │
    │  CloudWatch │          │  Monitoring  │          │  App Insights│
    └─────────────┘          └─────────────┘          └─────────────┘
```

---

## Pricing Considerations

### Pricing Models

| Component | AWS | GCP | Azure |
|---|---|---|---|
| **Training (GPU)** | Per-second billing | Per-second billing | Per-minute billing |
| **Inference endpoint** | Per-hour (always-on) | Per-hour or per-request | Per-hour (always-on) |
| **Serverless inference** | ✅ (per-request) | ✅ (per-request) | ❌ (preview) |
| **Spot / Preemptible** | Up to 90 % off | Up to 91 % off | Up to 80 % off |
| **Free tier** | 250 hrs Studio, 50 hrs training/mo | $300 credit (90 days) | $200 credit (30 days) |

### Cost Estimation Example — Monthly (Approximate)

```
Scenario: 100 training hours (V100 GPU) + 1 real-time endpoint (T4 GPU)

┌──────────────────────────┬───────────┬───────────┬───────────┐
│ Component                │   AWS     │   GCP     │  Azure    │
├──────────────────────────┼───────────┼───────────┼───────────┤
│ Training (100 hrs V100)  │  ~$310    │  ~$250    │  ~$300    │
│ Inference (730 hrs T4)   │  ~$390    │  ~$340    │  ~$370    │
│ Storage (1 TB S3/GCS)    │  ~$23     │  ~$20     │  ~$21     │
├──────────────────────────┼───────────┼───────────┼───────────┤
│ Estimated Total          │  ~$723    │  ~$610    │  ~$691    │
└──────────────────────────┴───────────┴───────────┴───────────┘

* Prices are approximate and vary by region. Always consult the official
  pricing calculator for your configuration.
```

### Tips to Reduce Costs

1. **Use spot / preemptible instances** for fault-tolerant training
2. **Serverless inference** for bursty, low-traffic models
3. **Auto-scaling to zero** during off-peak (Vertex AI supports this natively)
4. **Reserved capacity / committed use discounts** for steady-state workloads
5. **Right-size instances** — avoid over-provisioning GPU memory

---

## Decision Framework

```
                         ┌──────────────────────┐
                         │  Choose Your Cloud   │
                         └──────────┬───────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    ▼               ▼               ▼
           ┌───────────────┐ ┌───────────────┐ ┌───────────────┐
           │ Already on    │ │ Need TPUs or  │ │ Enterprise    │
           │ AWS? Heavy    │ │ BigQuery      │ │ (AD, hybrid,  │
           │ S3/Lambda     │ │ integration?  │ │ compliance)?  │
           │ usage?        │ │               │ │               │
           └──────┬────────┘ └──────┬────────┘ └──────┬────────┘
                  │                 │                  │
                  ▼                 ▼                  ▼
           ┌───────────────┐ ┌───────────────┐ ┌───────────────┐
           │  SageMaker    │ │  Vertex AI    │ │  Azure ML     │
           └───────────────┘ └───────────────┘ └───────────────┘
```

### When to Choose Which

| Choose **AWS SageMaker** when… | Choose **GCP Vertex AI** when… | Choose **Azure ML** when… |
|---|---|---|
| Your org is AWS-first | You need TPU training | Enterprise security is top priority |
| You want the widest instance selection | You rely on BigQuery for data | You need hybrid cloud (Arc) |
| You need mature spot training | You want Kubeflow portability | Your org uses Microsoft 365 / AD |
| You want SageMaker JumpStart LLMs | You want AutoML-first approach | You want built-in Responsible AI |
| You need SageMaker Canvas (no-code) | You want Gemini / PaLM integration | You want MLflow as native backend |

---

## Code Examples

### AWS SageMaker — Launch a Training Job

```python
import sagemaker
from sagemaker.pytorch import PyTorch

session = sagemaker.Session()
role = sagemaker.get_execution_role()

estimator = PyTorch(
    entry_point="train.py",
    source_dir="src/",
    role=role,
    instance_count=1,
    instance_type="ml.p3.2xlarge",       # V100 GPU
    framework_version="2.1",
    py_version="py310",
    use_spot_instances=True,              # up to 90% savings
    max_wait=7200,                        # max seconds to wait for spot
    max_run=3600,                         # max training time
    hyperparameters={
        "epochs": 10,
        "batch-size": 64,
        "lr": 0.001,
    },
    output_path=f"s3://{session.default_bucket()}/output",
)

estimator.fit({
    "train": "s3://my-bucket/data/train/",
    "val":   "s3://my-bucket/data/val/",
})
```

### GCP Vertex AI — Launch a Custom Training Job

```python
from google.cloud import aiplatform

aiplatform.init(project="my-project", location="us-central1")

job = aiplatform.CustomTrainingJob(
    display_name="my-training-job",
    script_path="train.py",
    container_uri="us-docker.pkg.dev/vertex-ai/training/pytorch-gpu.2-1:latest",
    requirements=["transformers==4.35.0"],
)

model = job.run(
    replica_count=1,
    machine_type="n1-standard-8",
    accelerator_type="NVIDIA_TESLA_V100",
    accelerator_count=1,
    args=["--epochs=10", "--batch-size=64", "--lr=0.001"],
    base_output_dir="gs://my-bucket/output",
)

# Deploy to an endpoint
endpoint = model.deploy(
    machine_type="n1-standard-4",
    accelerator_type="NVIDIA_TESLA_T4",
    accelerator_count=1,
    min_replica_count=1,
    max_replica_count=3,
)
```

### Azure ML — Submit a Training Job

```python
from azure.ai.ml import MLClient, command, Input
from azure.identity import DefaultAzureCredential

ml_client = MLClient(
    DefaultAzureCredential(),
    subscription_id="<sub-id>",
    resource_group_name="my-rg",
    workspace_name="my-workspace",
)

job = command(
    code="./src",
    command="python train.py --epochs 10 --batch-size 64 --lr 0.001",
    environment="AzureML-pytorch-2.1-cuda12@latest",
    compute="gpu-cluster",
    inputs={
        "train_data": Input(type="uri_folder", path="azureml://datastores/data/train"),
    },
    instance_count=1,
)

returned_job = ml_client.jobs.create_or_update(job)
print(f"Job URL: {returned_job.studio_url}")
```

### Multi-Cloud Config — Terraform Snippet

```hcl
# terraform/main.tf — provision ML infrastructure on AWS
resource "aws_sagemaker_notebook_instance" "ml_notebook" {
  name          = "ml-dev-notebook"
  instance_type = "ml.t3.medium"
  role_arn      = aws_iam_role.sagemaker_role.arn

  tags = {
    Environment = "development"
    Team        = "ml-platform"
  }
}

resource "aws_sagemaker_endpoint_configuration" "inference" {
  name = "model-endpoint-config"

  production_variants {
    variant_name           = "primary"
    model_name             = aws_sagemaker_model.my_model.name
    instance_type          = "ml.g4dn.xlarge"
    initial_instance_count = 1
  }
}
```

---

## Summary

| Aspect | AWS SageMaker | GCP Vertex AI | Azure ML |
|---|---|---|---|
| **Best for** | AWS-native orgs | Data / AI-first teams | Enterprise / Microsoft shops |
| **Unique edge** | Instance breadth, spot training | TPUs, BigQuery, AutoML | AD integration, Responsible AI |
| **LLM story** | JumpStart + Bedrock | Model Garden + Gemini | Azure OpenAI + Prompt Flow |
| **Pipeline style** | SageMaker Pipelines (proprietary) | Kubeflow Pipelines (portable) | SDK Pipelines + Designer |
| **Pricing** | Per-second, serverless option | Per-second, scale-to-zero | Per-minute, committed use |
| **Learning curve** | Medium | Medium | Medium-High (enterprise config) |

> **Bottom line:** There is no universally "best" cloud for ML. The right choice depends on your existing cloud footprint, data ecosystem, compliance requirements, and team expertise. Many organizations adopt a **multi-cloud** or **cloud-agnostic** strategy using tools like MLflow, Kubeflow, and Terraform to avoid vendor lock-in.

---

## Revision Questions

1. **Compare and contrast** the three major cloud ML platforms on at least four dimensions (training, serving, pipelines, pricing). When would you choose GCP Vertex AI over AWS SageMaker?

2. **Explain** the concept of spot / preemptible instances for ML training. What safeguards must you implement to use them reliably?

3. **You are designing** an ML platform for a healthcare company that already uses Azure AD and requires strict compliance (HIPAA). Which cloud would you recommend and why?

4. **Describe** two strategies to reduce inference costs on any cloud platform. Include specific service names or configurations.

5. **A startup** is building a real-time recommendation system. They have no existing cloud commitment but need rapid iteration and strong AutoML. Make a recommendation with justification.

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← 08: Feedback Loops](../08-Model-Monitoring/06-feedback-loops.md) | [📂 Infrastructure](./README.md) | [Kubernetes for ML →](./02-kubernetes-for-ml.md) |

---

> **Next up:** [Chapter 2 — Kubernetes for ML](./02-kubernetes-for-ml.md) — Container orchestration for scalable model training and serving.
