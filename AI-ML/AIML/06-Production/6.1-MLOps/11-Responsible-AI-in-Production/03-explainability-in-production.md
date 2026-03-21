# 🔍 Explainability in Production

> **Unit 11 · Responsible AI in Production · Chapter 3**
>
> How to make deployed ML models explainable — SHAP and LIME at serving time, feature importance logging, building explanation APIs, balancing latency vs explainability depth, and creating model cards for transparency.

---

## Table of Contents

1. [Chapter Overview](#chapter-overview)
2. [Why Explainability Matters in Production](#why-explainability-matters-in-production)
3. [SHAP at Serving Time](#shap-at-serving-time)
4. [LIME at Serving Time](#lime-at-serving-time)
5. [Feature Importance Logging](#feature-importance-logging)
6. [Explanation APIs](#explanation-apis)
7. [Balancing Latency vs Explainability](#balancing-latency-vs-explainability)
8. [Model Cards](#model-cards)
9. [Summary](#summary)
10. [Revision Questions](#revision-questions)
11. [Navigation](#navigation)

---

## Chapter Overview

A model that says "denied" without explanation is a liability — legally, ethically, and operationally. Regulators require explanations, users deserve them, and debugging teams need them. But generating explanations at inference time introduces latency, compute cost, and architectural complexity. This chapter covers practical approaches to **production explainability**: using SHAP and LIME at serving time, logging feature contributions for every prediction, building explanation APIs that return human-readable reasons, managing the trade-off between explanation depth and response latency, and documenting models with **model cards**.

```
Explainability Spectrum in Production
┌───────────────────────────────────────────────────────────────┐
│                                                               │
│  Low Latency ◄──────────────────────────────► High Fidelity   │
│                                                               │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────┐  │
│  │ Pre-computed│  │ Cached     │  │ Real-time  │  │ Full   │  │
│  │ global     │  │ SHAP for   │  │ SHAP per   │  │ LIME   │  │
│  │ feature    │  │ top-K      │  │ request    │  │ per    │  │
│  │ importance │  │ archetypes │  │            │  │ request│  │
│  └────────────┘  └────────────┘  └────────────┘  └────────┘  │
│   ~0 ms            ~1 ms           ~50-200 ms     ~1-5 sec    │
│   Same for all     Approximate     Exact          Most        │
│   requests         per cluster     per instance   faithful    │
└───────────────────────────────────────────────────────────────┘
```

---

## Why Explainability Matters in Production

### Stakeholder Needs

| Stakeholder | What They Need | Example |
|---|---|---|
| **End user** | Simple reason for the decision | "Your application was declined primarily due to debt-to-income ratio" |
| **Customer support** | Factors driving the prediction | Top-3 features with direction and magnitude |
| **Data scientist** | Full feature attribution for debugging | SHAP values for all features |
| **Auditor / Regulator** | Evidence that model isn't discriminatory | Feature importance excluding protected attributes |
| **Legal team** | Documentation of decision-making process | Model card + explanation logs |

### Explainability Methods Comparison

| Method | Type | Speed | Fidelity | Model-Agnostic? |
|---|---|---|---|---|
| **SHAP (TreeExplainer)** | Exact | Fast (~ms) | High | Tree models only |
| **SHAP (KernelExplainer)** | Approximate | Slow (~sec) | Medium-High | ✅ Yes |
| **LIME** | Local surrogate | Slow (~sec) | Medium | ✅ Yes |
| **Integrated Gradients** | Gradient-based | Fast (~ms) | High | Neural nets only |
| **Feature importance** | Global | Pre-computed | Low (global only) | ✅ Yes |
| **Attention weights** | Architecture | Free | Low-Medium | Transformers only |
| **Counterfactual** | What-if | Slow | High | ✅ Yes |

---

## SHAP at Serving Time

### TreeSHAP for Tree-Based Models

```python
# explainability/shap_serving.py
"""SHAP explanations at serving time for tree-based models."""
import shap
import numpy as np
import pandas as pd
import joblib
from dataclasses import dataclass
from typing import Optional
import time


@dataclass
class Explanation:
    prediction: float
    base_value: float
    feature_names: list[str]
    shap_values: list[float]
    top_features: list[dict]
    computation_time_ms: float


class SHAPExplainer:
    """Production-ready SHAP explainer for tree-based models."""

    def __init__(self, model, feature_names: list[str], top_k: int = 5):
        self.model = model
        self.feature_names = feature_names
        self.top_k = top_k
        self.explainer = shap.TreeExplainer(model)

    def explain(self, features: np.ndarray) -> Explanation:
        """Generate SHAP explanation for a single prediction."""
        start = time.perf_counter()

        if features.ndim == 1:
            features = features.reshape(1, -1)

        prediction = self.model.predict_proba(features)[0, 1]
        shap_values = self.explainer.shap_values(features)

        # Handle binary classification (take class 1 SHAP values)
        if isinstance(shap_values, list):
            sv = shap_values[1][0]
        else:
            sv = shap_values[0]

        # Get top-K contributing features
        indices = np.argsort(np.abs(sv))[::-1][: self.top_k]
        top_features = [
            {
                "feature": self.feature_names[i],
                "shap_value": round(float(sv[i]), 4),
                "feature_value": round(float(features[0, i]), 4),
                "direction": "increases" if sv[i] > 0 else "decreases",
            }
            for i in indices
        ]

        elapsed_ms = (time.perf_counter() - start) * 1000

        return Explanation(
            prediction=round(float(prediction), 4),
            base_value=round(float(self.explainer.expected_value[1]
                            if isinstance(self.explainer.expected_value, list)
                            else self.explainer.expected_value), 4),
            feature_names=self.feature_names,
            shap_values=[round(float(v), 4) for v in sv],
            top_features=top_features,
            computation_time_ms=round(elapsed_ms, 2),
        )

    def explain_batch(self, features: np.ndarray) -> list[Explanation]:
        """Generate explanations for a batch of predictions."""
        return [self.explain(features[i]) for i in range(len(features))]


# ── Usage ────────────────────────────────────────────────
if __name__ == "__main__":
    from sklearn.ensemble import GradientBoostingClassifier
    from sklearn.datasets import make_classification

    X, y = make_classification(n_samples=1000, n_features=10, random_state=42)
    feature_names = [f"feature_{i}" for i in range(10)]

    model = GradientBoostingClassifier(n_estimators=100, random_state=42)
    model.fit(X, y)

    explainer = SHAPExplainer(model, feature_names, top_k=5)
    explanation = explainer.explain(X[0])

    print(f"Prediction: {explanation.prediction}")
    print(f"Base value: {explanation.base_value}")
    print(f"Time: {explanation.computation_time_ms:.1f} ms")
    print(f"\nTop features:")
    for f in explanation.top_features:
        print(f"  {f['feature']}: {f['shap_value']:+.4f} ({f['direction']})")
```

### Pre-computed SHAP for Speed

```python
# explainability/shap_cache.py
"""Cache SHAP explanations for common input patterns."""
import numpy as np
import hashlib
import json
from typing import Optional
from functools import lru_cache


class SHAPCache:
    """
    Cache explanations for input archetypes to reduce latency.

    Strategy: cluster training data into K archetypes, pre-compute
    SHAP for each, serve cached explanation for nearest archetype.
    """

    def __init__(self, explainer, training_data: np.ndarray, n_archetypes: int = 100):
        from sklearn.cluster import MiniBatchKMeans

        self.explainer = explainer
        self.kmeans = MiniBatchKMeans(n_clusters=n_archetypes, random_state=42)
        self.kmeans.fit(training_data)

        # Pre-compute SHAP for each centroid
        centroids = self.kmeans.cluster_centers_
        self._cached = {}
        for i in range(n_archetypes):
            explanation = explainer.explain(centroids[i])
            self._cached[i] = explanation

    def explain_fast(self, features: np.ndarray) -> dict:
        """Return cached explanation for nearest archetype."""
        if features.ndim == 1:
            features = features.reshape(1, -1)

        cluster_id = int(self.kmeans.predict(features)[0])
        cached = self._cached[cluster_id]

        return {
            "explanation": cached,
            "archetype_id": cluster_id,
            "is_cached": True,
            "note": "Approximate explanation from nearest archetype",
        }

    def explain_exact_or_cached(
        self, features: np.ndarray, distance_threshold: float = 1.0
    ) -> dict:
        """Use cache if close to archetype, compute exact otherwise."""
        if features.ndim == 1:
            features = features.reshape(1, -1)

        cluster_id = int(self.kmeans.predict(features)[0])
        centroid = self.kmeans.cluster_centers_[cluster_id]
        distance = float(np.linalg.norm(features[0] - centroid))

        if distance < distance_threshold:
            return self.explain_fast(features)

        # Compute exact explanation
        exact = self.explainer.explain(features[0])
        return {
            "explanation": exact,
            "archetype_id": cluster_id,
            "distance_to_archetype": round(distance, 4),
            "is_cached": False,
        }
```

---

## LIME at Serving Time

### LIME Explainer

```python
# explainability/lime_serving.py
"""LIME explanations at serving time."""
import lime
import lime.lime_tabular
import numpy as np
import time
from dataclasses import dataclass


@dataclass
class LIMEExplanation:
    prediction: float
    top_features: list[dict]
    intercept: float
    local_prediction: float
    r_squared: float
    computation_time_ms: float


class LIMEExplainer:
    """Production LIME explainer for tabular data."""

    def __init__(
        self,
        model,
        training_data: np.ndarray,
        feature_names: list[str],
        num_samples: int = 500,
        top_k: int = 5,
    ):
        self.model = model
        self.feature_names = feature_names
        self.top_k = top_k
        self.num_samples = num_samples

        self.explainer = lime.lime_tabular.LimeTabularExplainer(
            training_data,
            feature_names=feature_names,
            class_names=["negative", "positive"],
            mode="classification",
        )

    def explain(self, features: np.ndarray) -> LIMEExplanation:
        """Generate LIME explanation for a single prediction."""
        start = time.perf_counter()

        if features.ndim == 1:
            features = features.reshape(1, -1)

        prediction = float(self.model.predict_proba(features)[0, 1])

        exp = self.explainer.explain_instance(
            features[0],
            self.model.predict_proba,
            num_features=self.top_k,
            num_samples=self.num_samples,
        )

        top_features = [
            {
                "feature": name,
                "weight": round(weight, 4),
                "direction": "increases" if weight > 0 else "decreases",
            }
            for name, weight in exp.as_list()
        ]

        elapsed_ms = (time.perf_counter() - start) * 1000

        return LIMEExplanation(
            prediction=round(prediction, 4),
            top_features=top_features,
            intercept=round(float(exp.intercept[1]), 4),
            local_prediction=round(float(exp.local_pred[0]), 4),
            r_squared=round(float(exp.score), 4),
            computation_time_ms=round(elapsed_ms, 2),
        )


# ── Usage ────────────────────────────────────────────────
if __name__ == "__main__":
    from sklearn.ensemble import RandomForestClassifier
    from sklearn.datasets import make_classification

    X, y = make_classification(n_samples=500, n_features=8, random_state=42)
    feature_names = [f"feat_{i}" for i in range(8)]

    model = RandomForestClassifier(n_estimators=50, random_state=42)
    model.fit(X, y)

    explainer = LIMEExplainer(model, X, feature_names, num_samples=300, top_k=5)
    explanation = explainer.explain(X[0])

    print(f"Prediction: {explanation.prediction}")
    print(f"R²: {explanation.r_squared}")
    print(f"Time: {explanation.computation_time_ms:.0f} ms")
    print(f"\nTop features:")
    for f in explanation.top_features:
        print(f"  {f['feature']}: {f['weight']:+.4f} ({f['direction']})")
```

---

## Feature Importance Logging

### Structured Explanation Logging

```python
# explainability/explanation_logger.py
"""Log explanations alongside predictions for audit and debugging."""
import json
import logging
import time
from dataclasses import dataclass, asdict
from typing import Optional
from pathlib import Path

logger = logging.getLogger(__name__)


@dataclass
class PredictionLog:
    request_id: str
    timestamp: float
    model_name: str
    model_version: str
    prediction: float
    top_features: list[dict]
    explanation_method: str
    computation_time_ms: float
    demographic_group: Optional[str] = None


class ExplanationLogger:
    """Log every prediction with its explanation for audit trails."""

    def __init__(self, log_path: str, model_name: str, model_version: str):
        self.model_name = model_name
        self.model_version = model_version
        self.log_path = Path(log_path)
        self.log_path.parent.mkdir(parents=True, exist_ok=True)

        # Structured file logger
        self.file_logger = logging.getLogger(f"explanation.{model_name}")
        handler = logging.FileHandler(self.log_path)
        handler.setFormatter(logging.Formatter("%(message)s"))
        self.file_logger.addHandler(handler)
        self.file_logger.setLevel(logging.INFO)

    def log(
        self,
        request_id: str,
        prediction: float,
        top_features: list[dict],
        explanation_method: str,
        computation_time_ms: float,
        demographic_group: Optional[str] = None,
    ):
        """Log a prediction with its explanation."""
        entry = PredictionLog(
            request_id=request_id,
            timestamp=time.time(),
            model_name=self.model_name,
            model_version=self.model_version,
            prediction=prediction,
            top_features=top_features,
            explanation_method=explanation_method,
            computation_time_ms=computation_time_ms,
            demographic_group=demographic_group,
        )
        self.file_logger.info(json.dumps(asdict(entry), default=str))

    def query_logs(self, n_recent: int = 10) -> list[dict]:
        """Read recent explanation logs (for debugging)."""
        lines = self.log_path.read_text().strip().split("\n")
        return [json.loads(line) for line in lines[-n_recent:]]
```

### Aggregated Feature Importance Tracking

```python
# explainability/feature_tracker.py
"""Track feature importance distribution over time."""
from collections import defaultdict
import numpy as np
from datetime import datetime
import json


class FeatureImportanceTracker:
    """Track how feature contributions shift over time."""

    def __init__(self, feature_names: list[str]):
        self.feature_names = feature_names
        self._windows = []
        self._current = defaultdict(list)

    def record(self, shap_values: dict[str, float]):
        """Record SHAP values from a single prediction."""
        for name, value in shap_values.items():
            self._current[name].append(abs(value))

    def flush_window(self) -> dict:
        """Compute and store stats for the current window."""
        if not self._current:
            return {}

        window = {
            "timestamp": datetime.utcnow().isoformat(),
            "n_predictions": max(len(v) for v in self._current.values()),
            "features": {},
        }

        for name in self.feature_names:
            values = self._current.get(name, [0.0])
            window["features"][name] = {
                "mean_abs_shap": round(float(np.mean(values)), 4),
                "std_abs_shap": round(float(np.std(values)), 4),
                "max_abs_shap": round(float(np.max(values)), 4),
            }

        self._windows.append(window)
        self._current.clear()
        return window

    def detect_importance_drift(self, threshold: float = 0.5) -> list[str]:
        """Detect features whose importance changed significantly."""
        if len(self._windows) < 2:
            return []

        prev = self._windows[-2]["features"]
        curr = self._windows[-1]["features"]
        drifted = []

        for name in self.feature_names:
            prev_mean = prev.get(name, {}).get("mean_abs_shap", 0)
            curr_mean = curr.get(name, {}).get("mean_abs_shap", 0)

            if prev_mean > 0:
                change = abs(curr_mean - prev_mean) / prev_mean
                if change > threshold:
                    drifted.append(name)

        return drifted
```

---

## Explanation APIs

### FastAPI Explanation Endpoint

```python
# serving/explanation_api.py
"""FastAPI service that serves predictions with explanations."""
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import numpy as np
import uuid

app = FastAPI(title="ML Prediction + Explanation API")


class PredictionRequest(BaseModel):
    features: list[float]
    explain: bool = False
    explanation_method: str = "shap"  # "shap" | "lime" | "none"


class FeatureContribution(BaseModel):
    feature: str
    value: float
    shap_value: float
    direction: str


class PredictionResponse(BaseModel):
    request_id: str
    prediction: float
    confidence: float
    explanation: list[FeatureContribution] | None = None
    explanation_method: str | None = None
    computation_time_ms: float


# In production, these would be loaded at startup
# model = joblib.load("model.joblib")
# shap_explainer = SHAPExplainer(model, feature_names)
# lime_explainer = LIMEExplainer(model, training_data, feature_names)


@app.post("/predict", response_model=PredictionResponse)
async def predict(request: PredictionRequest):
    """Predict with optional explanation."""
    import time
    start = time.perf_counter()
    request_id = str(uuid.uuid4())

    features = np.array(request.features).reshape(1, -1)

    # prediction = model.predict_proba(features)[0, 1]
    prediction = 0.73  # placeholder

    explanation = None
    method = None

    if request.explain:
        method = request.explanation_method
        if method == "shap":
            # exp = shap_explainer.explain(features[0])
            explanation = [
                FeatureContribution(
                    feature="income",
                    value=75000,
                    shap_value=0.15,
                    direction="increases",
                ),
                FeatureContribution(
                    feature="debt_ratio",
                    value=0.35,
                    shap_value=-0.08,
                    direction="decreases",
                ),
            ]
        elif method == "lime":
            # exp = lime_explainer.explain(features[0])
            explanation = []
        else:
            raise HTTPException(400, f"Unknown method: {method}")

    elapsed_ms = (time.perf_counter() - start) * 1000

    return PredictionResponse(
        request_id=request_id,
        prediction=prediction,
        confidence=0.73,
        explanation=explanation,
        explanation_method=method,
        computation_time_ms=round(elapsed_ms, 2),
    )


@app.get("/model/feature-importance")
async def global_feature_importance():
    """Return pre-computed global feature importance."""
    # In production: loaded from model artifact at startup
    return {
        "model_name": "loan_approval_v3",
        "method": "mean_abs_shap",
        "features": [
            {"name": "income", "importance": 0.245},
            {"name": "debt_ratio", "importance": 0.198},
            {"name": "credit_score", "importance": 0.187},
            {"name": "employment_years", "importance": 0.132},
            {"name": "loan_amount", "importance": 0.098},
        ],
    }
```

### API Response Format

```
GET /predict
{
    "features": [75000, 0.35, 720, 5, 25000],
    "explain": true,
    "explanation_method": "shap"
}

Response:
{
    "request_id": "a1b2c3d4-...",
    "prediction": 0.73,
    "confidence": 0.73,
    "explanation": [
        {
            "feature": "income",
            "value": 75000,
            "shap_value": 0.15,
            "direction": "increases"
        },
        {
            "feature": "debt_ratio",
            "value": 0.35,
            "shap_value": -0.08,
            "direction": "decreases"
        }
    ],
    "explanation_method": "shap",
    "computation_time_ms": 12.5
}
```

---

## Balancing Latency vs Explainability

### Strategy Matrix

```
┌───────────────────────────────────────────────────────────────┐
│       Latency vs Explainability Decision Matrix               │
│                                                               │
│  Use Case            │ Strategy           │ Latency   │ Notes │
│  ────────────────────┼────────────────────┼───────────┼─────  │
│  Real-time serving   │ Pre-computed       │ ~0 ms     │ Same  │
│  (p99 < 50ms)        │ global importance  │           │ for   │
│                      │                    │           │ all   │
│  ────────────────────┼────────────────────┼───────────┼─────  │
│  Real-time serving   │ TreeSHAP           │ ~1-5 ms   │ Tree  │
│  (p99 < 100ms)       │ (exact)            │           │ only  │
│  ────────────────────┼────────────────────┼───────────┼─────  │
│  On-demand explain   │ Cached archetypes  │ ~1-10 ms  │ ~90%  │
│  (user clicks "why") │ + exact fallback   │ / ~50 ms  │ cache │
│  ────────────────────┼────────────────────┼───────────┼─────  │
│  Async explanation   │ Compute in         │ N/A       │ Best  │
│  (email, dashboard)  │ background queue   │ (async)   │ quality│
│  ────────────────────┼────────────────────┼───────────┼─────  │
│  Audit / compliance  │ Full SHAP/LIME     │ ~1-5 sec  │ Batch │
│                      │ batch job          │ per row   │ OK    │
└───────────────────────────────────────────────────────────────┘
```

### Async Explanation Architecture

```
┌───────────────────────────────────────────────────────────────┐
│           Async Explanation Architecture                      │
│                                                               │
│  User Request                                                 │
│       │                                                       │
│       ▼                                                       │
│  ┌──────────┐                                                 │
│  │  Model   │──► prediction (fast, ~5 ms)                     │
│  │  Server  │                                                 │
│  └────┬─────┘                                                 │
│       │                                                       │
│       ├──► Return prediction immediately                      │
│       │                                                       │
│       └──► Queue explanation job                              │
│               │                                               │
│               ▼                                               │
│       ┌──────────────┐                                        │
│       │ Explanation  │                                        │
│       │ Worker       │──► Compute SHAP/LIME (~200ms-5s)       │
│       │ (background) │                                        │
│       └──────┬───────┘                                        │
│              │                                                │
│              ▼                                                │
│       ┌──────────────┐                                        │
│       │ Explanation  │  ← User can fetch later via            │
│       │ Store        │    GET /explanations/{request_id}      │
│       │ (Redis/DB)   │                                        │
│       └──────────────┘                                        │
└───────────────────────────────────────────────────────────────┘
```

### Async Explanation Worker

```python
# serving/async_explanation.py
"""Async explanation computation with Redis queue."""
import json
import time
from dataclasses import asdict


class AsyncExplanationService:
    """Decouple prediction from explanation to maintain low latency."""

    def __init__(self, explainer, redis_client, ttl_seconds: int = 3600):
        self.explainer = explainer
        self.redis = redis_client
        self.ttl = ttl_seconds

    def enqueue(self, request_id: str, features: list[float]):
        """Queue an explanation job (called from prediction endpoint)."""
        self.redis.lpush(
            "explanation_queue",
            json.dumps({"request_id": request_id, "features": features}),
        )

    def process_queue(self):
        """Worker loop: process explanation requests from queue."""
        while True:
            _, job_raw = self.redis.brpop("explanation_queue")
            job = json.loads(job_raw)

            try:
                import numpy as np
                features = np.array(job["features"])
                explanation = self.explainer.explain(features)

                self.redis.setex(
                    f"explanation:{job['request_id']}",
                    self.ttl,
                    json.dumps(asdict(explanation), default=str),
                )
            except Exception as e:
                self.redis.setex(
                    f"explanation:{job['request_id']}",
                    self.ttl,
                    json.dumps({"error": str(e)}),
                )

    def get_explanation(self, request_id: str) -> dict | None:
        """Retrieve a computed explanation."""
        result = self.redis.get(f"explanation:{request_id}")
        if result:
            return json.loads(result)
        return None
```

---

## Model Cards

### What Is a Model Card?

A **model card** is a standardized document that describes a model's intended use, performance characteristics, limitations, and ethical considerations. Think of it as a nutrition label for ML models.

### Model Card Template

```python
# explainability/model_card.py
"""Generate model cards for ML models."""
from dataclasses import dataclass, field, asdict
from datetime import datetime
import json


@dataclass
class ModelCard:
    # ── Model Details ────────────────────────────────────
    model_name: str = ""
    model_version: str = ""
    model_type: str = ""
    description: str = ""
    owner: str = ""
    created_date: str = ""
    last_updated: str = ""
    framework: str = ""
    license: str = ""

    # ── Intended Use ─────────────────────────────────────
    primary_use: str = ""
    primary_users: list[str] = field(default_factory=list)
    out_of_scope_uses: list[str] = field(default_factory=list)

    # ── Training Data ────────────────────────────────────
    training_data_description: str = ""
    training_data_size: int = 0
    training_data_date_range: str = ""
    preprocessing_steps: list[str] = field(default_factory=list)

    # ── Evaluation Data ──────────────────────────────────
    evaluation_data_description: str = ""
    evaluation_data_size: int = 0

    # ── Performance Metrics ──────────────────────────────
    overall_metrics: dict = field(default_factory=dict)
    per_group_metrics: dict = field(default_factory=dict)

    # ── Fairness Metrics ─────────────────────────────────
    fairness_metrics: dict = field(default_factory=dict)
    sensitive_attributes_tested: list[str] = field(default_factory=list)

    # ── Limitations ──────────────────────────────────────
    known_limitations: list[str] = field(default_factory=list)
    known_failure_modes: list[str] = field(default_factory=list)

    # ── Ethical Considerations ───────────────────────────
    ethical_considerations: list[str] = field(default_factory=list)
    risks: list[str] = field(default_factory=list)
    mitigation_strategies: list[str] = field(default_factory=list)

    def to_json(self) -> str:
        return json.dumps(asdict(self), indent=2)

    def to_markdown(self) -> str:
        """Render model card as markdown."""
        lines = [
            f"# Model Card: {self.model_name}",
            f"\n**Version:** {self.model_version} | **Owner:** {self.owner} | **Updated:** {self.last_updated}",
            f"\n## Model Details\n",
            f"- **Type:** {self.model_type}",
            f"- **Framework:** {self.framework}",
            f"- **Description:** {self.description}",
            f"\n## Intended Use\n",
            f"- **Primary use:** {self.primary_use}",
            f"- **Users:** {', '.join(self.primary_users)}",
            f"- **Out of scope:** {', '.join(self.out_of_scope_uses)}",
            f"\n## Training Data\n",
            f"- **Description:** {self.training_data_description}",
            f"- **Size:** {self.training_data_size:,} examples",
            f"\n## Performance\n",
            "| Metric | Value |",
            "|--------|-------|",
        ]
        for k, v in self.overall_metrics.items():
            lines.append(f"| {k} | {v} |")

        if self.fairness_metrics:
            lines.append(f"\n## Fairness\n")
            lines.append(f"Sensitive attributes tested: {', '.join(self.sensitive_attributes_tested)}\n")
            lines.append("| Metric | Value |")
            lines.append("|--------|-------|")
            for k, v in self.fairness_metrics.items():
                lines.append(f"| {k} | {v} |")

        if self.known_limitations:
            lines.append(f"\n## Limitations\n")
            for lim in self.known_limitations:
                lines.append(f"- {lim}")

        if self.ethical_considerations:
            lines.append(f"\n## Ethical Considerations\n")
            for ec in self.ethical_considerations:
                lines.append(f"- {ec}")

        return "\n".join(lines)


# ── Usage ────────────────────────────────────────────────
if __name__ == "__main__":
    card = ModelCard(
        model_name="Loan Approval Model",
        model_version="3.2.1",
        model_type="GradientBoostingClassifier",
        description="Predicts probability of loan repayment",
        owner="ML Platform Team",
        created_date="2024-01-15",
        last_updated="2024-06-01",
        framework="scikit-learn 1.4",
        primary_use="Automated loan pre-screening",
        primary_users=["Loan officers", "Automated pipeline"],
        out_of_scope_uses=[
            "Final loan decision (requires human review)",
            "Applicants under 18",
        ],
        training_data_description="US loan applications 2020-2023",
        training_data_size=500_000,
        overall_metrics={"accuracy": 0.923, "f1": 0.891, "auc_roc": 0.956},
        fairness_metrics={
            "demographic_parity_diff": 0.07,
            "equalized_odds_diff": 0.09,
            "disparate_impact_ratio": 0.84,
        },
        sensitive_attributes_tested=["gender", "race", "age_group"],
        known_limitations=[
            "Performance degrades for applicants with thin credit files",
            "Not validated for commercial loans",
        ],
        ethical_considerations=[
            "Model should not be used as sole decision-maker",
            "Adverse action notices required for denials",
        ],
    )

    print(card.to_markdown())
```

---

## Summary

| Concept | Key Takeaway |
|---|---|
| **SHAP at serving time** | TreeSHAP is fast (~ms) for tree models; KernelSHAP is slow (~sec) but model-agnostic |
| **LIME at serving time** | Generates local surrogate models; slower but highly interpretable for any model |
| **Feature importance logging** | Log top-K SHAP values with every prediction for audit trails and debugging |
| **Explanation APIs** | Serve explanations via REST endpoints; support optional `?explain=true` parameter |
| **Latency trade-offs** | Pre-computed (~0ms) → TreeSHAP (~5ms) → Cached archetypes (~10ms) → Full LIME (~5s) |
| **Async explanations** | Compute expensive explanations in background workers; users fetch later |
| **Model cards** | Standardized documentation of model purpose, performance, fairness, and limitations |

> **Rule of thumb:** Always have a fast explanation path (pre-computed or TreeSHAP) for real-time serving, and a thorough explanation path (full SHAP/LIME) for audit and compliance. Log explanations for every prediction — you'll need them when regulators or users ask "why?"

---

## Revision Questions

1. **Compare** TreeSHAP and LIME for a production credit scoring model. Which would you use for real-time explanations at the API level, and which for quarterly compliance audits? Justify your answer.

2. **Design** a caching strategy for SHAP explanations that achieves <10ms p99 latency while providing accurate-enough explanations for 90% of requests. What data structure and invalidation policy would you use?

3. **A user** receives a loan denial and requests an explanation. Design the API response that is both technically accurate and understandable to a non-technical person. What information should be included and excluded?

4. **Your model card** shows that the model has a known limitation for thin-credit-file applicants. How would you operationalize this — what guardrails, fallbacks, or alternative paths would you implement in the serving pipeline?

5. **Explain** the trade-off between synchronous and asynchronous explanation generation. Under what circumstances would you choose each approach, and what infrastructure is required for the async pattern?

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Fairness Monitoring](./02-fairness-monitoring.md) | [📂 Responsible AI](./README.md) | [Model Governance →](./04-model-governance.md) |
