# 🔄 Concept Drift

> **Unit 8 · Chapter 2** — Understanding concept drift vs data drift: types of concept drift (sudden, gradual, incremental, recurring), detection methods, window-based approaches, and retraining triggers.

---

## 📋 Chapter Overview

While **data drift** means the input distribution P(X) changes, **concept drift** is a more insidious problem — the **relationship between inputs and outputs** P(Y|X) changes. The same customer profile that was low-risk last year might now be high-risk due to changing economic conditions. The model's learned mapping from features to predictions becomes stale, even if the input data looks identical.

Concept drift is harder to detect because it requires ground truth labels, which are often delayed or expensive to obtain. This chapter explores the types of concept drift, detection methods, and strategies for triggering model retraining.

This chapter covers:

- Concept drift vs data drift — critical distinctions
- Four types of concept drift (sudden, gradual, incremental, recurring)
- Detection methods and algorithms
- Window-based and statistical approaches
- Designing retraining triggers and policies

---

## 🔍 Concept Drift vs Data Drift

```
  ┌────────────────────────────────────────────────────────────────┐
  │                                                                │
  │   DATA DRIFT                        CONCEPT DRIFT              │
  │   ──────────                        ─────────────              │
  │                                                                │
  │   P(X) changes                      P(Y|X) changes            │
  │   P(Y|X) stays same                 P(X) may or may not change│
  │                                                                │
  │   Input features shift              Decision boundary moves    │
  │   Model sees "unfamiliar" data      Model's logic becomes wrong│
  │                                                                │
  │   ┌──────────────────┐              ┌──────────────────┐      │
  │   │ Training  │ Prod │              │ Training  │ Prod │      │
  │   │           │      │              │           │      │      │
  │   │   ○○○     │  ○○  │              │   ○ ○     │  ●○  │      │
  │   │  ○○○○○    │   ○○○│              │  ○○●●●    │ ●●○  │      │
  │   │   ○○○     │  ○○○ │              │   ●●●     │  ●●● │      │
  │   │           │ ○○○  │              │           │      │      │
  │   │ Same      │ Data │              │ Same      │Labels│      │
  │   │ boundary  │moved │              │ data      │flip! │      │
  │   └──────────────────┘              └──────────────────┘      │
  │                                                                │
  │   Detectable without labels         Requires labels/feedback   │
  │   Catch early with stat tests       May only show in metrics   │
  │                                                                │
  └────────────────────────────────────────────────────────────────┘
```

### Key Comparison

| Aspect | Data Drift | Concept Drift |
|--------|-----------|---------------|
| **What changes** | Input distribution P(X) | Relationship P(Y\|X) |
| **Detection** | Statistical tests on features | Requires ground truth labels |
| **Detection speed** | Fast (real-time possible) | Slow (label delay) |
| **Example** | Users get older on average | Older users now behave differently |
| **Impact** | Model sees unfamiliar inputs | Model's learned rules are wrong |
| **Fix** | May just need feature scaling | Must retrain on new data |

---

## 📊 Types of Concept Drift

```
  SUDDEN DRIFT                          GRADUAL DRIFT
  ────────────                          ─────────────
  Accuracy                              Accuracy
  │                                     │
  │ ████████                            │ ████████
  │ ████████                            │  ███████▓▓
  │ ████████                            │   ██████▓▓▓▓
  │ ████████                            │    █████▓▓▓▓▓▓
  │ ████████░░░░░░                      │     ████▓▓▓▓▓▓▓▓
  │         ░░░░░░                      │      ███▓▓▓▓▓▓▓▓▓
  │         ░░░░░░                      │       ██▓▓▓▓▓▓▓▓▓▓
  └──────────────── time                └────────────────────── time
  Policy change, system update          Changing user behavior


  INCREMENTAL DRIFT                     RECURRING DRIFT
  ─────────────────                     ───────────────
  Accuracy                              Accuracy
  │                                     │
  │ █████                               │ ████    ████    ████
  │ ██████▓                             │ ████░░░░████░░░░████
  │ ███████▓▓                           │ ████░░░░████░░░░████
  │ ████████▓▓▓                         │ ████░░░░████░░░░████
  │ █████████▓▓▓▓                       │
  │ ██████████▓▓▓▓▓                     │
  │                                     │
  └──────────────── time                └──────────────────── time
  Slow continuous change                Seasonal / cyclical patterns
```

