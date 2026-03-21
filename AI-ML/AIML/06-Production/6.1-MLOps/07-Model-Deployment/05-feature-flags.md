# 🚩 Feature Flags for ML Models

> **Unit 7 · Chapter 5** — Toggle ML models on/off in production using feature flag systems for controlled, code-level rollout and experimentation.

---

## 📋 Chapter Overview

**Feature flags** (also called feature toggles) allow you to enable or disable functionality at runtime **without redeploying code**. For ML systems, this means toggling between models, gradually rolling out a new model to specific user segments, or instantly killing a misbehaving model — all through configuration, not deployments.

This chapter covers:

- Feature flags vs deployment strategies
- Types of feature flags for ML
- Integration with LaunchDarkly and Unleash
- Gradual rollout with percentage-based flags
- Python implementation patterns

---

## 🏗️ Feature Flag Architecture for ML

```
                     ┌──────────────────────────────┐
                     │     Feature Flag Service      │
                     │  ┌────────────────────────┐  │
                     │  │  Flag: "model_v2"      │  │
                     │  │  ──────────────────     │  │
                     │  │  Enabled: true          │  │
                     │  │  Rollout: 25%           │  │
                     │  │  Segments: ["premium"]  │  │
                     │  └────────────┬───────────┘  │
                     └───────────────┼──────────────┘
                                     │
                          SDK polls / streams
                                     │
                                     ▼
        ┌──────────────────────────────────────────────────┐
        │              ML Application Server               │
        │  ┌──────────────────────────────────────────┐    │
        │  │  if flag_client.is_enabled("model_v2"):  │    │
        │  │      prediction = model_v2.predict(x)    │    │
        │  │  else:                                    │    │
        │  │      prediction = model_v1.predict(x)    │    │
        │  └──────────────────────────────────────────┘    │
        │                                                   │
        │  ┌─────────────────┐   ┌─────────────────┐       │
        │  │   Model v1      │   │   Model v2      │       │
        │  │   (default)     │   │   (flagged)     │       │
        │  └─────────────────┘   └─────────────────┘       │
        └──────────────────────────────────────────────────┘
```

---

## 🏷️ Types of Feature Flags for ML

| Flag Type | Description | ML Use Case |
|-----------|-------------|-------------|
| **Release Flag** | Toggle new feature on/off | Enable new recommendation model |
| **Experiment Flag** | Split traffic for A/B testing | Compare model v1 vs v2 on business KPIs |
| **Ops Flag** | Kill switch for emergencies | Disable misbehaving model instantly |
| **Permission Flag** | Enable for specific users/segments | New model for premium users only |
| **Percentage Flag** | Gradual rollout by percentage | 5% → 25% → 50% → 100% |

### Feature Flags vs Infrastructure Strategies

```
  Feature Flags                    Infrastructure Strategies
  (Application Level)              (Network/Routing Level)
  ─────────────────                ─────────────────────────

  ┌─────────────────┐              ┌─────────────────┐
  │  App Code        │              │  Load Balancer  │
  │  ┌─────────────┐│              │  ┌─────────────┐│
  │  │ if flag:    ││              │  │ weight: 95  ││
  │  │   model_v2  ││              │  │ weight:  5  ││
  │  │ else:       ││              │  └─────────────┘│
  │  │   model_v1  ││              └────────┬────────┘
  │  └─────────────┘│                       │
  └─────────────────┘              ┌────────┼────────┐
                                   ▼                 ▼
  • Decision in code              │ Model v1 │   │ Model v2 │
  • Fine-grained targeting        └──────────┘   └──────────┘
  • No infra changes needed
  • Can combine user attributes    • Decision at network layer
                                   • Requires routing infra
                                   • Simpler application code
```

---

## 🐍 Python Implementation — Custom Feature Flag Client

