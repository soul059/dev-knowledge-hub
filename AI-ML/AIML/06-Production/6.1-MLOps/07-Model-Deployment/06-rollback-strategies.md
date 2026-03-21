# 🔙 Rollback Strategies for ML Models

> **Unit 7 · Chapter 6** — Automated and manual rollback techniques: triggers, model registry rollback, keeping previous models ready, and recovery patterns.

---

## 📋 Chapter Overview

Even with the best testing and deployment strategies, models can fail in production. **Rollback** is the safety net — the ability to quickly revert to a known-good model when things go wrong. Unlike traditional software rollbacks, ML rollbacks must account for model artifacts, feature compatibility, and silent failures like prediction drift.

This chapter covers:

- Automated vs manual rollback triggers
- Rollback architectures and patterns
- Model registry rollback workflows
- Keeping previous models warm and ready
- Recovery playbooks and post-mortem analysis

---

## 🏗️ Rollback Architecture

```
  ┌───────────────────────────────────────────────────────────────┐
  │                    MONITORING LAYER                           │
  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────┐ │
  │  │ Latency  │  │  Error   │  │  Drift   │  │  Business    │ │
  │  │ Monitor  │  │  Rate    │  │ Detector │  │  KPI Monitor │ │
  │  └────┬─────┘  └────┬─────┘  └────┬─────┘  └──────┬───────┘ │
  └───────┼──────────────┼─────────────┼───────────────┼─────────┘
          │              │             │               │
          └──────────────┴──────┬──────┴───────────────┘
                                │
                                ▼
                   ┌────────────────────────┐
                   │   Rollback Controller  │
                   │   ──────────────────   │
                   │   Evaluate triggers    │
                   │   Decision: rollback?  │
                   └────────────┬───────────┘
                                │
                    ┌───────────┼───────────┐
                    │                       │
                    ▼                       ▼
           ┌────────────────┐      ┌────────────────┐
           │  AUTO ROLLBACK │      │ MANUAL REVIEW  │
           │  ────────────  │      │  ────────────  │
           │  Instant swap  │      │  Alert on-call │
           │  to model v(n-1)│     │  Human decides │
           └────────┬───────┘      └────────┬───────┘
                    │                       │
                    └───────────┬───────────┘
                                │
                                ▼
                   ┌────────────────────────┐
                   │    Model Registry      │
                   │    ────────────────    │
                   │    v1 ✓ (Production)   │
                   │    v2 ✗ (Rolled back)  │
                   │    v3   (Staging)      │
                   └────────────────────────┘
```

---

## ⚡ Automated vs Manual Rollback

### Decision Matrix

```
                    SEVERITY
                    │
         HIGH  ─────┤  ┌─────────────────┐   ┌─────────────────┐
                    │  │  AUTO ROLLBACK   │   │  AUTO ROLLBACK   │
                    │  │  + Page on-call  │   │  + Page on-call  │
                    │  └─────────────────┘   └─────────────────┘
                    │
         MED  ──────┤  ┌─────────────────┐   ┌─────────────────┐
                    │  │  ALERT + MANUAL  │   │  AUTO ROLLBACK   │
                    │  │  Human decides   │   │  + Alert team    │
                    │  └─────────────────┘   └─────────────────┘
                    │
         LOW  ──────┤  ┌─────────────────┐   ┌─────────────────┐
                    │  │  LOG + MONITOR   │   │  ALERT + MANUAL  │
                    │  │  Watch closely   │   │  Human decides   │
                    │  └─────────────────┘   └─────────────────┘
                    │
                    └──────────┬───────────────────┬──────────
                              LOW               HIGH
                                  CONFIDENCE
                          (that issue is model-caused)
```

### Comparison Table

| Aspect | Automated Rollback | Manual Rollback |
|--------|-------------------|-----------------|
| **Speed** | Seconds to minutes | Minutes to hours |
| **Trigger** | Threshold breach | Human judgment |
| **Risk of false alarm** | Higher | Lower |
| **Best for** | Clear failure signals (errors, crashes) | Ambiguous situations (slight drift) |
| **Requires** | Pre-defined thresholds | On-call engineer availability |
| **Coverage** | 24/7 | Business hours (unless on-call) |

