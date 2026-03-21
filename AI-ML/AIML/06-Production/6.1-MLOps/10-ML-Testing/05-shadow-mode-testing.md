# 👻 Shadow Mode Testing

> **Unit 10 · ML Testing · Chapter 5**
>
> How to safely evaluate a new model by running it alongside production without serving its predictions — shadow deployment architecture, comparing shadow vs production outputs, logging shadow predictions, analyzing divergence, and deciding when to promote.

---

## Table of Contents

1. [Chapter Overview](#chapter-overview)
2. [What Is Shadow Mode?](#what-is-shadow-mode)
3. [Shadow Mode Architecture](#shadow-mode-architecture)
4. [Implementing Shadow Deployments](#implementing-shadow-deployments)
5. [Logging Shadow Predictions](#logging-shadow-predictions)
6. [Comparing Shadow vs Production](#comparing-shadow-vs-production)
7. [Decision Framework: When to Promote](#decision-framework-when-to-promote)
8. [Shadow Mode vs Other Strategies](#shadow-mode-vs-other-strategies)
9. [Summary](#summary)
10. [Revision Questions](#revision-questions)
11. [Navigation](#navigation)

---

## Chapter Overview

Staging tests and golden datasets are valuable, but they can't replicate the **true distribution of production traffic**. A model might pass every offline test and still behave differently on real requests — unusual feature combinations, edge-case inputs, latency under load. **Shadow mode** (also called **dark launching**) solves this by deploying the new model alongside the current production model: every request is sent to both, but only the production model's predictions are served to users. The shadow model's predictions are logged and compared. This gives you a risk-free way to evaluate a candidate model on real traffic before it ever affects a single user. This chapter covers the architecture, implementation, logging strategy, comparison analysis, and promotion decision framework.

```
Traditional Deployment                Shadow Mode Deployment
┌──────────────────────┐             ┌──────────────────────────────────┐
│                      │             │                                  │
│  Users ──► Model v2  │             │  Users ──► Model v1 ──► Response │
│                      │             │       │                          │
│  Hope for the best 🤞│             │       └──► Model v2 ──► Log only │
│                      │             │             (shadow)    (not     │
│  Rollback if broken  │             │                         served)  │
└──────────────────────┘             └──────────────────────────────────┘
     Risk: YOLO deploy                    Risk: Zero — users see v1 only
```

---

## What Is Shadow Mode?

### Definition

Shadow mode is a deployment pattern where a **candidate model** receives the same inputs as the **production model** but its outputs are **logged, not served**. Users are unaffected — they always receive predictions from the production model.

### When to Use Shadow Mode

| Use Case | Why Shadow Mode? |
|---|---|
| **Major model architecture change** | New model type may behave very differently |
| **New feature set** | Features may interact unexpectedly with real data |
| **Retraining on new data** | Data distribution may have shifted |
| **High-stakes predictions** | Financial, medical — can't afford mistakes |
| **Latency-sensitive services** | Verify new model meets latency SLA |
| **Regulatory compliance** | Need evidence that new model is safe |

### What Shadow Mode Is NOT

```
┌──────────────────────────────────────────────────────────────┐
│  Shadow Mode ≠ A/B Testing                                   │
│                                                              │
│  Shadow Mode:                                                │
│  • Both models get ALL traffic                               │
│  • Only production model's output is served                  │
│  • Zero user impact                                          │
│  • Compares model outputs (predictions)                      │
│                                                              │
│  A/B Testing:                                                │
│  • Traffic is SPLIT between models                           │
│  • Both models' outputs are served (to different users)      │
│  • Users in test group see new model                         │
│  • Compares business outcomes (click rate, revenue)          │
│                                                              │
│  Shadow mode answers: "Are the predictions similar?"         │
│  A/B testing answers:  "Do users prefer the new model?"      │
└──────────────────────────────────────────────────────────────┘
```

---

## Shadow Mode Architecture

### Full Architecture Diagram

```
                              ┌─────────────────────────┐
                              │     Load Balancer        │
                              └────────────┬────────────┘
                                           │
                                           ▼
                              ┌─────────────────────────┐
                              │     API Gateway /        │
                              │     Request Router       │
                              └──────┬──────────┬───────┘
                                     │          │
                         ┌───────────┘          └───────────┐
                         │                                  │
                         ▼                                  ▼
              ┌─────────────────────┐          ┌─────────────────────┐
              │   Production Model   │          │    Shadow Model     │
              │   (Model v1)         │          │    (Model v2)       │
              │                     │          │                     │
              │   Serves response   │          │   Logs only         │
              │   to user           │          │   (async, non-      │
              │                     │          │    blocking)        │
              └──────────┬──────────┘          └──────────┬──────────┘
                         │                                │
                         │     ┌──────────────────┐       │
                         └────►│  Prediction Log  │◄──────┘
                               │  (unified store) │
                               └────────┬─────────┘
                                        │
                                        ▼
                               ┌──────────────────┐
                               │  Comparison       │
                               │  Dashboard        │
                               │                  │
                               │  • Agreement rate │
                               │  • Latency diff   │
                               │  • Error rates    │
                               └──────────────────┘
```

### Key Design Principles

```
┌──────────────────────────────────────────────────────────────┐
│              Shadow Mode Design Principles                   │
│                                                              │
│  1. NON-BLOCKING                                             │
│     Shadow inference must not slow down production response  │
│     → Run shadow async or in a separate thread/process       │
│                                                              │
│  2. FAULT-ISOLATED                                           │
│     Shadow model crash must not affect production             │
│     → Wrap in try/except, run in separate container          │
│                                                              │
│  3. SAME INPUT                                               │
│     Shadow must receive IDENTICAL input to production         │
│     → Fork the request at the router level                   │
│                                                              │
│  4. COMPLETE LOGGING                                         │
│     Log both predictions with same request_id                │
│     → Enables row-level comparison                           │
│                                                              │
│  5. TIME-BOUNDED                                             │
│     Shadow test should have a defined end date               │
│     → 1–2 weeks typical; promote or kill                     │
└──────────────────────────────────────────────────────────────┘
```

---

## Implementing Shadow Deployments

### Approach 1: Application-Level Shadow (Python)

```python
# src/shadow_router.py
import asyncio
import logging
import time
import uuid
from dataclasses import dataclass
from typing import Any

logger = logging.getLogger(__name__)


@dataclass
class ShadowResult:
    request_id: str
    production_prediction: Any
    shadow_prediction: Any | None
    production_latency_ms: float
    shadow_latency_ms: float | None
    shadow_error: str | None


class ShadowRouter:
    def __init__(self, production_model, shadow_model, logger_backend):
        self.production = production_model
        self.shadow = shadow_model
        self.logger_backend = logger_backend

    async def predict(self, features: dict) -> dict:
        request_id = str(uuid.uuid4())

        # Production prediction — synchronous, always served
        start = time.perf_counter()
        prod_prediction = self.production.predict(features)
        prod_latency = (time.perf_counter() - start) * 1000

        # Shadow prediction — async, fire-and-forget
        asyncio.create_task(
            self._shadow_predict(request_id, features, prod_prediction, prod_latency)
        )

        return prod_prediction

    async def _shadow_predict(
        self, request_id, features, prod_prediction, prod_latency
    ):
        """Run shadow prediction asynchronously — never blocks production."""
        shadow_pred = None
        shadow_latency = None
        shadow_error = None

        try:
            start = time.perf_counter()
            shadow_pred = self.shadow.predict(features)
            shadow_latency = (time.perf_counter() - start) * 1000
        except Exception as e:
            shadow_error = str(e)
            logger.warning(f"Shadow model error: {e}")

        # Log both predictions
        await self.logger_backend.log(ShadowResult(
            request_id=request_id,
            production_prediction=prod_prediction,
            shadow_prediction=shadow_pred,
            production_latency_ms=prod_latency,
            shadow_latency_ms=shadow_latency,
            shadow_error=shadow_error,
        ))
```

### Approach 2: Infrastructure-Level Shadow (Istio)

```yaml
# k8s/shadow-deployment.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: model-api
spec:
  hosts:
    - model-api.example.com
  http:
    - route:
        - destination:
            host: model-v1          # Production — gets response
            port:
              number: 8080
      mirror:
        host: model-v2-shadow       # Shadow — mirrored traffic
        port:
          number: 8080
      mirrorPercentage:
        value: 100.0                # Mirror 100% of traffic
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: model-v2-shadow
spec:
  replicas: 2                       # Fewer replicas than prod (cost saving)
  selector:
    matchLabels:
      app: model-v2-shadow
  template:
    metadata:
      labels:
        app: model-v2-shadow
    spec:
      containers:
        - name: model
          image: registry.example.com/model:v2-candidate
          ports:
            - containerPort: 8080
          env:
            - name: SHADOW_MODE
              value: "true"          # Model knows it's in shadow mode
          resources:
            requests:
              cpu: "1"
              memory: "2Gi"
```

### Approach 3: FastAPI with Shadow Middleware

```python
# src/app_with_shadow.py
from fastapi import FastAPI, Request
from contextlib import asynccontextmanager
import asyncio
import logging

logger = logging.getLogger(__name__)


def create_app(prod_model, shadow_model=None):
    app = FastAPI()

    @app.post("/predict")
    async def predict(request: Request):
        body = await request.json()

        # Always: production prediction
        prod_result = prod_model.predict(body)

        # If shadow model is configured, run async
        if shadow_model is not None:
            asyncio.create_task(
                run_shadow(body, prod_result, shadow_model)
            )

        return prod_result

    return app


async def run_shadow(features, prod_result, shadow_model):
    """Fire-and-forget shadow prediction."""
    try:
        shadow_result = shadow_model.predict(features)
        log_comparison(features, prod_result, shadow_result)
    except Exception as e:
        logger.error(f"Shadow prediction failed: {e}")


def log_comparison(features, prod_result, shadow_result):
    """Log prediction comparison for later analysis."""
    import json
    record = {
        "features": features,
        "production": prod_result,
        "shadow": shadow_result,
        "agree": prod_result.get("label") == shadow_result.get("label"),
    }
    # Write to log file, database, or event stream
    logger.info(json.dumps(record))
```

---

## Logging Shadow Predictions

### What to Log

```
┌──────────────────────────────────────────────────────────────┐
│                   Shadow Prediction Log                       │
│                                                              │
│  For each request, log:                                      │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  request_id:       "abc-123-def"                     │    │
│  │  timestamp:        "2024-03-15T14:30:22Z"            │    │
│  │  features_hash:    "sha256:9f3a..."                  │    │
│  │                                                      │    │
│  │  production:                                         │    │
│  │    model_version:  "v1.3"                            │    │
│  │    prediction:     {"label": 1, "prob": 0.87}        │    │
│  │    latency_ms:     23                                │    │
│  │                                                      │    │
│  │  shadow:                                             │    │
│  │    model_version:  "v2.0-candidate"                  │    │
│  │    prediction:     {"label": 1, "prob": 0.92}        │    │
│  │    latency_ms:     31                                │    │
│  │    error:          null                              │    │
│  │                                                      │    │
│  │  agreement:        true                              │    │
│  │  prob_diff:        0.05                              │    │
│  └──────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────┘
```

### Implementation

```python
# src/shadow_logger.py
import json
import hashlib
from datetime import datetime, timezone
from dataclasses import dataclass, asdict
from typing import Any
import logging

logger = logging.getLogger("shadow")


@dataclass
class ShadowLogEntry:
    request_id: str
    timestamp: str
    features_hash: str
    prod_version: str
    prod_label: Any
    prod_prob: float
    prod_latency_ms: float
    shadow_version: str
    shadow_label: Any | None
    shadow_prob: float | None
    shadow_latency_ms: float | None
    shadow_error: str | None
    labels_agree: bool
    prob_diff: float | None


class ShadowLogger:
    def __init__(self, output_path: str = "logs/shadow_predictions.jsonl"):
        self.output_path = output_path

    def log(
        self,
        request_id: str,
        features: dict,
        prod_result: dict,
        shadow_result: dict | None,
        prod_latency_ms: float,
        shadow_latency_ms: float | None,
        shadow_error: str | None = None,
    ):
        features_hash = hashlib.sha256(
            json.dumps(features, sort_keys=True).encode()
        ).hexdigest()[:16]

        shadow_label = shadow_result.get("label") if shadow_result else None
        shadow_prob = shadow_result.get("probability") if shadow_result else None
        prod_label = prod_result["label"]
        prod_prob = prod_result["probability"]

        entry = ShadowLogEntry(
            request_id=request_id,
            timestamp=datetime.now(timezone.utc).isoformat(),
            features_hash=features_hash,
            prod_version=prod_result.get("model_version", "unknown"),
            prod_label=prod_label,
            prod_prob=prod_prob,
            prod_latency_ms=round(prod_latency_ms, 2),
            shadow_version=shadow_result.get("model_version", "unknown") if shadow_result else "error",
            shadow_label=shadow_label,
            shadow_prob=shadow_prob,
            shadow_latency_ms=round(shadow_latency_ms, 2) if shadow_latency_ms else None,
            shadow_error=shadow_error,
            labels_agree=prod_label == shadow_label if shadow_label is not None else False,
            prob_diff=round(abs(prod_prob - shadow_prob), 4) if shadow_prob is not None else None,
        )

        with open(self.output_path, "a") as f:
            f.write(json.dumps(asdict(entry)) + "\n")
```

---

## Comparing Shadow vs Production

### Analysis Script

```python
# scripts/analyze_shadow.py
"""
Analyze shadow predictions vs production predictions.
Run daily during shadow testing period.
"""
import pandas as pd
import numpy as np
from scipy import stats


def analyze_shadow_results(log_path: str) -> dict:
    df = pd.read_json(log_path, lines=True)
    df = df[df["shadow_error"].isna()]   # Exclude errors

    total = len(df)
    report = {}

    # --- 1. Agreement Rate ---
    agreement_rate = df["labels_agree"].mean()
    report["agreement_rate"] = f"{agreement_rate:.2%}"

    # --- 2. Probability Difference ---
    prob_diffs = df["prob_diff"].dropna()
    report["mean_prob_diff"] = f"{prob_diffs.mean():.4f}"
    report["median_prob_diff"] = f"{prob_diffs.median():.4f}"
    report["p95_prob_diff"] = f"{prob_diffs.quantile(0.95):.4f}"

    # --- 3. Latency Comparison ---
    report["prod_p50_latency_ms"] = f"{df['prod_latency_ms'].median():.1f}"
    report["shadow_p50_latency_ms"] = f"{df['shadow_latency_ms'].median():.1f}"
    report["prod_p99_latency_ms"] = f"{df['prod_latency_ms'].quantile(0.99):.1f}"
    report["shadow_p99_latency_ms"] = f"{df['shadow_latency_ms'].quantile(0.99):.1f}"

    # --- 4. Disagreement Analysis ---
    disagreements = df[~df["labels_agree"]]
    report["disagreement_count"] = len(disagreements)
    report["disagreement_rate"] = f"{len(disagreements)/total:.2%}"

    # --- 5. Error Rate ---
    error_df = pd.read_json(log_path, lines=True)
    error_count = error_df["shadow_error"].notna().sum()
    report["shadow_error_rate"] = f"{error_count/len(error_df):.4%}"

    return report


def print_shadow_report(report: dict):
    print("=" * 60)
    print("        SHADOW MODE ANALYSIS REPORT")
    print("=" * 60)
    print()
    for key, value in report.items():
        label = key.replace("_", " ").title()
        print(f"  {label:.<40} {value}")
    print()
    print("=" * 60)


if __name__ == "__main__":
    report = analyze_shadow_results("logs/shadow_predictions.jsonl")
    print_shadow_report(report)
```

### Dashboard Metrics

```
┌──────────────────────────────────────────────────────────────┐
│             Shadow Mode Dashboard                            │
│                                                              │
│  Agreement Rate        ████████████████████░░  96.3%  ✅     │
│  Shadow Error Rate     ░░░░░░░░░░░░░░░░░░░░░   0.02% ✅     │
│  Mean Prob Difference  ██░░░░░░░░░░░░░░░░░░░░   0.034 ✅     │
│                                                              │
│  Latency (p50)                                               │
│    Production:  23 ms                                        │
│    Shadow:      31 ms  (+35%)                         ⚠️     │
│                                                              │
│  Latency (p99)                                               │
│    Production:  85 ms                                        │
│    Shadow:     142 ms  (+67%)                         ⚠️     │
│                                                              │
│  Disagreements by Segment:                                   │
│    High-value users:   2.1%  ✅                              │
│    New users:          8.7%  ⚠️  ← investigate              │
│    Category "C":      14.2%  ❌  ← blocking issue           │
│                                                              │
│  Duration:  Day 5 of 14                                      │
│  Requests analyzed: 1,234,567                                │
└──────────────────────────────────────────────────────────────┘
```

---

## Decision Framework: When to Promote

### Promotion Criteria

```
┌─────────────────────────────────────────────────────────────┐
│              Shadow → Production Promotion Gates            │
│                                                             │
│  Gate 1: AGREEMENT                                          │
│  ├── Label agreement rate > 95%                             │
│  └── Mean probability diff < 0.05                           │
│                                                             │
│  Gate 2: RELIABILITY                                        │
│  ├── Shadow error rate < 0.1%                               │
│  └── No shadow crashes in 48 hours                          │
│                                                             │
│  Gate 3: LATENCY                                            │
│  ├── Shadow p50 latency < 1.5× production p50              │
│  └── Shadow p99 latency < production SLA                    │
│                                                             │
│  Gate 4: FAIRNESS                                           │
│  ├── No segment with > 10% disagreement                     │
│  └── Disagreement not correlated with protected attributes  │
│                                                             │
│  Gate 5: DURATION                                           │
│  ├── Minimum 7 days of shadow data                          │
│  └── Minimum 100,000 requests analyzed                      │
│                                                             │
│  ALL GATES PASS ──► Promote to production via canary        │
│  ANY GATE FAILS  ──► Investigate, fix, restart shadow       │
└─────────────────────────────────────────────────────────────┘
```

### Automated Promotion Check

```python
# scripts/check_promotion.py
"""
Automated check: should we promote shadow model to production?
Run daily during shadow period. Exits with code 0 (promote) or 1 (wait).
"""
import pandas as pd
import sys


def check_promotion_gates(log_path: str) -> tuple[bool, list[str]]:
    df = pd.read_json(log_path, lines=True)
    clean = df[df["shadow_error"].isna()]
    failures = []

    # Gate 1: Agreement
    agreement = clean["labels_agree"].mean()
    if agreement < 0.95:
        failures.append(f"Agreement {agreement:.2%} < 95%")

    mean_diff = clean["prob_diff"].mean()
    if mean_diff > 0.05:
        failures.append(f"Mean prob diff {mean_diff:.4f} > 0.05")

    # Gate 2: Reliability
    error_rate = df["shadow_error"].notna().mean()
    if error_rate > 0.001:
        failures.append(f"Error rate {error_rate:.4%} > 0.1%")

    # Gate 3: Latency
    prod_p50 = clean["prod_latency_ms"].median()
    shadow_p50 = clean["shadow_latency_ms"].median()
    if shadow_p50 > prod_p50 * 1.5:
        failures.append(
            f"Shadow p50 {shadow_p50:.0f}ms > 1.5× prod {prod_p50:.0f}ms"
        )

    # Gate 4: Duration
    if len(clean) < 100_000:
        failures.append(f"Only {len(clean):,} requests (need 100,000)")

    days = (
        pd.to_datetime(df["timestamp"]).max() -
        pd.to_datetime(df["timestamp"]).min()
    ).days
    if days < 7:
        failures.append(f"Only {days} days of data (need 7)")

    return len(failures) == 0, failures


if __name__ == "__main__":
    promote, failures = check_promotion_gates("logs/shadow_predictions.jsonl")
    if promote:
        print("✅ ALL GATES PASSED — safe to promote")
        sys.exit(0)
    else:
        print("❌ PROMOTION BLOCKED:")
        for f in failures:
            print(f"  • {f}")
        sys.exit(1)
```

---

## Shadow Mode vs Other Strategies

### Comparison

| Strategy | User Impact | Traffic Split | Measures | Duration | Risk |
|---|---|---|---|---|---|
| **Shadow mode** | None (zero) | 100% to both | Prediction agreement | 1–2 weeks | None |
| **A/B test** | Yes (subset) | Split (e.g., 50/50) | Business KPIs | 2–4 weeks | Medium |
| **Canary** | Yes (small %) | 1–5% to new | Errors, latency | Hours–days | Low |
| **Blue/green** | Instant switch | 100% switch | All metrics | Minutes | High |
| **Feature flag** | Per-user toggle | Configurable | Varies | Varies | Low |

### Recommended Deployment Sequence

```
Step 1: Shadow Mode                  Step 2: Canary                     Step 3: Full Rollout
┌──────────────────────┐            ┌──────────────────────┐            ┌──────────────────────┐
│                      │            │                      │            │                      │
│  v1 ──► Response     │  promote   │  v1 ──► 95% traffic  │  expand    │  v2 ──► 100% traffic │
│  v2 ──► Log only     │  ───────►  │  v2 ──► 5% traffic   │  ───────►  │                      │
│                      │            │                      │            │  v1 retired           │
│  Duration: 1-2 weeks │            │  Duration: 24-72 hrs │            │                      │
└──────────────────────┘            └──────────────────────┘            └──────────────────────┘

  Answers: "Are            Answers: "Are            Full deployment
  predictions              business metrics         with confidence
  compatible?"             preserved?"
```

### Cost Considerations

```
┌──────────────────────────────────────────────────────────────┐
│                 Shadow Mode Cost                             │
│                                                              │
│  Extra compute:                                              │
│  • Shadow model needs its own pods/GPUs                      │
│  • Typically 50-100% of prod compute added                   │
│  • Can reduce by sampling (shadow 10% of traffic)            │
│                                                              │
│  Storage:                                                    │
│  • Prediction logs (both models) for comparison              │
│  • ~1 KB per request × 1M requests/day = ~1 GB/day          │
│                                                              │
│  Engineering:                                                │
│  • Shadow routing infrastructure (one-time)                  │
│  • Comparison dashboard (one-time)                           │
│  • Analysis script (reusable)                                │
│                                                              │
│  Savings:                                                    │
│  • Avoids production incidents (cost: $10K–$1M each)         │
│  • Worth it for any high-stakes model                        │
└──────────────────────────────────────────────────────────────┘
```

---

## Summary

| Concept | Key Takeaway |
|---|---|
| **What is shadow mode** | New model runs on real traffic but predictions are logged, not served |
| **Architecture** | Fork requests at router level; shadow runs async, non-blocking |
| **Implementation** | Application-level (Python async) or infrastructure-level (Istio mirror) |
| **Logging** | Log both predictions with same request_id; include latency and errors |
| **Comparison** | Agreement rate, probability diff, latency ratio, error rate, per-segment |
| **Promotion gates** | Agreement > 95%, error < 0.1%, latency < 1.5×, min 7 days, 100K requests |
| **vs A/B testing** | Shadow = zero risk (no user impact); A/B = measures business outcomes |
| **Deployment sequence** | Shadow → Canary → Full rollout (each answers a different question) |

> **Rule of thumb:** Shadow mode is the safest way to validate a new model on real traffic. It answers "will this model produce similar predictions?" without risking a single user. Use it for every major model update, then follow with a canary deployment to validate business metrics.

---

## Revision Questions

1. **Explain** the difference between shadow mode and A/B testing. For a fraud detection model, which would you use first and why?

2. **Design** a shadow mode architecture for a recommendation system serving 50,000 requests/minute. Address non-blocking execution, fault isolation, logging, and cost optimization.

3. **A shadow analysis** shows 96% agreement overall, but only 78% agreement for users in the "new user" segment. What would you investigate, and would you promote the model?

4. **Compare** application-level shadow routing (Python async) vs infrastructure-level (Istio mirroring). What are the tradeoffs in terms of flexibility, operational complexity, and fault isolation?

5. **Your shadow model** has better offline accuracy (F1: 0.93 vs 0.89) but shadow analysis shows the p99 latency is 2.5× production. How would you investigate and resolve this before promotion?

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Integration Testing](./04-integration-testing.md) | [📂 ML Testing](./README.md) | [CI/CD for ML →](./06-cicd-for-ml.md) |
