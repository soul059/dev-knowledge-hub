# Serverless Inference

## Overview

Serverless inference runs ML models on cloud functions (AWS Lambda, Google Cloud Functions, Azure Functions) that scale to zero when idle and scale up automatically on demand. This eliminates server management but introduces constraints like cold starts, memory limits, and package size restrictions. This chapter covers when serverless works for ML, its limitations, and practical examples.

---

## Serverless Architecture for ML

```
  ┌──────────────────────────────────────────────────────────────────┐
  │                   Serverless Inference                           │
  │                                                                  │
  │  ┌──────────┐    ┌──────────────┐    ┌───────────────────────┐  │
  │  │  Client  │───►│  API Gateway │───►│  Cloud Function       │  │
  │  │  Request │    │  (triggers   │    │  ┌─────────────────┐  │  │
  │  └──────────┘    │   function)  │    │  │ Load model      │  │  │
  │                  └──────────────┘    │  │ (cold start)    │  │  │
  │                                      │  └────────┬────────┘  │  │
  │                                      │  ┌────────▼────────┐  │  │
  │                                      │  │ Predict         │  │  │
  │  ┌──────────┐                       │  └────────┬────────┘  │  │
  │  │ Response │◄───────────────────────│  ┌────────▼────────┐  │  │
  │  └──────────┘                       │  │ Return JSON     │  │  │
  │                                      │  └─────────────────┘  │  │
  │  No servers to manage                └───────────────────────┘  │
  │  Pay only for execution time                                     │
  │  Auto-scales 0 → 1000+ concurrent                               │
  └──────────────────────────────────────────────────────────────────┘
```

---

## The Cold Start Problem

```
  ┌──────────────────────────────────────────────────────────────────┐
  │                                                                  │
  │  Cold Start (first request after idle)                           │
  │  ┌───────────┬──────────────┬──────────┬────────┐               │
  │  │ Provision │ Download     │ Load     │ Predict│               │
  │  │ container │ code + deps  │ model    │        │               │
  │  │ ~200ms    │ ~500ms–2s    │ ~1–10s   │ ~50ms  │               │
  │  └───────────┴──────────────┴──────────┴────────┘               │
  │  Total: 2–12+ seconds  ❌                                       │
  │                                                                  │
  │  Warm Start (container already running)                          │
  │  ┌────────┐                                                      │
  │  │ Predict│                                                      │
  │  │ ~50ms  │                                                      │
  │  └────────┘                                                      │
  │  Total: ~50ms  ✅                                                │
  │                                                                  │
  │  Mitigation strategies:                                          │
  │  • Provisioned concurrency (keep N instances warm)               │
  │  • Smaller models (< 250 MB)                                     │
  │  • ONNX Runtime for faster loading                               │
  │  • Cache model in /tmp after first load                          │
  │  • Warm-up pings via CloudWatch scheduled events                 │
  └──────────────────────────────────────────────────────────────────┘
```

---

## AWS Lambda Example

**Project structure:**

```
lambda-ml/
├── handler.py
├── model.onnx              # small model (< 50 MB ideal)
├── requirements.txt
└── template.yaml            # SAM template
```

**Lambda handler:**

```python
# handler.py
import json
import os
import numpy as np
import onnxruntime as ort

# Load model once (reused across warm invocations)
MODEL_PATH = os.path.join(os.path.dirname(__file__), "model.onnx")
session = None


def get_session():
    global session
    if session is None:
        session = ort.InferenceSession(MODEL_PATH)
    return session


def lambda_handler(event, context):
    """AWS Lambda handler for ML inference."""
    try:
        body = json.loads(event.get("body", "{}"))
    except json.JSONDecodeError:
        return {
            "statusCode": 400,
            "body": json.dumps({"error": "Invalid JSON"}),
        }
    
    # Validate input
    required_fields = ["age", "income", "tenure"]
    missing = [f for f in required_fields if f not in body]
    if missing:
        return {
            "statusCode": 400,
            "body": json.dumps({"error": f"Missing fields: {missing}"}),
        }
    
    # Run inference
    sess = get_session()
    input_name = sess.get_inputs()[0].name
    features = np.array(
        [[body["age"], body["income"], body["tenure"]]],
        dtype=np.float32,
    )
    
    result = sess.run(None, {input_name: features})
    prediction = int(result[0][0])
    probability = float(result[1][0][1]) if len(result) > 1 else None
    
    response = {
        "prediction": prediction,
        "label": "churn" if prediction == 1 else "no_churn",
    }
    if probability is not None:
        response["probability"] = round(probability, 4)
    
    return {
        "statusCode": 200,
        "headers": {"Content-Type": "application/json"},
        "body": json.dumps(response),
    }
```

**SAM template (template.yaml):**

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Transform: AWS::Serverless-2016-10-31

Globals:
  Function:
    Timeout: 30
    MemorySize: 1024

Resources:
  MLFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: handler.lambda_handler
      Runtime: python3.11
      Architectures:
        - x86_64
      Events:
        Predict:
          Type: Api
          Properties:
            Path: /predict
            Method: post
      # Keep warm to reduce cold starts
      ProvisionedConcurrencyConfig:
        ProvisionedConcurrentExecutions: 2
```

**Deploy:**

```bash
# Package and deploy with AWS SAM
sam build
sam deploy --guided

# Test
curl -X POST https://abc123.execute-api.us-east-1.amazonaws.com/Prod/predict \
  -H "Content-Type: application/json" \
  -d '{"age": 35, "income": 75000, "tenure": 24}'
