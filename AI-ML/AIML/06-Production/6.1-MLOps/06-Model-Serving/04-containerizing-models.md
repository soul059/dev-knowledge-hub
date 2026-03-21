# Containerizing Models

## Overview

Containerization packages an ML model, its dependencies, and serving code into a portable, reproducible unit. Docker is the standard tool for this. This chapter covers Dockerfiles for ML serving, multi-stage builds to minimize image size, BentoML for automated packaging, and Docker Compose for multi-service deployments.

---

## Why Containerize ML Models

```
  ┌──────────────────────────────────────────────────────────────────┐
  │                Without Containers                                │
  │                                                                  │
  │  Dev Machine            Staging             Production           │
  │  ┌──────────┐        ┌──────────┐        ┌──────────┐          │
  │  │ Python   │        │ Python   │        │ Python   │          │
  │  │ 3.11     │        │ 3.9      │        │ 3.10     │          │
  │  │ sklearn  │        │ sklearn  │        │ sklearn  │          │
  │  │ 1.3.0    │        │ 1.2.0    │  ❌    │ 1.1.0    │          │
  │  └──────────┘        └──────────┘        └──────────┘          │
  │  "Works on my machine" → Breaks in production                   │
  │                                                                  │
  │                With Containers                                   │
  │                                                                  │
  │  ┌────────────────────────────────────────────────────────┐     │
  │  │  Docker Image (identical everywhere)                   │     │
  │  │  Python 3.11 + sklearn 1.3.0 + model.joblib + app.py  │     │
  │  └────────────────────────────────────────────────────────┘     │
  │       │                │                │                        │
  │       ▼                ▼                ▼                        │
  │  Dev Machine        Staging         Production     ✅           │
  └──────────────────────────────────────────────────────────────────┘
```

---

## Basic Dockerfile for ML Serving

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install dependencies first (cached layer)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code and model
COPY app.py .
COPY models/ models/

# Non-root user for security
RUN useradd -m appuser
USER appuser

EXPOSE 8000

CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

```
# requirements.txt
fastapi==0.104.1
uvicorn==0.24.0
scikit-learn==1.3.2
joblib==1.3.2
numpy==1.26.2
```

**Build and run:**

```bash
docker build -t churn-model:v1 .
docker run -p 8000:8000 churn-model:v1

# Test
curl -X POST http://localhost:8000/predict \
  -H "Content-Type: application/json" \
  -d '{"age": 35, "income": 75000, "tenure": 24}'
```

---

## Multi-Stage Build (Smaller Images)

Multi-stage builds separate the build environment from the runtime, dramatically reducing image size.

```
  ┌─────────────────────────────────────────────────┐
  │  Stage 1: Builder (install + compile)           │
  │  ┌───────────────────────────────────────────┐  │
  │  │ python:3.11 (full)                        │  │
  │  │ gcc, build tools, pip install             │  │
  │  │ Size: ~1.2 GB                             │  │
  │  └────────────────────┬──────────────────────┘  │
  │                        │ copy only artifacts    │
  │  Stage 2: Runtime      ▼                        │
  │  ┌───────────────────────────────────────────┐  │
  │  │ python:3.11-slim                          │  │
  │  │ installed packages + app + model          │  │
  │  │ Size: ~350 MB                             │  │
  │  └───────────────────────────────────────────┘  │
  └─────────────────────────────────────────────────┘
```

```dockerfile
# ── Stage 1: Builder ────────────────────────────────
FROM python:3.11 AS builder

WORKDIR /build

COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

# ── Stage 2: Runtime ───────────────────────────────
FROM python:3.11-slim

WORKDIR /app

# Copy only installed packages from builder
COPY --from=builder /install /usr/local

COPY app.py .
COPY models/ models/

RUN useradd -m appuser
USER appuser

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health')"

CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "2"]
```

---

## Docker Best Practices for ML

```
  ┌──────────────────────────────────────────────────────────────────┐
  │  Best Practice                    │ Why                          │
  ├───────────────────────────────────┼──────────────────────────────┤
  │  Pin exact dependency versions    │ Reproducibility              │
  │  Use slim / alpine base images    │ Smaller image, less attack   │
  │                                   │ surface                      │
  │  COPY requirements.txt first      │ Leverage Docker layer cache  │
  │  Use --no-cache-dir with pip      │ Smaller image size           │
  │  Run as non-root user             │ Security                     │
  │  Add HEALTHCHECK                  │ Orchestrator can restart     │
  │  Use .dockerignore                │ Exclude unnecessary files    │
  │  Multi-stage builds               │ Separate build from runtime  │
  │  Don't embed secrets in image     │ Security — use env vars      │
  │  Tag images with model version    │ Traceability                 │
  └───────────────────────────────────┴──────────────────────────────┘
```