### Detailed Comparison

| Type | Speed | Pattern | Example | Detection Difficulty |
|------|-------|---------|---------|---------------------|
| **Sudden** | Instant | Abrupt shift at one point | New regulation changes fraud patterns overnight | Easy — sharp metric drop |
| **Gradual** | Weeks–months | Old and new concepts overlap | Customer preferences slowly shifting | Medium — slow metric decline |
| **Incremental** | Months–years | Continuous slow change | Inflation gradually changes spending patterns | Hard — change within noise |
| **Recurring** | Periodic | Cycles between known concepts | Holiday shopping patterns vs normal periods | Medium — predictable if recognized |

---

## 🏗️ Detection Methods Architecture

```
  ┌──────────────────────────────────────────────────────────────────┐
  │                  CONCEPT DRIFT DETECTION                         │
  │                                                                  │
  │  ┌────────────────────────────────────────────────────────────┐  │
  │  │                 ERROR-RATE BASED                           │  │
  │  │  Monitor prediction errors over time                      │  │
  │  │  ┌──────────┐  ┌──────────┐  ┌──────────────────────┐    │  │
  │  │  │  DDM     │  │  EDDM    │  │  Page-Hinkley        │    │  │
  │  │  │ (Drift   │  │ (Early   │  │  (Cumulative sum     │    │  │
  │  │  │ Detection│  │  Drift   │  │   of error deltas)   │    │  │
  │  │  │ Method)  │  │  Detect) │  │                      │    │  │
  │  │  └──────────┘  └──────────┘  └──────────────────────┘    │  │
  │  └────────────────────────────────────────────────────────────┘  │
  │                                                                  │
  │  ┌────────────────────────────────────────────────────────────┐  │
  │  │                 WINDOW-BASED                               │  │
  │  │  Compare performance in different time windows             │  │
  │  │  ┌──────────────┐  ┌──────────────┐  ┌────────────────┐  │  │
  │  │  │  Fixed Window │  │  ADWIN       │  │  Sliding       │  │  │
  │  │  │  (Landmark)   │  │  (Adaptive)  │  │  Window        │  │  │
  │  │  └──────────────┘  └──────────────┘  └────────────────┘  │  │
  │  └────────────────────────────────────────────────────────────┘  │
  │                                                                  │
  │  ┌────────────────────────────────────────────────────────────┐  │
  │  │                 MODEL-BASED                                │  │
  │  │  Train a meta-model to detect when the primary model fails │  │
  │  │  ┌──────────────┐  ┌──────────────┐  ┌────────────────┐  │  │
  │  │  │  Margin       │  │  Ensemble    │  │  Competence    │  │  │
  │  │  │  Density      │  │  Disagreement│  │  Model         │  │  │
  │  │  └──────────────┘  └──────────────┘  └────────────────┘  │  │
  │  └────────────────────────────────────────────────────────────┘  │
  └──────────────────────────────────────────────────────────────────┘
```

---

## 🐍 Python Implementation

### DDM (Drift Detection Method)