---

## 🚨 Rollback Triggers

### Automated Triggers

| Trigger | Threshold | Response Time |
|---------|-----------|---------------|
| **HTTP 5xx rate** | > 5% for 5 min | Immediate rollback |
| **p99 latency** | > 2× baseline for 10 min | Immediate rollback |
| **Prediction null/NaN** | > 0.1% of requests | Immediate rollback |
| **Model crash/OOM** | Any occurrence | Immediate rollback |
| **Output distribution shift** | KL divergence > 0.5 | Rollback after confirmation |
| **Error rate spike** | > 3σ from baseline | Immediate rollback |

### Manual Triggers

| Trigger | Detection | Response |
|---------|-----------|----------|
| **Business KPI drop** | Dashboard / alert | Review → decide |
| **User complaints** | Support tickets | Investigate → decide |
| **Gradual accuracy decay** | Monitoring dashboard | Evaluate → plan |
| **Upstream data change** | Feature store alerts | Assess impact → decide |

---

## 🐍 Python Implementation

### Rollback Controller

```python
import time
import logging
from dataclasses import dataclass
from typing import Callable, Optional, Dict, Any
from enum import Enum

logger = logging.getLogger("rollback_controller")

class RollbackAction(Enum):
    NONE = "none"
    ALERT = "alert"
    ROLLBACK = "rollback"

@dataclass
class RollbackThresholds:
    max_error_rate: float = 0.05
    max_p99_latency_ms: float = 500.0
    max_null_prediction_rate: float = 0.001
    max_kl_divergence: float = 0.5
    error_rate_window_seconds: int = 300
    latency_window_seconds: int = 600

@dataclass
class ModelMetrics:
    error_rate: float
    p99_latency_ms: float
    null_prediction_rate: float
    kl_divergence: float
    requests_per_second: float

class RollbackController:
    """Monitors model health and triggers rollback when thresholds are breached."""

    def __init__(
        self,
        thresholds: Optional[RollbackThresholds] = None,
        metrics_fetcher: Optional[Callable[[], ModelMetrics]] = None,
        rollback_executor: Optional[Callable[[str], bool]] = None,
        alert_sender: Optional[Callable[[str], None]] = None,
    ):
        self.thresholds = thresholds or RollbackThresholds()
        self.fetch_metrics = metrics_fetcher
        self.execute_rollback = rollback_executor
        self.send_alert = alert_sender
        self.consecutive_failures = 0
        self.max_consecutive_before_rollback = 3

    def evaluate(self, metrics: ModelMetrics) -> tuple[RollbackAction, str]:
        """Evaluate metrics against thresholds."""
        # Critical: immediate rollback
        if metrics.error_rate > self.thresholds.max_error_rate:
            return RollbackAction.ROLLBACK, (
                f"Error rate {metrics.error_rate:.2%} exceeds "
                f"threshold {self.thresholds.max_error_rate:.2%}"
            )

        if metrics.null_prediction_rate > self.thresholds.max_null_prediction_rate:
            return RollbackAction.ROLLBACK, (
                f"Null prediction rate {metrics.null_prediction_rate:.4%} exceeds "
                f"threshold {self.thresholds.max_null_prediction_rate:.4%}"
            )

        if metrics.p99_latency_ms > self.thresholds.max_p99_latency_ms:
            return RollbackAction.ROLLBACK, (
                f"p99 latency {metrics.p99_latency_ms:.0f}ms exceeds "
                f"threshold {self.thresholds.max_p99_latency_ms:.0f}ms"
            )

        # Warning: alert but don't rollback
        if metrics.kl_divergence > self.thresholds.max_kl_divergence * 0.8:
            return RollbackAction.ALERT, (
                f"KL divergence {metrics.kl_divergence:.4f} approaching "
                f"threshold {self.thresholds.max_kl_divergence:.4f}"
            )

        return RollbackAction.NONE, "All metrics within bounds"

    def monitor_loop(self, check_interval: int = 60):
        """Continuous monitoring loop with rollback capability."""
        logger.info("Starting rollback monitor")

        while True:
            try:
                metrics = self.fetch_metrics()
                action, reason = self.evaluate(metrics)

                if action == RollbackAction.ROLLBACK:
                    self.consecutive_failures += 1
                    logger.warning(
                        f"Failure {self.consecutive_failures}/"
                        f"{self.max_consecutive_before_rollback}: {reason}"
                    )

                    if self.consecutive_failures >= self.max_consecutive_before_rollback:
                        logger.critical(f"ROLLING BACK: {reason}")
                        if self.execute_rollback:
                            self.execute_rollback(reason)
                        if self.send_alert:
                            self.send_alert(f"🚨 Model rolled back: {reason}")
                        return

                elif action == RollbackAction.ALERT:
                    if self.send_alert:
                        self.send_alert(f"⚠️ Warning: {reason}")
                    self.consecutive_failures = 0

                else:
                    self.consecutive_failures = 0
                    logger.debug(f"Health OK: {reason}")

            except Exception as e:
                logger.error(f"Monitor error: {e}")

            time.sleep(check_interval)
```

