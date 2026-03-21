# 📦 Deployment Strategies for ML Models

> **Unit 7 · Chapter 1** — A comprehensive overview of deployment strategies: direct, shadow, canary, blue-green, and A/B deployments.

---

## 📋 Chapter Overview

Deploying a new ML model to production is one of the riskiest steps in the MLOps lifecycle. A poor deployment can degrade user experience, introduce silent failures, or cause revenue loss. **Deployment strategies** provide structured approaches for rolling out new models while controlling risk.

This chapter covers:

- The five core deployment strategies for ML systems
- Risk-vs-speed tradeoffs for each approach
- Architecture patterns and decision criteria
- When to use which strategy

---

## 🔁 The Deployment Spectrum

```
  LOW RISK / SLOW                                    HIGH RISK / FAST
  ◄─────────────────────────────────────────────────────────────────►

  Shadow ──► Canary ──► Blue-Green ──► A/B Testing ──► Direct (Big Bang)
    │            │            │              │               │
    │            │            │              │               └─ All traffic
    │            │            │              │                  switches at once
    │            │            │              └─ Split traffic
    │            │            │                 for experiments
    │            │            └─ Two full envs,
    │            │               instant swap
    │            └─ Gradual rollout
    │               (1% → 10% → 100%)
    └─ New model gets
       real traffic but
       responses are NOT served
```

---

## 1️⃣ Direct Deployment (Big Bang)

The simplest strategy — replace the old model with the new one for all users simultaneously.

```
  BEFORE                        AFTER
  ┌──────────┐                  ┌──────────┐
  │ Model v1 │  ──────────►     │ Model v2 │
  └──────────┘                  └──────────┘
       ▲                             ▲
       │                             │
  All Traffic                   All Traffic
```

### Characteristics

| Aspect         | Detail                                      |
|----------------|---------------------------------------------|
| **Risk**       | 🔴 High — no fallback if model fails        |
| **Complexity** | 🟢 Low — simple swap                        |
| **Rollback**   | Manual redeploy of previous version         |
| **Best For**   | Non-critical systems, dev/staging, MVP apps |

### Python Example — Simple Model Swap

```python
import joblib
import os
import shutil

def direct_deploy(new_model_path: str, production_path: str):
    """Replace production model with new model (big bang)."""
    # Backup current model
    backup_path = production_path + ".backup"
    if os.path.exists(production_path):
        shutil.copy2(production_path, backup_path)
        print(f"Backed up current model to {backup_path}")

    # Swap model
    shutil.copy2(new_model_path, production_path)
    print(f"Deployed new model from {new_model_path}")

    # Validate
    model = joblib.load(production_path)
    print(f"Model loaded successfully: {type(model).__name__}")
    return model

# Usage
direct_deploy("models/v2/model.pkl", "serving/model.pkl")
```

---

## 2️⃣ Shadow Deployment

The new model receives real traffic and generates predictions, but **responses are not served to users**. Instead, predictions are logged for offline evaluation.

```
                     ┌──────────┐
  Request ──────────►│ Model v1 │──────► Response (served)
       │             └──────────┘
       │
       └────────────►┌──────────┐
                     │ Model v2 │──────► Log only (NOT served)
                     └──────────┘
                          │
                          ▼
                   ┌─────────────┐
                   │  Evaluation │
                   │   Pipeline  │
                   └─────────────┘
```

### Characteristics

| Aspect         | Detail                                               |
|----------------|------------------------------------------------------|
| **Risk**       | 🟢 Lowest — users never see v2 predictions           |
| **Complexity** | 🟡 Medium — dual inference + logging infrastructure  |
| **Cost**       | 🔴 High — runs two models simultaneously             |
| **Best For**   | High-stakes models (finance, healthcare)              |

### Python Example — Shadow Inference