```

---

## Google Cloud Functions Example

```python
# main.py — Google Cloud Function
import json
import numpy as np
import onnxruntime as ort
import functions_framework

session = ort.InferenceSession("model.onnx")


@functions_framework.http
def predict(request):
    """Google Cloud Function for ML inference."""
    if request.method != "POST":
        return json.dumps({"error": "POST only"}), 405
    
    body = request.get_json(silent=True)
    if not body:
        return json.dumps({"error": "Invalid JSON"}), 400
    
    features = np.array(
        [[body["age"], body["income"], body["tenure"]]],
        dtype=np.float32,
    )
    
    input_name = session.get_inputs()[0].name
    result = session.run(None, {input_name: features})
    
    return json.dumps({
        "prediction": int(result[0][0]),
        "label": "churn" if result[0][0] == 1 else "no_churn",
    })
```

```bash
# Deploy
gcloud functions deploy predict \
  --runtime python311 \
  --trigger-http \
  --allow-unauthenticated \
  --memory 1024MB \
  --timeout 30s
```

---

## When Serverless Works vs Doesn't

```
  ┌───────────────────────┬────────────────────┬─────────────────────┐
  │ Dimension             │ ✅ Serverless Fits │ ❌ Serverless Hurts │
  ├───────────────────────┼────────────────────┼─────────────────────┤
  │ Traffic pattern       │ Bursty, infrequent │ Steady, high volume │
  │ Model size            │ < 250 MB           │ > 500 MB (GPT, etc)│
  │ Latency tolerance     │ Seconds OK         │ < 100ms required   │
  │ GPU needed            │ No                 │ Yes                 │
  │ Compute per request   │ Light (sklearn,    │ Heavy (transformers │
  │                       │ XGBoost, ONNX)     │ large CNNs)        │
  │ Cost at scale         │ Expensive at high  │ Dedicated cheaper   │
  │                       │ sustained QPS      │ at high QPS         │
  │ Ops overhead          │ Near zero          │ N/A                 │
  │ Framework             │ sklearn, XGBoost,  │ Large PyTorch /     │
  │                       │ lightweight ONNX   │ TensorFlow models   │
  └───────────────────────┴────────────────────┴─────────────────────┘
```

---

## Serverless Constraints

```
  ┌──────────────────────────────────────────────────────────────────┐
  │  Platform Limits (approximate — check current docs)              │
  │                                                                  │
  │  ┌─────────────────────┬──────────────┬────────────────────┐    │
  │  │ Constraint          │ AWS Lambda   │ GCP Cloud Function │    │
  │  ├─────────────────────┼──────────────┼────────────────────┤    │
  │  │ Max memory          │ 10 GB        │ 32 GB              │    │
  │  │ Max timeout         │ 15 min       │ 60 min             │    │
  │  │ Package size (zip)  │ 250 MB       │ 500 MB (source)    │    │
  │  │ Container image     │ 10 GB        │ N/A                │    │
  │  │ Temp storage        │ 10 GB /tmp   │ In-memory only     │    │
  │  │ GPU support         │ ❌           │ ❌                 │    │
  │  │ Concurrency         │ 1000 default │ 1000 default       │    │
  │  └─────────────────────┴──────────────┴────────────────────┘    │
  │                                                                  │
  │  Workarounds:                                                    │
  │  • Use Lambda container images for larger packages (up to 10 GB)│
  │  • Use ONNX format for faster model loading and smaller size     │
  │  • Quantize models to reduce size (float32 → int8)              │
  │  • Use Lambda Layers for shared dependencies                     │
  └──────────────────────────────────────────────────────────────────┘
```

---

## Lambda with Container Image (Large Models)

```dockerfile
# Dockerfile for Lambda container (supports up to 10 GB)
FROM public.ecr.aws/lambda/python:3.11

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY handler.py .
COPY model.onnx .

CMD ["handler.lambda_handler"]
```

```bash
# Build and push to ECR
docker build -t ml-lambda .
aws ecr get-login-password | docker login --username AWS --password-stdin $ECR_URI
docker tag ml-lambda:latest $ECR_URI/ml-lambda:latest
docker push $ECR_URI/ml-lambda:latest
```

---

## Cost Comparison

```
  ┌──────────────────────────────────────────────────────────────────┐
  │                                                                  │
  │  Requests/day:  100        10,000       1,000,000                │
  │                                                                  │
  │  Serverless:    ~$0.01     ~$1.00       ~$100+                   │
  │  (Lambda)       cheapest   still cheap  expensive                │
  │                                                                  │
  │  Always-on:     ~$50/mo    ~$50/mo      ~$50–200/mo             │
  │  (EC2/ECS)      overkill   break-even   cheapest                 │
  │                                                                  │
  │  Rule of thumb:                                                  │
  │  • < 100K requests/month → serverless wins                      │
  │  • > 500K requests/month → dedicated infra wins                 │
  │  • In between → depends on traffic pattern                       │
  └──────────────────────────────────────────────────────────────────┘
```

---

## Revision Questions

1. **What is the cold start problem and how can you mitigate it?**
2. **What types of ML models are well-suited for serverless inference?**
3. **Why is ONNX Runtime preferred over scikit-learn for Lambda deployments?**
4. **At what traffic volume does serverless become more expensive than dedicated servers?**
5. **What are the key platform constraints of AWS Lambda for ML workloads?**

---

[Previous: 04-containerizing-models.md](04-containerizing-models.md) | [Next: ../07-Model-Deployment/01-deployment-strategies.md](../07-Model-Deployment/01-deployment-strategies.md)