### Model Registry Rollback

```python
from dataclasses import dataclass, field
from typing import List, Optional
from datetime import datetime

@dataclass
class ModelVersion:
    version: str
    artifact_path: str
    registered_at: datetime
    stage: str  # "production", "staging", "archived", "rolled_back"
    metrics: Dict[str, float] = field(default_factory=dict)
    rollback_reason: Optional[str] = None

class ModelRegistry:
    """Simple model registry with rollback support."""

    def __init__(self):
        self.models: Dict[str, List[ModelVersion]] = {}

    def register(self, model_name: str, version: ModelVersion):
        if model_name not in self.models:
            self.models[model_name] = []
        self.models[model_name].append(version)
        logger.info(f"Registered {model_name} v{version.version}")

    def get_production_model(self, model_name: str) -> Optional[ModelVersion]:
        """Get the current production model."""
        versions = self.models.get(model_name, [])
        for v in reversed(versions):
            if v.stage == "production":
                return v
        return None

    def get_previous_production(self, model_name: str) -> Optional[ModelVersion]:
        """Get the most recent non-current production model."""
        versions = self.models.get(model_name, [])
        production_versions = [v for v in versions if v.stage in ("production", "archived")]

        if len(production_versions) < 2:
            return None
        # Sort by registration time, return second-most-recent
        sorted_versions = sorted(production_versions, key=lambda v: v.registered_at, reverse=True)
        return sorted_versions[1]

    def rollback(self, model_name: str, reason: str) -> Optional[ModelVersion]:
        """Roll back to the previous production model version."""
        current = self.get_production_model(model_name)
        previous = self.get_previous_production(model_name)

        if not current or not previous:
            logger.error(f"Cannot rollback {model_name}: no previous version found")
            return None

        # Demote current
        current.stage = "rolled_back"
        current.rollback_reason = reason
        logger.warning(
            f"Rolled back {model_name} v{current.version}: {reason}"
        )

        # Promote previous
        previous.stage = "production"
        logger.info(
            f"Restored {model_name} v{previous.version} to production"
        )

        return previous

    def rollback_to_version(
        self, model_name: str, target_version: str, reason: str
    ) -> Optional[ModelVersion]:
        """Roll back to a specific version."""
        current = self.get_production_model(model_name)
        versions = self.models.get(model_name, [])
        target = next((v for v in versions if v.version == target_version), None)

        if not target:
            logger.error(f"Version {target_version} not found")
            return None

        if current:
            current.stage = "rolled_back"
            current.rollback_reason = reason

        target.stage = "production"
        logger.info(f"Rolled back to {model_name} v{target_version}")
        return target

# ─── Usage ───────────────────────────────────────────────────

registry = ModelRegistry()

# Register model versions
registry.register("fraud_detector", ModelVersion(
    version="1.0", artifact_path="s3://models/fraud/v1.0/",
    registered_at=datetime(2024, 1, 15), stage="archived",
    metrics={"auc": 0.92, "precision": 0.88},
))
registry.register("fraud_detector", ModelVersion(
    version="2.0", artifact_path="s3://models/fraud/v2.0/",
    registered_at=datetime(2024, 3, 1), stage="production",
    metrics={"auc": 0.95, "precision": 0.91},
))

# Rollback!
restored = registry.rollback(
    "fraud_detector",
    reason="v2.0 has high false positive rate in production"
)
print(f"Restored to v{restored.version}")
```