```python
import logging
from typing import Any, Dict

logger = logging.getLogger("shadow_deployment")

class ShadowDeployment:
    def __init__(self, primary_model, shadow_model):
        self.primary = primary_model
        self.shadow = shadow_model

    def predict(self, features: Dict[str, Any]) -> Any:
        # Primary model serves the response
        primary_pred = self.primary.predict(features)

        # Shadow model runs but result is only logged
        try:
            shadow_pred = self.shadow.predict(features)
            logger.info(
                "shadow_comparison",
                extra={
                    "primary_pred": primary_pred,
                    "shadow_pred": shadow_pred,
                    "match": primary_pred == shadow_pred,
                },
            )
        except Exception as e:
            logger.error(f"Shadow model failed: {e}")

        return primary_pred  # Always return primary

# Usage
deployment = ShadowDeployment(model_v1, model_v2)
result = deployment.predict({"feature_a": 1.2, "feature_b": 3.4})
```

---

## 3️⃣ Canary Deployment

Route a **small percentage** of traffic to the new model while the majority still hits the old model. Gradually increase traffic if metrics are healthy.

```
                          ┌──────────┐
  ┌─── 95% traffic ─────►│ Model v1 │──► Response
  │                       └──────────┘
  │
  Request ─── Router
  │
  │                       ┌──────────┐
  └───  5% traffic ──────►│ Model v2 │──► Response
                          └──────────┘
                               │
                          Monitor KPIs
                          ──────────────
                          Latency ✓
                          Error Rate ✓
                          Accuracy ✓
```

### Characteristics

| Aspect         | Detail                                            |
|----------------|---------------------------------------------------|
| **Risk**       | 🟢 Low — only small % affected if model fails     |
| **Complexity** | 🟡 Medium — traffic routing + monitoring required  |
| **Rollback**   | Fast — route all traffic back to v1                |
| **Best For**   | Gradual rollout with real user impact validation   |

> 📌 Covered in detail in [Chapter 3 — Canary Deployments](./03-canary-deployments.md).

---

## 4️⃣ Blue-Green Deployment

Maintain **two identical production environments**. "Blue" runs the current model; "Green" runs the new model. Switch all traffic at once by flipping the load balancer.

```
                     ┌─────────────────────┐
                     │   Load Balancer      │
                     │   (traffic switch)   │
                     └────────┬────────────┘
                              │
              ┌───────────────┼───────────────┐
              │                               │
              ▼                               ▼
     ┌─────────────────┐            ┌─────────────────┐
     │  🔵 BLUE (v1)   │            │  🟢 GREEN (v2)  │
     │  ──────────     │            │  ──────────     │
     │  Current Prod   │            │  New Version    │
     │  (active)       │            │  (standby)      │
     └─────────────────┘            └─────────────────┘
```

### Characteristics

| Aspect         | Detail                                            |
|----------------|---------------------------------------------------|
| **Risk**       | 🟡 Medium — instant rollback available             |
| **Complexity** | 🟡 Medium — two full environments                  |
| **Cost**       | 🔴 High — double infrastructure                    |
| **Best For**   | Zero-downtime requirements, fast rollback needs    |

> 📌 Covered in detail in [Chapter 4 — Blue-Green Deployments](./04-blue-green-deployments.md).

---

## 5️⃣ A/B Testing Deployment

Split traffic between models **for experimentation**. Unlike canary (risk mitigation), A/B testing aims to **measure which model performs better** on a business metric.

```
                          ┌──────────┐
  ┌─── Group A (50%) ───►│ Model v1 │──► Measure KPI
  │                       └──────────┘
  │
  Request ─── Splitter
  │
  │                       ┌──────────┐
  └─── Group B (50%) ───►│ Model v2 │──► Measure KPI
                          └──────────┘
                               │
                    ┌──────────┴──────────┐
                    │ Statistical Test    │
                    │ (t-test, chi²,      │
                    │  Mann-Whitney)      │
                    └─────────────────────┘
```

### Characteristics

