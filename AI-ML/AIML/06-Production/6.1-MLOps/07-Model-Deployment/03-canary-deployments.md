# 🐤 Canary Deployments for ML Models

> **Unit 7 · Chapter 3** — Gradual rollout strategy: route a small percentage of traffic to the new model, monitor, and progressively increase.

---

## 📋 Chapter Overview

A **canary deployment** takes its name from the "canary in a coal mine" — a small, early signal that warns of danger before it becomes catastrophic. In ML, this means sending a small fraction of production traffic (e.g., 1%) to a new model, monitoring its behavior, and only scaling up if all metrics are healthy.

This chapter covers:

- Canary rollout phases (1% → 10% → 50% → 100%)
- Monitoring criteria during each phase
- Automated promotion and rollback triggers
- Infrastructure architecture for canary routing

---

## 🏗️ Canary Architecture

```
                    ┌─────────────────────────────────────────┐
                    │            API Gateway / LB             │
                    │  ┌───────────────────────────────────┐  │
                    │  │       Traffic Router               │  │
                    │  │  ┌─────────┐    ┌──────────────┐  │  │
                    │  │  │ Weight  │    │  Routing     │  │  │
                    │  │  │ Config  │───►│  Rules       │  │  │
                    │  │  │ (99/1)  │    │              │  │  │
                    │  │  └─────────┘    └──────┬───────┘  │  │
                    │  └───────────────────────┬┼──────────┘  │
                    └──────────────────────────┼┼─────────────┘
                                               ││
                              ┌────────────────┘└────────────────┐
                              │                                  │
                              ▼                                  ▼
                    ┌──────────────────┐              ┌──────────────────┐
                    │   Stable (v1)    │              │   Canary (v2)    │
                    │   ────────────   │              │   ────────────   │
                    │   99% traffic    │              │    1% traffic    │
                    │   3 replicas     │              │    1 replica     │
                    └────────┬─────────┘              └────────┬─────────┘
                             │                                 │
                             └─────────────┬───────────────────┘
                                           │
                                           ▼
                              ┌──────────────────────┐
                              │   Metrics Collector  │
                              │   ────────────────   │
                              │   Prometheus / DD    │
                              └──────────┬───────────┘
                                         │
                                         ▼
                              ┌──────────────────────┐
                              │   Canary Analyzer    │
                              │   ────────────────   │
                              │   Compare v1 vs v2   │
                              │   Auto promote/      │
                              │   rollback            │
                              └──────────────────────┘
```

---

## 📈 Rollout Phases

```
  Phase 1        Phase 2        Phase 3        Phase 4        Phase 5
  ─────────      ─────────      ─────────      ─────────      ─────────
  1% canary      5% canary      10% canary     50% canary     100% (done)
  30 min         1 hour         2 hours        6 hours        Full deploy

  ┌──────────┐  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
  │██████████│  │██████████│   │█████████ │   │█████     │   │          │
  │██████████│  │█████████ │   │████████  │   │          │   │          │
  │██████████│  │█████████ │   │████████  │   │          │   │          │
  │██████████│  │█████████ │   │████████  │   │          │   │          │
  │          │  │          │   │          │   │█████     │   │██████████│
  │  v1:99%  │  │  v1:95%  │   │  v1:90%  │   │  v1:50%  │   │ v2:100%  │
  │  v2: 1%  │  │  v2: 5%  │   │  v2:10%  │   │  v2:50%  │   │ (stable) │
  └──────────┘  └──────────┘   └──────────┘   └──────────┘   └──────────┘
       │              │              │              │              │
       ▼              ▼              ▼              ▼              ▼
   Monitor ✓      Monitor ✓     Monitor ✓     Monitor ✓       Done ✓
```

---

## 🔍 Monitoring Criteria

### Metrics to Track at Each Phase

| Metric Category | Specific Metrics | Threshold |
|-----------------|-----------------|-----------|
| **Latency** | p50, p95, p99 response time | < 20% degradation vs v1 |
| **Error Rate** | HTTP 5xx, prediction errors | < 1% absolute increase |
| **Model Quality** | Prediction distribution shift | KL divergence < 0.1 |
| **Business KPIs** | CTR, conversion, revenue | No significant drop |
| **Infrastructure** | CPU, memory, GPU utilization | Within capacity limits |

### Rollback Triggers

```
  Automatic Rollback if ANY condition is met:
  ─────────────────────────────────────────────

  ┌─────────────────────────────────────────────────┐
  │  ❌  Error rate > 5%                            │
  │  ❌  p99 latency > 2× baseline                  │
  │  ❌  Prediction NaN/Null rate > 0.1%            │
  │  ❌  Memory usage > 90% of limit                │
  │  ❌  Model output distribution KL div > 0.5     │
  └─────────────────────────────────────────────────┘
           │
           ▼
  ┌─────────────────────────────────────────────────┐
  │  🔙  Route 100% traffic back to v1              │
  │  📢  Alert on-call engineer                     │
  │  📝  Log rollback reason + metrics snapshot     │
  └─────────────────────────────────────────────────┘
```