**.dockerignore:**

```
__pycache__/
*.pyc
.git/
.env
data/raw/
notebooks/
*.ipynb
tests/
.venv/
```

---

## BentoML — Automated Model Packaging

BentoML automates the packaging of models into production-ready containers with APIs, without writing Dockerfiles manually.

```
  ┌──────────────────────────────────────────────────────────────┐
  │                    BentoML Workflow                           │
  │                                                               │
  │  ┌──────────┐   ┌──────────────┐   ┌──────────────────────┐│
  │  │ Train    │──►│ Save model  │──►│ Define service.py    ││
  │  │ model    │   │ to BentoML  │   │ (API + preprocessing)││
  │  └──────────┘   │ model store │   └──────────┬───────────┘│
  │                  └──────────────┘              │             │
  │                                      ┌─────────▼──────────┐ │
  │                                      │ bentoml build      │ │
  │                                      │ → Bento (artifact) │ │
  │                                      └─────────┬──────────┘ │
  │                                      ┌─────────▼──────────┐ │
  │                                      │ bentoml containerize│ │
  │                                      │ → Docker image     │ │
  │                                      └─────────────────────┘ │
  └──────────────────────────────────────────────────────────────┘
```

```python
# save_model.py — Save trained model to BentoML store
import bentoml
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.datasets import make_classification

X, y = make_classification(n_samples=1000, n_features=3, random_state=42)
model = GradientBoostingClassifier().fit(X, y)

saved_model = bentoml.sklearn.save_model("churn_classifier", model)
print(f"Saved: {saved_model}")
# Saved: Model(tag="churn_classifier:abc123")
```

```python
# service.py — Define BentoML service
import numpy as np
import bentoml
from bentoml.io import JSON, NumpyNdarray

runner = bentoml.sklearn.get("churn_classifier:latest").to_runner()
svc = bentoml.Service("churn_service", runners=[runner])


@svc.api(input=JSON(), output=JSON())
async def predict(input_data: dict) -> dict:
    features = np.array([[
        input_data["age"],
        input_data["income"],
        input_data["tenure"],
    ]])
    prediction = await runner.predict.async_run(features)
    proba = await runner.predict_proba.async_run(features)
    return {
        "prediction": int(prediction[0]),
        "probability": round(float(proba[0][1]), 4),
    }
```

```yaml
# bentofile.yaml
service: "service:svc"
include:
  - "service.py"
python:
  packages:
    - scikit-learn==1.3.2
    - numpy==1.26.2
docker:
  python_version: "3.11"
```

```bash
# Build and containerize
bentoml build
bentoml containerize churn_service:latest -t churn-service:v1
docker run -p 3000:3000 churn-service:v1
```

---

## Docker Compose — Multi-Service ML Deployment

```yaml
# docker-compose.yml
version: "3.8"

services:
  model-api:
    build: .
    ports:
      - "8000:8000"
    environment:
      - MODEL_PATH=/app/models/churn_model.joblib
      - LOG_LEVEL=info
    volumes:
      - ./models:/app/models:ro
    healthcheck:
      test: ["CMD", "python", "-c",
             "import urllib.request; urllib.request.urlopen('http://localhost:8000/health')"]
      interval: 30s
      timeout: 5s
      retries: 3
    deploy:
      resources:
        limits:
          cpus: "2.0"
          memory: 2G
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data

  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - model-api

volumes:
  redis-data:
```

```bash
# Launch all services
docker compose up -d

# Scale model API to 3 replicas
docker compose up -d --scale model-api=3

# View logs
docker compose logs -f model-api
```

---

## Image Size Comparison

```
  ┌──────────────────────────────┬────────────┐
  │ Approach                     │ Image Size │
  ├──────────────────────────────┼────────────┤
  │ python:3.11 (naive)          │ ~1.2 GB    │
  │ python:3.11-slim             │ ~450 MB    │
  │ Multi-stage (slim runtime)   │ ~350 MB    │
  │ BentoML (auto-optimized)     │ ~400 MB    │
  │ Distroless Python            │ ~250 MB    │
  │ python:3.11-alpine + deps    │ ~200 MB    │
  └──────────────────────────────┴────────────┘
```

---

## Revision Questions

1. **Why is Docker layer caching important and how do you optimize for it?**
2. **What is a multi-stage Docker build and how does it reduce image size?**
3. **What security practices should you follow when containerizing ML models?**
4. **How does BentoML simplify the model packaging and serving workflow?**
5. **When would you use Docker Compose for an ML serving deployment?**

---

[Previous: 03-model-servers.md](03-model-servers.md) | [Next: 05-serverless-inference.md](05-serverless-inference.md)