```python
import hashlib
import time
import logging
from dataclasses import dataclass, field
from typing import Any, Dict, List, Optional
from enum import Enum

logger = logging.getLogger("feature_flags")

@dataclass
class FeatureFlag:
    name: str
    enabled: bool = False
    rollout_percentage: float = 0.0
    allowed_segments: List[str] = field(default_factory=list)
    metadata: Dict[str, Any] = field(default_factory=dict)

class FeatureFlagClient:
    """Lightweight feature flag client for ML model toggling."""

    def __init__(self):
        self.flags: Dict[str, FeatureFlag] = {}

    def register_flag(self, flag: FeatureFlag):
        self.flags[flag.name] = flag
        logger.info(f"Registered flag: {flag.name} (enabled={flag.enabled})")

    def is_enabled(
        self,
        flag_name: str,
        user_id: Optional[str] = None,
        user_segment: Optional[str] = None,
    ) -> bool:
        """Check if a feature flag is enabled for a given user."""
        flag = self.flags.get(flag_name)
        if not flag or not flag.enabled:
            return False

        # Segment-based targeting
        if flag.allowed_segments and user_segment:
            if user_segment not in flag.allowed_segments:
                return False

        # Percentage-based rollout (consistent per user)
        if flag.rollout_percentage < 100.0 and user_id:
            hash_val = int(
                hashlib.md5(f"{flag_name}:{user_id}".encode()).hexdigest(), 16
            )
            bucket = (hash_val % 10000) / 100.0
            return bucket < flag.rollout_percentage

        return True

    def get_variant(
        self,
        flag_name: str,
        user_id: str,
        variants: List[str],
    ) -> str:
        """Get a specific variant for multi-variant flags."""
        hash_val = int(
            hashlib.md5(f"{flag_name}:{user_id}".encode()).hexdigest(), 16
        )
        idx = hash_val % len(variants)
        return variants[idx]

# ─── Usage: ML Model Toggle ─────────────────────────────────

flag_client = FeatureFlagClient()

# Register a gradual rollout flag
flag_client.register_flag(FeatureFlag(
    name="recommendation_model_v2",
    enabled=True,
    rollout_percentage=25.0,  # 25% of users
    allowed_segments=["premium", "beta"],
))

# Register a kill switch
flag_client.register_flag(FeatureFlag(
    name="fraud_model_v3",
    enabled=False,  # Disabled — killed due to production issue
))

def get_prediction(user_id: str, features: dict, segment: str = "default"):
    """Route prediction to the appropriate model based on feature flags."""
    if flag_client.is_enabled("recommendation_model_v2", user_id, segment):
        return model_v2.predict(features)
    return model_v1.predict(features)

# 25% of premium users get model_v2
print(get_prediction("user_001", {"item": "laptop"}, "premium"))
print(get_prediction("user_002", {"item": "phone"}, "free"))  # Always v1
```

---

## 🔗 Integration with LaunchDarkly

```python
import ldclient
from ldclient import Context
from ldclient.config import Config

# Initialize LaunchDarkly client
ldclient.set_config(Config("sdk-key-your-key-here"))
client = ldclient.get()

def get_model_prediction(user_id: str, features: dict) -> Any:
    """Use LaunchDarkly to decide which model to use."""
    # Build user context
    context = Context.builder(user_id).set("plan", "premium").build()

    # Check which model variant to use
    model_variant = client.variation(
        "ml-model-variant",   # Flag key in LaunchDarkly
        context,
        "v1",                 # Default value if flag not found
    )

    models = {
        "v1": model_v1,
        "v2": model_v2,
        "v3_experimental": model_v3,
    }

    model = models.get(model_variant, model_v1)
    prediction = model.predict(features)

    # Track the event for experimentation
    client.track("prediction_made", context, data={
        "model_variant": model_variant,
        "prediction": prediction,
    })

    return prediction
```

---

## 🔗 Integration with Unleash