---

## 🐍 Python Implementation

### Canary Controller

```python
import time
import logging
from dataclasses import dataclass, field
from enum import Enum
from typing import Dict, Optional, Callable

logger = logging.getLogger("canary_controller")

class CanaryPhase(Enum):
    PHASE_1 = ("1%", 0.01, 1800)    # 30 minutes
    PHASE_2 = ("5%", 0.05, 3600)    # 1 hour
    PHASE_3 = ("10%", 0.10, 7200)   # 2 hours
    PHASE_4 = ("50%", 0.50, 21600)  # 6 hours
    PHASE_5 = ("100%", 1.00, 0)     # Full deploy

    def __init__(self, label, weight, duration_seconds):
        self.label = label
        self.weight = weight
        self.duration_seconds = duration_seconds

@dataclass
class CanaryMetrics:
    error_rate: float = 0.0
    p99_latency_ms: float = 0.0
    prediction_null_rate: float = 0.0
    memory_usage_pct: float = 0.0
    kl_divergence: float = 0.0

@dataclass
class CanaryThresholds:
    max_error_rate: float = 0.05
    max_p99_latency_ms: float = 500.0
    max_null_rate: float = 0.001
    max_memory_pct: float = 90.0
    max_kl_divergence: float = 0.5

class CanaryController:
    """Manages gradual canary rollout with automated promotion and rollback."""

    def __init__(
        self,
        thresholds: Optional[CanaryThresholds] = None,
        metrics_fetcher: Optional[Callable[[], CanaryMetrics]] = None,
    ):
        self.thresholds = thresholds or CanaryThresholds()
        self.fetch_metrics = metrics_fetcher
        self.phases = list(CanaryPhase)
        self.current_phase_idx = 0
        self.is_rolled_back = False

    @property
    def current_phase(self) -> CanaryPhase:
        return self.phases[self.current_phase_idx]

    def check_health(self, metrics: CanaryMetrics) -> tuple[bool, str]:
        """Evaluate canary health against thresholds."""
        checks = [
            (metrics.error_rate <= self.thresholds.max_error_rate,
             f"Error rate {metrics.error_rate:.2%} > {self.thresholds.max_error_rate:.2%}"),
            (metrics.p99_latency_ms <= self.thresholds.max_p99_latency_ms,
             f"p99 latency {metrics.p99_latency_ms}ms > {self.thresholds.max_p99_latency_ms}ms"),
            (metrics.prediction_null_rate <= self.thresholds.max_null_rate,
             f"Null rate {metrics.prediction_null_rate:.4%} > {self.thresholds.max_null_rate:.4%}"),
            (metrics.memory_usage_pct <= self.thresholds.max_memory_pct,
             f"Memory {metrics.memory_usage_pct:.1f}% > {self.thresholds.max_memory_pct:.1f}%"),
            (metrics.kl_divergence <= self.thresholds.max_kl_divergence,
             f"KL divergence {metrics.kl_divergence:.4f} > {self.thresholds.max_kl_divergence:.4f}"),
        ]

        for passed, msg in checks:
            if not passed:
                return False, msg
        return True, "All checks passed"

    def promote(self) -> bool:
        """Advance to the next canary phase."""
        if self.current_phase_idx >= len(self.phases) - 1:
            logger.info("Canary already at 100% — deployment complete")
            return False

        self.current_phase_idx += 1
        phase = self.current_phase
        logger.info(f"Promoted to {phase.label} (weight={phase.weight})")
        return True

    def rollback(self, reason: str):
        """Roll back canary: route all traffic to stable version."""
        self.is_rolled_back = True
        self.current_phase_idx = 0
        logger.warning(f"ROLLBACK triggered: {reason}")

    def run_rollout(self):
        """Execute the full canary rollout with monitoring at each phase."""
        logger.info("Starting canary rollout")

        for phase in self.phases:
            self.current_phase_idx = self.phases.index(phase)
            logger.info(f"Phase: {phase.label} traffic to canary")

            if phase.duration_seconds == 0:
                logger.info("Full deployment reached — canary complete!")
                break

            # Monitor during phase duration
            check_interval = min(300, phase.duration_seconds // 3)
            elapsed = 0

            while elapsed < phase.duration_seconds:
                time.sleep(check_interval)
                elapsed += check_interval

                if self.fetch_metrics:
                    metrics = self.fetch_metrics()
                    healthy, msg = self.check_health(metrics)

                    if not healthy:
                        self.rollback(msg)
                        return False

                    logger.info(f"Health check OK at {elapsed}s — {msg}")

            # Promote to next phase
            logger.info(f"Phase {phase.label} complete — promoting")

        return True

# ─── Usage Example ───────────────────────────────────────────

def fetch_live_metrics() -> CanaryMetrics:
    """Fetch real metrics from monitoring system."""
    # In production: query Prometheus, Datadog, etc.
    return CanaryMetrics(
        error_rate=0.002,
        p99_latency_ms=145.0,
        prediction_null_rate=0.0001,
        memory_usage_pct=62.0,
        kl_divergence=0.03,
    )

controller = CanaryController(
    thresholds=CanaryThresholds(max_p99_latency_ms=300.0),
    metrics_fetcher=fetch_live_metrics,
)

success = controller.run_rollout()
print(f"Canary rollout {'succeeded' if success else 'rolled back'}")
```