---

## 🏥 Keeping Previous Models Ready

```
  WARM STANDBY PATTERN
  ──────────────────────

  ┌──────────────────────────────────────────────────┐
  │              Model Serving Cluster                │
  │                                                   │
  │  ┌─────────────────────┐  ┌────────────────────┐ │
  │  │  ACTIVE: Model v2   │  │  STANDBY: Model v1 │ │
  │  │  ─────────────────  │  │  ──────────────── │ │
  │  │  3 replicas         │  │  1 replica         │ │
  │  │  Serving traffic    │  │  Warm (loaded)     │ │
  │  │  Full resources     │  │  Minimal resources │ │
  │  └─────────────────────┘  └────────────────────┘ │
  │                                                   │
  │  On rollback:                                     │
  │  • Scale up v1 to 3 replicas                     │
  │  • Route traffic to v1                            │
  │  • Scale down v2                                  │
  └──────────────────────────────────────────────────┘
```

### Python — Warm Standby Manager

```python
class WarmStandbyManager:
    """Keep previous model loaded and ready for instant rollback."""

    def __init__(self, model_loader, active_version: str, standby_version: str):
        self.model_loader = model_loader
        self.active_version = active_version
        self.standby_version = standby_version
        self.models = {}

    def initialize(self):
        """Load both active and standby models."""
        self.models[self.active_version] = self.model_loader(self.active_version)
        self.models[self.standby_version] = self.model_loader(self.standby_version)
        logger.info(
            f"Active: v{self.active_version}, Standby: v{self.standby_version}"
        )

    def predict(self, features: dict) -> Any:
        """Serve prediction from active model."""
        return self.models[self.active_version].predict(features)

    def rollback(self, reason: str) -> bool:
        """Switch from active to standby — instant since model is already loaded."""
        logger.warning(f"ROLLBACK: {self.active_version} → {self.standby_version}")
        logger.warning(f"Reason: {reason}")

        # Swap active and standby
        self.active_version, self.standby_version = (
            self.standby_version,
            self.active_version,
        )

        logger.info(f"Now serving: v{self.active_version}")
        return True

    def health_check(self) -> dict:
        """Check that both models are loaded and responsive."""
        status = {}
        test_input = {"feature_a": 0.0, "feature_b": 0.0}

        for version, model in self.models.items():
            try:
                model.predict(test_input)
                status[version] = "healthy"
            except Exception as e:
                status[version] = f"unhealthy: {e}"

        return {
            "active": self.active_version,
            "standby": self.standby_version,
            "status": status,
        }
```

---

## 📋 Rollback Playbook

```
  ┌─────────────────────────────────────────────────────────┐
  │               ROLLBACK PLAYBOOK                         │
  │  ─────────────────────────────────────────────────────  │
  │                                                         │
  │  1. DETECT                                              │
  │     • Alert fires (automated) or issue reported         │
  │     • Confirm the issue is model-related                │
  │                                                         │
  │  2. DECIDE                                              │
  │     • Automated? → Rollback immediately                 │
  │     • Manual? → Assess severity + blast radius          │
  │                                                         │
  │  3. EXECUTE                                             │
  │     • Switch traffic to previous model version          │
  │     • Verify previous model is serving correctly        │
  │     • Update model registry (mark version as rolled back)│
  │                                                         │
  │  4. VERIFY                                              │
  │     • Confirm metrics return to baseline                │
  │     • Check no residual errors                          │
  │     • Monitor for 30+ minutes                           │
  │                                                         │
  │  5. COMMUNICATE                                         │
  │     • Notify stakeholders                               │
  │     • Update incident channel                           │
  │     • Document timeline                                 │
  │                                                         │
  │  6. POST-MORTEM                                         │
  │     • Root cause analysis                               │
  │     • What tests missed this?                           │
  │     • Action items to prevent recurrence                │
  └─────────────────────────────────────────────────────────┘
```

---

## 📊 Rollback Strategy Comparison