```python
from UnleashClient import UnleashClient

# Initialize Unleash client
unleash = UnleashClient(
    url="http://unleash.example.com/api",
    app_name="ml-prediction-service",
    custom_headers={"Authorization": "your-api-token"},
)
unleash.initialize_client()

def predict_with_flag(user_id: str, features: dict) -> dict:
    """Use Unleash feature flags for model routing."""
    context = {"userId": user_id}

    if unleash.is_enabled("new-churn-model", context):
        model = churn_model_v2
        variant = "v2"
    else:
        model = churn_model_v1
        variant = "v1"

    prediction = model.predict(features)

    return {
        "prediction": prediction,
        "model_variant": variant,
        "flag_enabled": unleash.is_enabled("new-churn-model", context),
    }
```

---

## 📈 Gradual Rollout Pattern

```
  Day 1          Day 3          Day 7          Day 14         Day 21
  ─────          ─────          ─────          ──────         ──────
  1% rollout     5% rollout     25% rollout    50% rollout    100% rollout

  ┌──────────┐  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
  │██████████│  │██████████│   │████████  │   │█████     │   │          │
  │██████████│  │█████████ │   │██████    │   │          │   │          │
  │  v1:99%  │  │  v1:95%  │   │  v1:75%  │   │  v1:50%  │   │ v2:100%  │
  │  v2: 1%  │  │  v2: 5%  │   │  v2:25%  │   │  v2:50%  │   │ (stable) │
  └──────────┘  └──────────┘   └──────────┘   └──────────┘   └──────────┘
       │              │              │              │              │
   Monitor ✓      Monitor ✓     Monitor ✓     Monitor ✓     Remove flag
```

### Python — Automated Gradual Rollout

```python
import time
import logging

logger = logging.getLogger("gradual_rollout")

class GradualRolloutManager:
    """Manage gradual percentage-based rollout via feature flags."""

    ROLLOUT_STAGES = [
        {"percentage": 1, "duration_hours": 24, "label": "Smoke test"},
        {"percentage": 5, "duration_hours": 48, "label": "Early validation"},
        {"percentage": 25, "duration_hours": 72, "label": "Quarter rollout"},
        {"percentage": 50, "duration_hours": 168, "label": "Half rollout"},
        {"percentage": 100, "duration_hours": 0, "label": "Full rollout"},
    ]

    def __init__(self, flag_client: FeatureFlagClient, flag_name: str):
        self.client = flag_client
        self.flag_name = flag_name

    def update_rollout_percentage(self, percentage: float):
        """Update the rollout percentage for the flag."""
        flag = self.client.flags.get(self.flag_name)
        if flag:
            flag.rollout_percentage = percentage
            logger.info(f"Updated {self.flag_name} rollout to {percentage}%")

    def check_metrics_healthy(self) -> bool:
        """Check if model metrics are within acceptable bounds."""
        # In production: query monitoring system
        # Return False to trigger rollback
        return True

    def execute_rollout(self) -> bool:
        """Execute the full gradual rollout plan."""
        for stage in self.ROLLOUT_STAGES:
            pct = stage["percentage"]
            label = stage["label"]
            hours = stage["duration_hours"]

            logger.info(f"Stage: {label} — setting rollout to {pct}%")
            self.update_rollout_percentage(pct)

            if hours == 0:
                logger.info("Full rollout reached!")
                return True

            # Monitor during stage
            checks = max(1, hours // 4)
            for i in range(checks):
                time.sleep(hours * 3600 / checks)
                if not self.check_metrics_healthy():
                    logger.error(f"Metrics unhealthy at {pct}% — rolling back to 0%")
                    self.update_rollout_percentage(0)
                    return False
                logger.info(f"Health check {i+1}/{checks} passed at {pct}%")

        return True

# Usage
manager = GradualRolloutManager(flag_client, "recommendation_model_v2")
success = manager.execute_rollout()
```

---

## ⚡ Kill Switch Pattern