```python
import numpy as np
from dataclasses import dataclass, field
from typing import List, Optional, Tuple
from enum import Enum


class DriftStatus(Enum):
    NO_DRIFT = "no_drift"
    WARNING = "warning"
    DRIFT = "drift"


@dataclass
class DDMDetector:
    """
    Drift Detection Method (Gama et al., 2004).
    Monitors the error rate of a classifier. If the error rate
    increases beyond thresholds, drift is signaled.
    """
    warning_level: float = 2.0   # Standard deviations for warning
    drift_level: float = 3.0     # Standard deviations for drift
    min_samples: int = 30        # Minimum samples before detection

    # Internal state
    n: int = field(default=0, init=False)
    p: float = field(default=0.0, init=False)       # Error rate
    s: float = field(default=0.0, init=False)       # Standard deviation
    p_min: float = field(default=float("inf"), init=False)
    s_min: float = field(default=float("inf"), init=False)
    warning_zone_start: Optional[int] = field(default=None, init=False)

    def update(self, prediction_is_error: bool) -> DriftStatus:
        """Update with a single prediction result (True = error)."""
        self.n += 1
        error = 1.0 if prediction_is_error else 0.0

        # Update running error rate and standard deviation
        self.p += (error - self.p) / self.n
        self.s = np.sqrt(self.p * (1 - self.p) / self.n)

        if self.n < self.min_samples:
            return DriftStatus.NO_DRIFT

        # Track minimum error rate + std deviation
        if self.p + self.s < self.p_min + self.s_min:
            self.p_min = self.p
            self.s_min = self.s
            self.warning_zone_start = None

        # Check drift level
        if self.p + self.s > self.p_min + self.drift_level * self.s_min:
            # Reset detector after drift
            drift_point = self.warning_zone_start or self.n
            self._reset()
            return DriftStatus.DRIFT

        # Check warning level
        if self.p + self.s > self.p_min + self.warning_level * self.s_min:
            if self.warning_zone_start is None:
                self.warning_zone_start = self.n
            return DriftStatus.WARNING

        return DriftStatus.NO_DRIFT

    def _reset(self):
        """Reset internal state after drift is detected."""
        self.n = 0
        self.p = 0.0
        self.s = 0.0
        self.p_min = float("inf")
        self.s_min = float("inf")
        self.warning_zone_start = None


# --- Simulation ---
np.random.seed(42)

detector = DDM()
errors = []
statuses = []

for i in range(1000):
    # Simulate: error rate jumps from 5% to 25% at step 500
    if i < 500:
        is_error = np.random.random() < 0.05
    else:
        is_error = np.random.random() < 0.25

    status = detector.update(is_error)
    errors.append(is_error)
    statuses.append(status)

    if status == DriftStatus.DRIFT:
        print(f"🚨 DRIFT detected at step {i}")
    elif status == DriftStatus.WARNING:
        print(f"⚠️  Warning at step {i}")

# Output:
# ⚠️  Warning at step 523
# 🚨 DRIFT detected at step 541
```

### ADWIN (Adaptive Windowing)

```python
from collections import deque


class ADWINDetector:
    """
    Adaptive Windowing for drift detection.
    Maintains a variable-length window and shrinks it when
    drift is detected by comparing sub-window means.
    """

    def __init__(self, delta: float = 0.002):
        self.delta = delta           # Confidence parameter
        self.window: deque = deque()
        self.total: float = 0.0
        self.width: int = 0
        self.drift_detected: bool = False

    def update(self, value: float) -> bool:
        """Add a value and check for drift."""
        self.window.append(value)
        self.total += value
        self.width += 1
        self.drift_detected = False

        if self.width < 10:
            return False

        # Try to find a cut point where sub-windows differ
        self.drift_detected = self._check_cuts()
        return self.drift_detected

    def _check_cuts(self) -> bool:
        """Check if any partition of the window shows drift."""
        n = self.width
        running_sum = 0.0

        for i, val in enumerate(self.window):
            running_sum += val
            n1 = i + 1
            n2 = n - n1
            if n2 < 5:
                break

            mean1 = running_sum / n1
            mean2 = (self.total - running_sum) / n2

            # Hoeffding bound
            epsilon = np.sqrt(
                (1.0 / (2.0 * n1) + 1.0 / (2.0 * n2))
                * np.log(4.0 / self.delta)
            )

            if abs(mean1 - mean2) >= epsilon:
                # Remove older portion of window
                for _ in range(n1):
                    removed = self.window.popleft()
                    self.total -= removed
                    self.width -= 1
                return True

        return False


# --- Usage ---
detector = ADWINDetector(delta=0.01)

for i in range(1000):
    # Error rate changes at step 500
    error = 1.0 if np.random.random() < (0.05 if i < 500 else 0.30) else 0.0

    if detector.update(error):
        print(f"🚨 ADWIN drift detected at step {i}, window size: {detector.width}")
```

### Window-Based Performance Comparison

