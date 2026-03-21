# REST APIs for ML

## Overview

REST APIs are the most common way to serve ML models in real-time. They expose an HTTP endpoint that accepts input features and returns predictions as JSON. This chapter covers building production-grade ML APIs with FastAPI and Flask, including input validation, error handling, health checks, and performance considerations.

---

## ML API Architecture

```
  ┌───────────────────────────────────────────────────────────────┐
  │                    ML Serving API                             │
  │                                                               │
  │  ┌──────────┐   ┌──────────────┐   ┌───────────────────────┐│
  │  │  Client  │──►│  Load       │──►│  API Server (FastAPI) ││
  │  │  Request │   │  Balancer   │   │                       ││
  │  │  (JSON)  │   │  (Nginx)   │   │  ┌─────────────────┐  ││
  │  └──────────┘   └──────────────┘   │  │ Input Validation│  ││
  │                                     │  └────────┬────────┘  ││
  │                                     │  ┌────────▼────────┐  ││
  │                                     │  │ Preprocessing   │  ││
  │                                     │  └────────┬────────┘  ││
  │                                     │  ┌────────▼────────┐  ││
  │                                     │  │ Model.predict() │  ││
  │                                     │  └────────┬────────┘  ││
  │                                     │  ┌────────▼────────┐  ││
  │  ┌──────────┐                      │  │ Response Format │  ││
  │  │ Response │◄─────────────────────│  └─────────────────┘  ││
  │  │ (JSON)   │                      └───────────────────────┘│
  │  └──────────┘                                                │
  └───────────────────────────────────────────────────────────────┘
```

---

## Request / Response Design

Good API design makes ML models easy to consume and debug.

```
  Request (POST /predict)              Response (200 OK)
  ┌──────────────────────┐             ┌──────────────────────────┐
  │ {                    │             │ {                        │
  │   "features": {      │     ──►     │   "prediction": 1,      │
  │     "age": 35,       │             │   "probability": 0.87,  │
  │     "income": 75000, │             │   "label": "churn",     │
  │     "tenure": 24     │             │   "model_version": "v3",│
  │   }                  │             │   "latency_ms": 12      │
  │ }                    │             │ }                        │
  └──────────────────────┘             └──────────────────────────┘

  Error Response (422)
  ┌──────────────────────────────┐
  │ {                            │
  │   "detail": [                │
  │     {                        │
  │       "loc": ["age"],        │
  │       "msg": "value must be  │
  │              > 0",           │
  │       "type": "value_error"  │
  │     }                        │
  │   ]                          │
  │ }                            │
  └──────────────────────────────┘
```

---

## Full FastAPI Example — Serving an Sklearn Model

```python
import time
import logging
from contextlib import asynccontextmanager

import joblib
import numpy as np
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field, validator

# ── Pydantic schemas for validation ─────────────────────────────

class PredictionRequest(BaseModel):
    age: float = Field(..., gt=0, lt=150, description="Customer age")
    income: float = Field(..., ge=0, description="Annual income in USD")
    tenure: int = Field(..., ge=0, description="Months as customer")
    
    @validator("income")
    def income_reasonable(cls, v):
        if v > 10_000_000:
            raise ValueError("Income seems unreasonably high")
        return v

    class Config:
        json_schema_extra = {
            "example": {"age": 35, "income": 75000, "tenure": 24}
        }


class PredictionResponse(BaseModel):
    prediction: int
    probability: float
    label: str
    model_version: str
    latency_ms: float


class HealthResponse(BaseModel):
    status: str
    model_loaded: bool
    model_version: str


# ── App lifecycle: load model once at startup ───────────────────

MODEL_VERSION = "v3"
ml_model = None


@asynccontextmanager
async def lifespan(app: FastAPI):
    global ml_model
    ml_model = joblib.load("models/churn_model.joblib")
    logging.info(f"Model {MODEL_VERSION} loaded")
    yield
    logging.info("Shutting down")


app = FastAPI(
    title="Churn Prediction API",
    version=MODEL_VERSION,
    lifespan=lifespan,
)


# ── Endpoints ───────────────────────────────────────────────────

@app.get("/health", response_model=HealthResponse)
def health():
    return HealthResponse(
        status="healthy",
        model_loaded=ml_model is not None,
        model_version=MODEL_VERSION,
    )


@app.post("/predict", response_model=PredictionResponse)
def predict(request: PredictionRequest):
    start = time.time()
    
    if ml_model is None:
        raise HTTPException(status_code=503, detail="Model not loaded")
    
    X = np.array([[request.age, request.income, request.tenure]])
    
    try:
        pred = ml_model.predict(X)[0]
        proba = ml_model.predict_proba(X)[0]
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Prediction failed: {e}")
    
    latency = (time.time() - start) * 1000
    label = "churn" if pred == 1 else "no_churn"
    
    return PredictionResponse(
        prediction=int(pred),
        probability=round(float(proba[1]), 4),
        label=label,
        model_version=MODEL_VERSION,
        latency_ms=round(latency, 2),
    )


@app.post("/predict/batch")
def predict_batch(requests: list[PredictionRequest]):
    """Score multiple records in a single API call."""
    start = time.time()
    
    X = np.array([[r.age, r.income, r.tenure] for r in requests])
    preds = ml_model.predict(X)
    probas = ml_model.predict_proba(X)[:, 1]
    
    latency = (time.time() - start) * 1000
    
    return {
        "predictions": [
            {
                "prediction": int(p),
                "probability": round(float(prob), 4),
                "label": "churn" if p == 1 else "no_churn",
            }
            for p, prob in zip(preds, probas)
        ],
        "count": len(requests),
        "latency_ms": round(latency, 2),
        "model_version": MODEL_VERSION,
    }


# Run: uvicorn app:app --host 0.0.0.0 --port 8000 --workers 4
```