---

## 🔧 Canary with Kubernetes (Istio)

```yaml
# VirtualService for Istio canary routing
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: ml-model-service
spec:
  hosts:
    - ml-model.example.com
  http:
    - route:
        - destination:
            host: ml-model
            subset: stable
          weight: 95
        - destination:
            host: ml-model
            subset: canary
          weight: 5
---
# DestinationRule defining subsets
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: ml-model
spec:
  host: ml-model
  subsets:
    - name: stable
      labels:
        version: v1
    - name: canary
      labels:
        version: v2
```

---

## 📊 Canary vs Shadow vs A/B

| Aspect | Canary | Shadow | A/B Testing |
|--------|--------|--------|-------------|
| **User sees new model?** | ✅ Yes (small %) | ❌ No | ✅ Yes (50%) |
| **Primary goal** | Risk mitigation | Offline validation | Experimentation |
| **Traffic to new model** | 1% → 100% | 100% (parallel) | 50% (fixed) |
| **Rollback trigger** | Operational metrics | N/A | Statistical analysis |
| **Duration** | Hours to days | Days to weeks | Days to weeks |
| **Infrastructure cost** | Medium | High (2× inference) | Medium |

---

## ⚡ Automated Canary with Flagger

```
  ┌──────────────────────────────────────────────┐
  │              Flagger Controller               │
  │  ────────────────────────────────────────     │
  │  1. Detect new deployment                     │
  │  2. Create canary service                     │
  │  3. Gradually shift traffic                   │
  │  4. Run webhook tests (optional)              │
  │  5. Query Prometheus for success rate          │
  │  6. Promote or rollback                       │
  └──────────────────────┬─────────────────────────┘
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
     ┌─────────┐   ┌─────────┐   ┌──────────┐
     │Promote  │   │Rollback │   │ Alert    │
     │to 100%  │   │to 0%    │   │ On-Call  │
     └─────────┘   └─────────┘   └──────────┘
```

---

## 🔑 Key Takeaways

| # | Takeaway |
|---|----------|
| 1 | Start with **1% traffic** — enough to detect issues, small enough to limit blast radius. |
| 2 | **Automate health checks** — human monitoring doesn't scale for progressive rollouts. |
| 3 | Define **rollback triggers before deployment**, not during an incident. |
| 4 | Use **service mesh** (Istio, Linkerd) for fine-grained traffic splitting in Kubernetes. |
| 5 | Canary is for **risk mitigation**, not experimentation — use A/B testing for that. |

---

## ❓ Revision Questions

1. **Why start a canary rollout at 1% and not 10%?**
   > 1% limits the blast radius — if the model has a critical bug, only 1% of users are affected. This provides enough traffic to detect obvious issues (errors, latency spikes) while minimizing risk.

2. **What metrics should you monitor during a canary rollout?**
   > Error rates, latency (p50/p95/p99), prediction null/NaN rates, model output distribution shift (KL divergence), and infrastructure metrics (CPU, memory, GPU utilization).

3. **How does a service mesh like Istio enable canary deployments?**
   > Istio's VirtualService resource allows weighted traffic splitting between service subsets (stable vs canary) at the network layer, without application code changes.

4. **When should a canary be automatically rolled back?**
   > When any pre-defined threshold is breached: error rate exceeds limit, latency spikes beyond acceptable range, prediction quality degrades, or infrastructure resources are exhausted.

5. **What is the difference between canary deployment and feature flags?**
   > Canary operates at the infrastructure/routing level — traffic is split before reaching the application. Feature flags operate at the application level — the code decides which model to use based on flag configuration.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← A/B Testing](./02-ab-testing.md) | [📂 Model Deployment](./README.md) | [Blue-Green Deployments →](./04-blue-green-deployments.md) |

---

*Unit 7 · Chapter 3 · Canary Deployments — MLOps Course Notes*