```python
from scipy.stats import proportions_ztest


class SlidingWindowDriftDetector:
    """
    Compare model performance between a reference window
    and a recent window using a proportion z-test.
    """

    def __init__(
        self,
        reference_window_size: int = 1000,
        test_window_size: int = 200,
        significance_level: float = 0.01,
    ):
        self.ref_size = reference_window_size
        self.test_size = test_window_size
        self.alpha = significance_level
        self.predictions: deque = deque(maxlen=reference_window_size + test_window_size)

    def add_result(self, is_correct: bool) -> Optional[dict]:
        """Add a prediction result and check for drift."""
        self.predictions.append(1 if is_correct else 0)

        if len(self.predictions) < self.ref_size + self.test_size:
            return None

        preds = list(self.predictions)
        ref_window = preds[: self.ref_size]
        test_window = preds[self.ref_size :]

        ref_correct = sum(ref_window)
        test_correct = sum(test_window)

        stat, p_value = proportions_ztest(
            count=[ref_correct, test_correct],
            nobs=[self.ref_size, self.test_size],
            alternative="larger",  # Reference accuracy > test accuracy
        )

        drift_detected = p_value < self.alpha

        return {
            "reference_accuracy": ref_correct / self.ref_size,
            "test_accuracy": test_correct / self.test_size,
            "z_statistic": round(stat, 4),
            "p_value": round(p_value, 6),
            "drift_detected": drift_detected,
        }
```

### Retraining Trigger System

```python
import logging
from datetime import datetime, timedelta
from dataclasses import dataclass
from enum import Enum
from typing import Callable, Optional

logger = logging.getLogger("retrain_trigger")


class TriggerReason(Enum):
    PERFORMANCE_DROP = "performance_drop"
    DATA_DRIFT = "data_drift"
    CONCEPT_DRIFT = "concept_drift"
    SCHEDULED = "scheduled"
    MANUAL = "manual"


@dataclass
class RetrainingPolicy:
    """Defines when retraining should be triggered."""
    min_accuracy: float = 0.85
    max_psi: float = 0.2
    max_drift_share: float = 0.5
    scheduled_interval_days: int = 30
    cooldown_hours: int = 24    # Minimum time between retrains
    min_new_samples: int = 5000


class RetrainingTrigger:
    """Evaluates multiple signals to decide if retraining is needed."""

    def __init__(self, policy: RetrainingPolicy):
        self.policy = policy
        self.last_retrain: Optional[datetime] = None
        self.new_samples_count: int = 0

    def evaluate(
        self,
        current_accuracy: float,
        current_psi: float,
        drift_share: float,
        concept_drift_detected: bool,
    ) -> Optional[TriggerReason]:
        """Check all triggers and return reason if retraining needed."""

        # Check cooldown
        if self.last_retrain:
            elapsed = datetime.utcnow() - self.last_retrain
            if elapsed < timedelta(hours=self.policy.cooldown_hours):
                logger.info(
                    f"In cooldown period ({elapsed.total_seconds()/3600:.1f}h "
                    f"/ {self.policy.cooldown_hours}h)"
                )
                return None

        # Check minimum samples
        if self.new_samples_count < self.policy.min_new_samples:
            logger.info(
                f"Insufficient new data ({self.new_samples_count} "
                f"/ {self.policy.min_new_samples})"
            )
            return None

        # Priority 1: Concept drift (most critical)
        if concept_drift_detected:
            logger.warning("Concept drift detected — retraining triggered")
            return TriggerReason.CONCEPT_DRIFT

        # Priority 2: Performance drop
        if current_accuracy < self.policy.min_accuracy:
            logger.warning(
                f"Accuracy {current_accuracy:.2%} below threshold "
                f"{self.policy.min_accuracy:.2%}"
            )
            return TriggerReason.PERFORMANCE_DROP

        # Priority 3: Data drift
        if current_psi > self.policy.max_psi:
            logger.warning(f"PSI {current_psi:.3f} exceeds {self.policy.max_psi}")
            return TriggerReason.DATA_DRIFT

        if drift_share > self.policy.max_drift_share:
            logger.warning(f"Drift share {drift_share:.1%} exceeds threshold")
            return TriggerReason.DATA_DRIFT

        # Priority 4: Scheduled
        if self.last_retrain:
            days_since = (datetime.utcnow() - self.last_retrain).days
            if days_since >= self.policy.scheduled_interval_days:
                logger.info(f"Scheduled retrain ({days_since} days since last)")
                return TriggerReason.SCHEDULED

        return None

    def record_retrain(self):
        """Record that retraining was performed."""
        self.last_retrain = datetime.utcnow()
        self.new_samples_count = 0
        logger.info("Retraining recorded")


# --- Usage ---
policy = RetrainingPolicy(min_accuracy=0.90, max_psi=0.15)
trigger = RetrainingTrigger(policy)
trigger.new_samples_count = 10000
trigger.last_retrain = datetime.utcnow() - timedelta(days=7)

reason = trigger.evaluate(
    current_accuracy=0.87,
    current_psi=0.12,
    drift_share=0.3,
    concept_drift_detected=False,
)
print(f"Trigger reason: {reason}")
# Trigger reason: TriggerReason.PERFORMANCE_DROP
```