| Strategy | Speed | Complexity | Data Loss Risk | Best For |
|----------|-------|-----------|----------------|----------|
| **Warm standby** | ⚡ Instant | Medium | None | High-traffic, low-latency services |
| **Registry rollback** | ⏱ Minutes | Low | None | Standard ML pipelines |
| **Blue-green switch** | ⚡ Instant | Medium | None | Services with dual environments |
| **Feature flag toggle** | ⚡ Instant | Low | None | Application-level model switching |
| **Redeployment** | ⏱ 10-30 min | Low | Possible | Simple systems, last resort |

---

## 🔧 MLflow Registry Rollback Example

```python
import mlflow
from mlflow.tracking import MlflowClient

def rollback_mlflow_model(model_name: str, reason: str):
    """Roll back model in MLflow registry to previous production version."""
    client = MlflowClient()

    # Get current production version
    prod_versions = client.get_latest_versions(model_name, stages=["Production"])
    if not prod_versions:
        raise ValueError(f"No production version found for {model_name}")

    current = prod_versions[0]
    current_version = int(current.version)

    # Find previous version
    all_versions = client.search_model_versions(f"name='{model_name}'")
    previous_versions = [
        v for v in all_versions
        if int(v.version) < current_version
    ]

    if not previous_versions:
        raise ValueError("No previous version available for rollback")

    previous = max(previous_versions, key=lambda v: int(v.version))

    # Demote current
    client.transition_model_version_stage(
        name=model_name,
        version=current.version,
        stage="Archived",
        archive_existing_versions=False,
    )
    client.set_model_version_tag(
        model_name, current.version, "rollback_reason", reason
    )

    # Promote previous
    client.transition_model_version_stage(
        name=model_name,
        version=previous.version,
        stage="Production",
    )

    print(f"Rolled back {model_name}: v{current.version} → v{previous.version}")
    print(f"Reason: {reason}")
    return previous

# Usage
rollback_mlflow_model(
    "fraud-detector",
    reason="v5 has 3x higher false positive rate than v4"
)
```

---

## 🔑 Key Takeaways

| # | Takeaway |
|---|----------|
| 1 | **Define rollback triggers before deployment**, not during an incident — panic decisions are poor decisions. |
| 2 | **Keep the previous model warm** — cold-starting a model during an incident adds critical minutes of downtime. |
| 3 | **Automate rollback for clear failures** (errors, crashes, latency spikes) — save human judgment for ambiguous cases. |
| 4 | **Model registry is your source of truth** — always update it during rollback to maintain audit trail. |
| 5 | **Conduct a post-mortem** after every rollback to improve testing and deployment processes. |

---

## ❓ Revision Questions

1. **Why should the previous model version be kept warm in production?**
   > Loading an ML model from cold storage can take seconds to minutes (downloading artifacts, loading weights into GPU memory, warming caches). During an incident, this delay extends the impact on users. A warm standby model can serve traffic immediately.

2. **What is the difference between automated and manual rollback triggers?**
   > Automated triggers fire on clear, measurable thresholds (error rates, latency) and execute rollback without human intervention. Manual triggers require human judgment for ambiguous signals (slight accuracy drops, user complaints) where the cause may not be the model.

3. **How does a model registry support rollback?**
   > The registry maintains versioned model artifacts with stage labels (production, staging, archived). Rollback involves transitioning the current version to "archived" and the previous version back to "production," providing a clear audit trail.

4. **When should you use redeployment as a rollback strategy instead of warm standby?**
   > When infrastructure constraints prevent running two models simultaneously, or for simple systems where the model loads quickly and a few minutes of degraded service is acceptable.

5. **What should a post-mortem after a rollback include?**
   > Timeline of events, root cause analysis, why existing tests didn't catch the issue, metrics during the incident, rollback execution details, and action items to prevent recurrence (e.g., new monitoring alerts, additional validation steps).

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Feature Flags](./05-feature-flags.md) | [📂 Model Deployment](./README.md) | [Model Monitoring: Data Drift Detection →](../08-Model-Monitoring/01-data-drift-detection.md) |

---

*Unit 7 · Chapter 6 · Rollback Strategies — MLOps Course Notes*