```python
class MLKillSwitch:
    """Emergency kill switch for ML models using feature flags."""

    def __init__(self, flag_client: FeatureFlagClient):
        self.client = flag_client

    def kill(self, flag_name: str, reason: str):
        """Immediately disable a model."""
        flag = self.client.flags.get(flag_name)
        if flag:
            flag.enabled = False
            flag.metadata["kill_reason"] = reason
            flag.metadata["killed_at"] = time.time()
            logger.critical(f"KILL SWITCH: {flag_name} disabled — {reason}")

    def revive(self, flag_name: str, rollout_percentage: float = 1.0):
        """Re-enable a killed model at low rollout."""
        flag = self.client.flags.get(flag_name)
        if flag:
            flag.enabled = True
            flag.rollout_percentage = rollout_percentage
            flag.metadata.pop("kill_reason", None)
            logger.info(f"REVIVED: {flag_name} at {rollout_percentage}%")

# Emergency: model producing bad predictions
kill_switch = MLKillSwitch(flag_client)
kill_switch.kill("fraud_model_v3", "High false positive rate detected")

# After fix, cautiously re-enable
kill_switch.revive("fraud_model_v3", rollout_percentage=1.0)
```

---

## 📊 Feature Flag Platforms Comparison

| Feature | LaunchDarkly | Unleash | Flagsmith | Custom |
|---------|-------------|---------|-----------|--------|
| **Hosting** | SaaS | Self-hosted / SaaS | Both | Self-hosted |
| **Pricing** | $$$  | Free (OSS) | Free tier | Free |
| **Python SDK** | ✅ Official | ✅ Official | ✅ Official | Build your own |
| **Percentage rollout** | ✅ | ✅ | ✅ | ✅ |
| **User targeting** | ✅ Advanced | ✅ Basic | ✅ | Manual |
| **A/B experiments** | ✅ Built-in | 🟡 Via variants | ✅ | Manual |
| **Audit log** | ✅ | ✅ | ✅ | Manual |
| **Real-time updates** | ✅ Streaming | ✅ Polling | ✅ | Manual |
| **Best for** | Enterprise | OSS-first teams | Mid-size | Simple needs |

---

## 🔑 Key Takeaways

| # | Takeaway |
|---|----------|
| 1 | Feature flags give **application-level control** over model routing — no infrastructure changes needed. |
| 2 | Always implement a **kill switch** — the ability to instantly disable a model is critical for production safety. |
| 3 | Use **consistent hashing** on user ID so the same user always gets the same model variant. |
| 4 | **Clean up stale flags** — remove flags for models that are fully rolled out to avoid code complexity. |
| 5 | Feature flags and canary deployments are **complementary** — flags control code paths, canary controls traffic routing. |

---

## ❓ Revision Questions

1. **What is the key difference between a feature flag and a canary deployment?**
   > Feature flags operate at the application/code level — the application decides which model to call. Canary deployments operate at the infrastructure/routing level — the load balancer decides which service receives traffic.

2. **Why is a kill switch important for ML models in production?**
   > ML models can fail silently (e.g., drift, bad predictions) without throwing errors. A kill switch lets you instantly revert to the safe model without waiting for a redeployment.

3. **How does percentage-based rollout work with feature flags?**
   > The user ID is hashed into a bucket (0-100). If the bucket number is below the rollout percentage, the user gets the new model. Consistent hashing ensures the same user always falls into the same bucket.

4. **When should you remove a feature flag?**
   > After the new model is fully rolled out to 100%, validated, and stable for a sufficient observation period (e.g., 2-4 weeks). Stale flags increase code complexity and technical debt.

5. **How would you combine feature flags with A/B testing?**
   > Use a multi-variant feature flag where each variant maps to a different model. The flag service assigns users to variants deterministically, and you track business metrics per variant to determine the winner.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Blue-Green Deployments](./04-blue-green-deployments.md) | [📂 Model Deployment](./README.md) | [Rollback Strategies →](./06-rollback-strategies.md) |

---

*Unit 7 · Chapter 5 · Feature Flags — MLOps Course Notes*