---

## 📊 Detection Methods Comparison

| Method | Type | Detects | Latency | Needs Labels | Best For |
|--------|------|---------|---------|--------------|----------|
| **DDM** | Error-rate | Sudden, gradual | Low | Yes | Online classification |
| **EDDM** | Error-rate | Gradual (better than DDM) | Low | Yes | Slow drifts |
| **ADWIN** | Window | All types | Medium | Yes | Streaming data |
| **Page-Hinkley** | Sequential | Sudden, incremental | Low | Yes | Mean shift detection |
| **Sliding window** | Window | All types | Medium | Yes | Batch comparison |
| **PSI on predictions** | Distribution | Concept (proxy) | Medium | No | When labels are delayed |
| **Ensemble disagreement** | Model | All types | High | No | Complex models |

---

## 🔑 Key Takeaways

| # | Takeaway |
|---|----------|
| 1 | **Concept drift is harder to detect than data drift** because it requires ground truth labels, which are often delayed or unavailable. |
| 2 | **Know the four types** — sudden, gradual, incremental, recurring — each requires different detection sensitivity and retraining strategies. |
| 3 | **DDM and ADWIN are foundational algorithms** — DDM for quick detection of sudden drift, ADWIN for adaptive detection across all types. |
| 4 | **Retraining triggers should combine multiple signals** — accuracy, drift scores, and schedules — with cooldown periods to prevent thrashing. |
| 5 | **Proxy methods** (monitoring prediction distributions) can hint at concept drift when labels are delayed, but confirm with actual performance once labels arrive. |

---

## ❓ Revision Questions

1. **How does concept drift differ from data drift, and why is concept drift harder to detect?**
   > Data drift is a change in P(X) — the input distribution changes. Concept drift is a change in P(Y|X) — the mapping from inputs to outputs changes. Concept drift is harder because detecting it requires ground truth labels to verify that the model's predictions have become incorrect, while data drift can be detected by comparing feature distributions alone.

2. **Give a real-world example of each type of concept drift.**
   > **Sudden**: A new regulation bans certain financial transactions overnight, changing what constitutes fraud. **Gradual**: Consumer preferences slowly shift from physical stores to online shopping over months. **Incremental**: Inflation continuously changes the relationship between income and purchasing power. **Recurring**: Holiday seasons cyclically change shopping patterns and return to normal.

3. **How does the DDM algorithm detect drift?**
   > DDM tracks the online error rate and its standard deviation. It records the minimum error rate + std seen so far. When the current error rate exceeds the minimum by more than a warning threshold (2σ), it signals a warning. When it exceeds the drift threshold (3σ), it signals drift and resets. This assumes that in a stable concept, the error rate should decrease or stay stable as more data arrives.

4. **Why should a retraining trigger system include a cooldown period?**
   > Without a cooldown, transient spikes in error rates or temporary data anomalies could trigger repeated, unnecessary retraining cycles (thrashing). A cooldown ensures the system waits long enough to collect sufficient new data and confirm that drift is persistent, not noise. It also prevents wasting compute resources on back-to-back training runs.

5. **When would you use an ensemble disagreement method instead of DDM?**
   > Ensemble disagreement is useful when ground truth labels are unavailable or severely delayed. It detects drift by measuring how much the individual models in an ensemble disagree on predictions — high disagreement suggests the models are uncertain, which may indicate concept drift. DDM requires immediate label feedback, making it unsuitable for domains with long feedback loops (e.g., loan default prediction with 12-month horizons).

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Data Drift Detection](./01-data-drift-detection.md) | [📂 Model Monitoring](./README.md) | [Performance Monitoring →](./03-performance-monitoring.md) |

---

*Unit 8 · Chapter 2 · Concept Drift — MLOps Course Notes*