---

## Flask Alternative (Simpler Setup)

```python
from flask import Flask, request, jsonify
import joblib
import numpy as np

app = Flask(__name__)
model = joblib.load("models/churn_model.joblib")


@app.route("/predict", methods=["POST"])
def predict():
    data = request.get_json()
    
    # Manual validation (Flask has no built-in schema validation)
    required = ["age", "income", "tenure"]
    missing = [f for f in required if f not in data]
    if missing:
        return jsonify({"error": f"Missing fields: {missing}"}), 400
    
    X = np.array([[data["age"], data["income"], data["tenure"]]])
    pred = model.predict(X)[0]
    proba = model.predict_proba(X)[0, 1]
    
    return jsonify({
        "prediction": int(pred),
        "probability": round(float(proba), 4),
    })


@app.route("/health")
def health():
    return jsonify({"status": "healthy"})


# Run: gunicorn app:app --bind 0.0.0.0:8000 --workers 4
```

---

## FastAPI vs Flask for ML Serving

```
  ┌──────────────────┬─────────────────────┬─────────────────────┐
  │ Feature          │ FastAPI             │ Flask               │
  ├──────────────────┼─────────────────────┼─────────────────────┤
  │ Validation       │ Built-in (Pydantic) │ Manual / extensions │
  │ Auto docs        │ ✅ Swagger + ReDoc  │ ❌ Needs extension  │
  │ Async support    │ ✅ Native           │ ⚠️  Limited          │
  │ Performance      │ High (Starlette)    │ Moderate            │
  │ Type hints       │ ✅ Enforced         │ ❌ Optional         │
  │ Learning curve   │ Low–medium          │ Very low            │
  │ Ecosystem        │ Growing fast        │ Mature, huge        │
  │ Best for         │ Production ML APIs  │ Prototypes, simple  │
  │                  │                     │ internal tools      │
  └──────────────────┴─────────────────────┴─────────────────────┘
```

---

## Production Checklist

```
  ┌──────────────────────────────────────────────────────────────┐
  │  ✅ Input validation with Pydantic schemas                   │
  │  ✅ Health endpoint (/health or /readyz)                     │
  │  ✅ Structured logging (request ID, latency, status)         │
  │  ✅ Error handling with meaningful HTTP status codes          │
  │  ✅ Model version in every response                          │
  │  ✅ Latency tracking in response                             │
  │  ✅ Multiple workers (uvicorn --workers or gunicorn)         │
  │  ✅ Graceful startup / shutdown (load model at startup)      │
  │  ✅ Rate limiting for public-facing APIs                     │
  │  ✅ Request/response logging for debugging & monitoring      │
  └──────────────────────────────────────────────────────────────┘
```

---

## Revision Questions

1. **Why is Pydantic validation important for ML serving APIs?**
2. **What should a production ML API health endpoint return?**
3. **Why should the model be loaded at startup rather than per-request?**
4. **Compare FastAPI and Flask for ML serving — when would you pick each?**
5. **What information should be included in every prediction response?**

---

[Previous: 01-serving-patterns.md](01-serving-patterns.md) | [Next: 03-model-servers.md](03-model-servers.md)