| Aspect         | Detail                                                  |
|----------------|---------------------------------------------------------|
| **Risk**       | 🟡 Medium — 50% traffic to unproven model               |
| **Complexity** | 🔴 High — statistical rigor + experiment tracking needed |
| **Duration**   | Days to weeks to reach statistical significance          |
| **Best For**   | Comparing models on business metrics (CTR, revenue)      |

> 📌 Covered in detail in [Chapter 2 — A/B Testing](./02-ab-testing.md).

---

## 📊 Strategy Comparison Table

| Strategy        | Risk    | Rollback Speed | Infra Cost | User Impact     | Primary Goal        |
|-----------------|---------|----------------|------------|-----------------|---------------------|
| **Direct**      | 🔴 High | ⏱ Slow         | 💰 Low     | All users       | Fast deployment     |
| **Shadow**      | 🟢 None | ⏱ N/A          | 💰💰💰 High | No users        | Offline validation  |
| **Canary**      | 🟢 Low  | ⚡ Fast         | 💰💰 Med   | Few users (1-5%)| Risk mitigation     |
| **Blue-Green**  | 🟡 Med  | ⚡ Instant      | 💰💰💰 High | All at switch   | Zero-downtime swap  |
| **A/B Testing** | 🟡 Med  | ⚡ Fast         | 💰💰 Med   | 50% of users    | Experimentation     |

---

## 🧭 Decision Flowchart

```
  Start: Deploy New Model
          │
          ▼
  ┌───────────────────────┐
  │ Is this safety-       │──── Yes ──►  Shadow Deployment
  │ critical / high-risk? │              (validate offline)
  └───────────┬───────────┘
              │ No
              ▼
  ┌───────────────────────┐
  │ Need to compare       │──── Yes ──►  A/B Testing
  │ business metrics?     │              (statistical rigor)
  └───────────┬───────────┘
              │ No
              ▼
  ┌───────────────────────┐
  │ Zero-downtime         │──── Yes ──►  Blue-Green
  │ required?             │              (instant switch)
  └───────────┬───────────┘
              │ No
              ▼
  ┌───────────────────────┐
  │ Want gradual rollout  │──── Yes ──►  Canary
  │ with monitoring?      │              (1% → 100%)
  └───────────┬───────────┘
              │ No
              ▼
        Direct Deploy
        (simple swap)
```

---

## 🔑 Key Takeaways

| # | Takeaway |
|---|----------|
| 1 | **No single strategy fits all** — choose based on risk tolerance, infrastructure budget, and business needs. |
| 2 | **Shadow deployments** are the safest but most expensive way to validate a new model. |
| 3 | **Canary and blue-green** both enable fast rollback but differ in how traffic is migrated. |
| 4 | **A/B testing** is the only strategy focused on statistical comparison of model performance. |
| 5 | **Direct deployment** should be reserved for low-risk scenarios or when speed is paramount. |

---

## ❓ Revision Questions

1. **What is the key difference between shadow deployment and canary deployment?**
   > Shadow runs the new model but never serves its predictions to users; canary serves the new model's predictions to a small subset of real users.

2. **Why is A/B testing considered higher complexity than canary deployment?**
   > A/B testing requires statistical rigor — proper sample sizing, significance testing, and experiment tracking — whereas canary only monitors operational health metrics.

3. **In what scenario would you choose blue-green over canary deployment?**
   > When you need zero-downtime switching and instant rollback for the entire user base, rather than a gradual percentage-based rollout.

4. **What is the main drawback of shadow deployment?**
   > It doubles inference costs since both models run on every request, and it cannot capture how users actually react to the new model's predictions.

5. **A team deploys a fraud-detection model. Which strategy should they start with, and why?**
   > Shadow deployment — fraud detection is safety-critical, so the new model should be validated offline against real traffic before serving any predictions.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Model Serving: Serverless Inference](../06-Model-Serving/05-serverless-inference.md) | [📂 Model Deployment](./README.md) | [A/B Testing →](./02-ab-testing.md) |

---

*Unit 7 · Chapter 1 · Deployment Strategies — MLOps Course Notes*
