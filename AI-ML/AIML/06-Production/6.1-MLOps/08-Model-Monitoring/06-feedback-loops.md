# 🔁 Feedback Loops

> **Unit 8 · Chapter 6** — Closing the ML loop: collecting ground truth labels, human-in-the-loop systems, active learning, data flywheel effects, and automated retraining pipelines.

---

## 📋 Chapter Overview

An ML model in production is not a finished product — it's a **living system** that must continuously learn and improve. The **feedback loop** is the mechanism that connects production predictions back to model improvement: collecting ground truth labels, identifying where the model struggles, and using that information to retrain better models.

Without feedback loops, models decay silently. With well-designed feedback loops, models **get better over time** — creating a **data flywheel** where more users generate more data, which trains better models, which attract more users. This virtuous cycle is the foundation of competitive advantage in ML-powered products.

This chapter covers:

- Collecting ground truth labels in production
- Human-in-the-loop (HITL) workflows
- Active learning for efficient labeling
- Automated retraining pipelines
- The data flywheel concept and network effects

---

## 🏗️ The ML Feedback Loop

```
  ┌──────────────────────────────────────────────────────────────────┐
  │                   THE ML FEEDBACK LOOP                          │
  │                                                                  │
  │          ┌──────────────┐                                       │
  │          │  TRAIN MODEL │◄─────────────────────────┐            │
  │          │  on labeled  │                          │            │
  │          │  data        │                          │            │
  │          └──────┬───────┘                          │            │
  │                 │                                  │            │
  │                 ▼                                  │            │
  │          ┌──────────────┐                          │            │
  │          │   DEPLOY     │                   ┌──────┴──────┐    │
  │          │   to prod    │                   │  RETRAIN    │    │
  │          └──────┬───────┘                   │  with new   │    │
  │                 │                            │  labeled    │    │
  │                 ▼                            │  data       │    │
  │          ┌──────────────┐                   └──────┬──────┘    │
  │          │   PREDICT    │                          │            │
  │          │   on live    │                          │            │
  │          │   data       │                          │            │
  │          └──────┬───────┘                          │            │
  │                 │                                  │            │
  │                 ▼                                  │            │
  │          ┌──────────────┐          ┌──────────────┐            │
  │          │  COLLECT     │          │   LABEL      │            │
  │          │  feedback /  │─────────▶│  ground truth│            │
  │          │  outcomes    │          │  (human/auto)│────────────┘
  │          └──────────────┘          └──────────────┘
  │                                                                  │
  │    Time to close loop:                                          │
  │    • Fraud detection: 30-90 days (chargeback)                   │
  │    • Recommendation: seconds (click/no-click)                   │
  │    • Medical diagnosis: months-years (follow-up)                │
  │    • Ad click prediction: seconds (immediate feedback)          │
  └──────────────────────────────────────────────────────────────────┘
```

---

## 📊 Ground Truth Collection Methods

```
  ┌────────────────────────────────────────────────────────────────┐
  │           GROUND TRUTH COLLECTION STRATEGIES                   │
  │                                                                │
  │  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────┐  │
  │  │  DIRECT FEEDBACK │  │  IMPLICIT        │  │  DELAYED     │  │
  │  │  ───────────────│  │  FEEDBACK        │  │  FEEDBACK    │  │
  │  │                  │  │  ────────────── │  │  ──────────  │  │
  │  │  User explicitly │  │  Inferred from  │  │  Outcome     │  │
  │  │  provides label  │  │  user behavior  │  │  observed    │  │
  │  │                  │  │                  │  │  later       │  │
  │  │  • 👍/👎 buttons │  │  • Click-through │  │  • Chargeback│  │
  │  │  • Star ratings  │  │  • Dwell time    │  │  • Loan      │  │
  │  │  • "Report wrong"│  │  • Purchase      │  │    default   │  │
  │  │  • Manual review │  │  • Skip/scroll   │  │  • Patient   │  │
  │  │                  │  │  • Bounce rate   │  │    outcome   │  │
  │  │                  │  │                  │  │              │  │
  │  │  Latency: instant│  │  Latency: sec-min│  │  Latency:    │  │
  │  │  Quality: high   │  │  Quality: noisy  │  │   days-years │  │
  │  │  Volume: low     │  │  Volume: high    │  │  Quality: high│  │
  │  └─────────────────┘  └─────────────────┘  └──────────────┘  │
  └────────────────────────────────────────────────────────────────┘
```

### Comparison Table

| Method | Feedback Latency | Label Quality | Volume | Cost | Example |
|--------|-----------------|---------------|--------|------|---------|
| **Direct user feedback** | Instant | High | Low (< 5% respond) | Low | Thumbs up/down on recommendation |
| **Implicit behavior** | Seconds–minutes | Noisy | Very high | Low | Click-through rate, dwell time |
| **Expert review** | Hours–days | Very high | Very low | High | Radiologist reviewing AI scan |
| **Delayed outcome** | Days–years | High | High (eventually) | Low | Loan default after 12 months |
| **Crowdsourced labeling** | Hours–days | Medium | Configurable | Medium | Amazon Mechanical Turk labels |
| **Active learning** | Hours | High | Targeted | Medium | Model selects uncertain samples for labeling |

---

## 👤 Human-in-the-Loop (HITL) Systems

```
  ┌──────────────────────────────────────────────────────────────────┐
  │               HUMAN-IN-THE-LOOP ARCHITECTURE                    │
  │                                                                  │
  │  ┌──────────────┐                                               │
  │  │   Request    │                                               │
  │  └──────┬───────┘                                               │
  │         │                                                        │
  │         ▼                                                        │
  │  ┌──────────────┐      HIGH CONFIDENCE                         │
  │  │  ML Model    │─────────────────────────▶ Automated response  │
  │  │  Prediction  │      (prob > 0.95)                            │
  │  └──────┬───────┘                                               │
  │         │                                                        │
  │         │ LOW/MEDIUM CONFIDENCE                                  │
  │         │ (0.4 < prob < 0.95)                                    │
  │         ▼                                                        │
  │  ┌──────────────────┐                                           │
  │  │  ROUTING LOGIC   │                                           │
  │  │  ──────────────  │                                           │
  │  │  Route by:       │                                           │
  │  │  • Confidence    │                                           │
  │  │  • Risk level    │                                           │
  │  │  • Case type     │                                           │
  │  └──────┬───────────┘                                           │
  │         │                                                        │
  │         ▼                                                        │
  │  ┌──────────────────┐     ┌──────────────────┐                 │
  │  │  HUMAN REVIEWER  │────▶│  Decision logged │                 │
  │  │  ──────────────  │     │  as ground truth  │                 │
  │  │  • Expert queue  │     │  ────────────── │                 │
  │  │  • Annotation UI │     │  Feeds back to   │                 │
  │  │  • SLA: < 1 hour │     │  training data   │                 │
  │  └──────────────────┘     └──────────────────┘                 │
  │                                                                  │
  │  VERY LOW CONFIDENCE (prob < 0.4):                              │
  │  → Flag for expert review + don't show automated prediction     │
  └──────────────────────────────────────────────────────────────────┘
```

---

## 🐍 Python Implementation

### Ground Truth Collector

```python
import logging
from dataclasses import dataclass, field
from datetime import datetime, timezone
from typing import Any, Dict, List, Optional
from enum import Enum
from collections import defaultdict

logger = logging.getLogger("feedback_loop")


class FeedbackType(Enum):
    DIRECT = "direct"          # User explicitly labeled
    IMPLICIT = "implicit"      # Inferred from behavior
    DELAYED = "delayed"        # Outcome observed later
    EXPERT = "expert"          # Expert/human review
    ACTIVE = "active_learning" # Model-requested label


@dataclass
class FeedbackRecord:
    prediction_id: str
    model_name: str
    model_version: str
    prediction: Any
    ground_truth: Any
    feedback_type: FeedbackType
    timestamp: str = field(
        default_factory=lambda: datetime.now(timezone.utc).isoformat()
    )
    confidence: Optional[float] = None
    metadata: Dict[str, Any] = field(default_factory=dict)

    @property
    def is_correct(self) -> bool:
        return self.prediction == self.ground_truth


class GroundTruthCollector:
    """
    Collects ground truth labels from multiple sources
    and matches them with predictions for retraining.
    """

    def __init__(self, storage_backend=None):
        self.storage = storage_backend
        self.pending_predictions: Dict[str, dict] = {}
        self.feedback_records: List[FeedbackRecord] = []
        self.stats = defaultdict(int)

    def register_prediction(
        self,
        prediction_id: str,
        model_name: str,
        model_version: str,
        prediction: Any,
        features: dict,
        confidence: float,
    ):
        """Register a prediction that awaits ground truth."""
        self.pending_predictions[prediction_id] = {
            "model_name": model_name,
            "model_version": model_version,
            "prediction": prediction,
            "features": features,
            "confidence": confidence,
            "timestamp": datetime.now(timezone.utc).isoformat(),
        }

    def record_feedback(
        self,
        prediction_id: str,
        ground_truth: Any,
        feedback_type: FeedbackType,
        metadata: Optional[dict] = None,
    ) -> Optional[FeedbackRecord]:
        """Match ground truth with a registered prediction."""
        if prediction_id not in self.pending_predictions:
            logger.warning(
                f"No prediction found for ID {prediction_id}"
            )
            return None

        pred_info = self.pending_predictions.pop(prediction_id)

        record = FeedbackRecord(
            prediction_id=prediction_id,
            model_name=pred_info["model_name"],
            model_version=pred_info["model_version"],
            prediction=pred_info["prediction"],
            ground_truth=ground_truth,
            feedback_type=feedback_type,
            confidence=pred_info["confidence"],
            metadata=metadata or {},
        )

        self.feedback_records.append(record)
        self.stats["total"] += 1
        self.stats["correct" if record.is_correct else "incorrect"] += 1
        self.stats[feedback_type.value] += 1

        logger.info(
            f"Feedback recorded: pred={record.prediction}, "
            f"truth={ground_truth}, correct={record.is_correct}"
        )

        return record

    def get_accuracy(self) -> float:
        """Calculate accuracy from collected feedback."""
        if not self.feedback_records:
            return 0.0
        correct = sum(1 for r in self.feedback_records if r.is_correct)
        return correct / len(self.feedback_records)

    def get_training_dataset(self) -> List[dict]:
        """Export feedback as training data for retraining."""
        if not self.pending_predictions:
            # All predictions have been matched
            pass

        training_data = []
        for record in self.feedback_records:
            pred_id = record.prediction_id
            # In production, features would be retrieved from feature store
            training_data.append({
                "prediction_id": pred_id,
                "ground_truth": record.ground_truth,
                "feedback_type": record.feedback_type.value,
                "timestamp": record.timestamp,
            })

        return training_data

    def summary(self) -> dict:
        return {
            "total_feedback": self.stats["total"],
            "accuracy": round(self.get_accuracy(), 4),
            "pending_predictions": len(self.pending_predictions),
            "by_type": {
                ft.value: self.stats[ft.value]
                for ft in FeedbackType
            },
        }


# --- Usage ---
collector = GroundTruthCollector()

# Register predictions as they're made
collector.register_prediction(
    prediction_id="pred-001",
    model_name="fraud_detector",
    model_version="v3",
    prediction=0,  # Not fraud
    features={"amount": 150, "merchant": "grocery"},
    confidence=0.92,
)

# Later, when ground truth arrives
collector.record_feedback(
    prediction_id="pred-001",
    ground_truth=0,  # Confirmed: not fraud
    feedback_type=FeedbackType.DELAYED,
)

print(collector.summary())
```

### Active Learning Selector

```python
import numpy as np
from typing import List, Tuple
from dataclasses import dataclass


@dataclass
class UnlabeledSample:
    sample_id: str
    features: list
    model_prediction: Any
    model_confidence: float


class ActiveLearningSelector:
    """
    Selects the most informative samples for human labeling
    to maximize model improvement per label.
    """

    def __init__(self, strategy: str = "uncertainty"):
        self.strategy = strategy

    def select(
        self,
        samples: List[UnlabeledSample],
        budget: int,
    ) -> List[UnlabeledSample]:
        """Select the top-N most informative samples."""
        if self.strategy == "uncertainty":
            return self._uncertainty_sampling(samples, budget)
        elif self.strategy == "margin":
            return self._margin_sampling(samples, budget)
        elif self.strategy == "entropy":
            return self._entropy_sampling(samples, budget)
        elif self.strategy == "random":
            return self._random_sampling(samples, budget)
        else:
            raise ValueError(f"Unknown strategy: {self.strategy}")

    def _uncertainty_sampling(
        self, samples: List[UnlabeledSample], budget: int,
    ) -> List[UnlabeledSample]:
        """Select samples with lowest prediction confidence."""
        sorted_samples = sorted(
            samples, key=lambda s: s.model_confidence
        )
        return sorted_samples[:budget]

    def _margin_sampling(
        self, samples: List[UnlabeledSample], budget: int,
    ) -> List[UnlabeledSample]:
        """
        Select samples where the margin between top-2
        class probabilities is smallest.
        """
        # For binary classification, margin = |p - 0.5| * 2
        def margin_score(s):
            return abs(s.model_confidence - 0.5)

        sorted_samples = sorted(samples, key=margin_score)
        return sorted_samples[:budget]

    def _entropy_sampling(
        self, samples: List[UnlabeledSample], budget: int,
    ) -> List[UnlabeledSample]:
        """Select samples with highest prediction entropy."""
        def entropy(s):
            p = s.model_confidence
            if p <= 0 or p >= 1:
                return 0
            return -(p * np.log2(p) + (1 - p) * np.log2(1 - p))

        sorted_samples = sorted(
            samples, key=entropy, reverse=True
        )
        return sorted_samples[:budget]

    def _random_sampling(
        self, samples: List[UnlabeledSample], budget: int,
    ) -> List[UnlabeledSample]:
        """Random baseline for comparison."""
        indices = np.random.choice(
            len(samples), size=min(budget, len(samples)), replace=False
        )
        return [samples[i] for i in indices]


# --- Usage ---
np.random.seed(42)

# Simulate unlabeled production data
unlabeled = [
    UnlabeledSample(
        sample_id=f"sample-{i}",
        features=[np.random.randn() for _ in range(5)],
        model_prediction=int(np.random.random() > 0.5),
        model_confidence=np.random.random(),
    )
    for i in range(1000)
]

# Select 50 most informative samples for labeling
selector = ActiveLearningSelector(strategy="uncertainty")
selected = selector.select(unlabeled, budget=50)

print(f"Selected {len(selected)} samples for labeling")
print(f"Confidence range: {selected[0].model_confidence:.3f} - "
      f"{selected[-1].model_confidence:.3f}")
# The model is least confident about these — labeling them
# provides maximum information for retraining.
```

### Automated Retraining Pipeline

```python
import logging
from datetime import datetime, timedelta
from typing import Optional, Callable
from enum import Enum

logger = logging.getLogger("retrain_pipeline")


class RetrainStatus(Enum):
    PENDING = "pending"
    RUNNING = "running"
    VALIDATING = "validating"
    DEPLOYING = "deploying"
    COMPLETED = "completed"
    FAILED = "failed"
    ROLLED_BACK = "rolled_back"


class AutoRetrainingPipeline:
    """
    End-to-end automated retraining pipeline that closes
    the feedback loop from production data to new model.
    """

    def __init__(
        self,
        feedback_collector: GroundTruthCollector,
        trainer: Callable,          # train(data) -> model
        evaluator: Callable,        # evaluate(model, data) -> metrics
        deployer: Callable,         # deploy(model) -> bool
        min_samples: int = 5000,
        min_accuracy_improvement: float = 0.01,
    ):
        self.collector = feedback_collector
        self.trainer = trainer
        self.evaluator = evaluator
        self.deployer = deployer
        self.min_samples = min_samples
        self.min_improvement = min_accuracy_improvement
        self.current_model = None
        self.current_accuracy = 0.0
        self.status = RetrainStatus.PENDING
        self.history = []

    def should_retrain(self) -> tuple[bool, str]:
        """Check if conditions are met for retraining."""
        feedback_count = len(self.collector.feedback_records)

        if feedback_count < self.min_samples:
            return False, (
                f"Insufficient feedback: {feedback_count}/{self.min_samples}"
            )

        current_accuracy = self.collector.get_accuracy()
        if current_accuracy < self.current_accuracy - 0.05:
            return True, (
                f"Accuracy degraded: {current_accuracy:.2%} "
                f"(was {self.current_accuracy:.2%})"
            )

        return True, f"Sufficient data collected: {feedback_count} samples"

    def run(self) -> dict:
        """Execute the full retraining pipeline."""
        should, reason = self.should_retrain()
        if not should:
            logger.info(f"Skipping retrain: {reason}")
            return {"status": "skipped", "reason": reason}

        run_id = datetime.utcnow().strftime("%Y%m%d_%H%M%S")
        logger.info(f"Starting retrain run {run_id}: {reason}")

        try:
            # Step 1: Prepare training data
            self.status = RetrainStatus.RUNNING
            training_data = self.collector.get_training_dataset()
            logger.info(f"Training data: {len(training_data)} samples")

            # Step 2: Train new model
            new_model = self.trainer(training_data)
            logger.info("Model training completed")

            # Step 3: Evaluate
            self.status = RetrainStatus.VALIDATING
            holdout = training_data[int(len(training_data) * 0.8) :]
            new_metrics = self.evaluator(new_model, holdout)
            new_accuracy = new_metrics.get("accuracy", 0)
            logger.info(
                f"New model accuracy: {new_accuracy:.2%} "
                f"(current: {self.current_accuracy:.2%})"
            )

            # Step 4: Compare with current model
            improvement = new_accuracy - self.current_accuracy
            if improvement < self.min_improvement:
                logger.warning(
                    f"Insufficient improvement: {improvement:.2%} "
                    f"(need {self.min_improvement:.2%})"
                )
                self.status = RetrainStatus.COMPLETED
                return {
                    "status": "skipped_no_improvement",
                    "current_accuracy": self.current_accuracy,
                    "new_accuracy": new_accuracy,
                }

            # Step 5: Deploy
            self.status = RetrainStatus.DEPLOYING
            deployed = self.deployer(new_model)

            if deployed:
                self.current_model = new_model
                self.current_accuracy = new_accuracy
                self.status = RetrainStatus.COMPLETED
                logger.info(
                    f"✅ New model deployed (accuracy: {new_accuracy:.2%})"
                )
            else:
                self.status = RetrainStatus.FAILED
                logger.error("Deployment failed")

            result = {
                "status": self.status.value,
                "run_id": run_id,
                "previous_accuracy": self.current_accuracy,
                "new_accuracy": new_accuracy,
                "improvement": improvement,
                "training_samples": len(training_data),
            }

            self.history.append(result)
            return result

        except Exception as e:
            self.status = RetrainStatus.FAILED
            logger.error(f"Retrain failed: {e}", exc_info=True)
            return {"status": "failed", "error": str(e)}
```

---

## 🔄 The Data Flywheel

```
  ┌──────────────────────────────────────────────────────────────────┐
  │                    THE DATA FLYWHEEL                              │
  │                                                                  │
  │         ┌───────────────────┐                                   │
  │         │   MORE USERS      │                                   │
  │         │   use the product │                                   │
  │         └────────┬──────────┘                                   │
  │                  │                                               │
  │                  ▼                                               │
  │         ┌───────────────────┐                                   │
  │         │   MORE DATA       │                                   │
  │         │   generated       │                                   │
  │         └────────┬──────────┘                                   │
  │                  │                           ┌──────────────┐   │
  │                  ▼                           │  FLYWHEEL    │   │
  │         ┌───────────────────┐               │  EFFECT:     │   │
  │         │  BETTER MODELS    │               │              │   │
  │         │  trained on more  │               │  Each turn   │   │
  │         │  diverse data     │               │  makes the   │   │
  │         └────────┬──────────┘               │  next turn   │   │
  │                  │                           │  easier and  │   │
  │                  ▼                           │  faster      │   │
  │         ┌───────────────────┐               │              │   │
  │         │  BETTER PRODUCT   │               │  Competitive │   │
  │         │  experience       │──────────┐    │  moat grows  │   │
  │         └───────────────────┘          │    │  over time   │   │
  │                                        │    └──────────────┘   │
  │                  ┌─────────────────────┘                       │
  │                  ▼                                               │
  │         ┌───────────────────┐                                   │
  │         │   MORE USERS      │  ◄── cycle repeats               │
  │         └───────────────────┘                                   │
  │                                                                  │
  │  Examples:                                                      │
  │  • Google Search: more queries → better ranking → more users    │
  │  • Netflix: more views → better recommendations → more subs     │
  │  • Tesla: more miles → better autopilot → more buyers           │
  │  • Spotify: more listens → better discovery → more listeners    │
  └──────────────────────────────────────────────────────────────────┘
```

### Flywheel vs Non-Flywheel Systems

| Aspect | Without Flywheel | With Flywheel |
|--------|-----------------|---------------|
| **Model over time** | Decays (drift) | Improves continuously |
| **Competitive moat** | Weak (code can be copied) | Strong (data advantage) |
| **Labeling cost** | Constant or increasing | Decreasing (model assists) |
| **User experience** | Stagnant | Continuously improving |
| **Time to value** | One-time | Compounding |

---

## 📊 Feedback Loop Strategies by Domain

| Domain | Feedback Signal | Latency | Strategy |
|--------|----------------|---------|----------|
| **Search ranking** | Click, dwell time | Seconds | Implicit feedback, online learning |
| **Fraud detection** | Chargeback, investigation | 30–90 days | Delayed labels, rule-based interim |
| **Content moderation** | User reports, expert review | Hours–days | HITL, active learning on edge cases |
| **Medical imaging** | Pathologist confirmation | Days–months | Expert review, HITL mandatory |
| **Ad click prediction** | Click/no-click | Seconds | Full automation, real-time updates |
| **Recommendation** | Purchase, rating, skip | Seconds–days | Multi-signal implicit feedback |
| **Autonomous driving** | Disengagement, accident | Instant–months | Simulation + delayed outcome |
| **NLP / chatbot** | User satisfaction, task completion | Minutes | Direct feedback + implicit signals |

---

## 🔑 Key Takeaways

| # | Takeaway |
|---|----------|
| 1 | **Feedback loops are what separate one-time ML projects from ML products** — without them, models inevitably degrade after deployment. |
| 2 | **Ground truth collection is the bottleneck** — the method depends on your domain: implicit feedback is cheap but noisy, expert labels are accurate but expensive. |
| 3 | **Active learning maximizes ROI on labeling** — by selectively labeling the most uncertain predictions, you get more model improvement per label dollar. |
| 4 | **The data flywheel creates compounding competitive advantage** — more users → more data → better models → better product → more users. |
| 5 | **Automated retraining pipelines must include validation gates** — never deploy a new model without verifying it outperforms the current one on a holdout set. |

---

## ❓ Revision Questions

1. **What is the difference between direct, implicit, and delayed feedback in ML systems?**
   > **Direct feedback** is when users explicitly provide labels (e.g., thumbs up/down). **Implicit feedback** is inferred from user behavior without explicit labeling (e.g., click-through rate, dwell time). **Delayed feedback** is when the true outcome is only observed after a significant time lag (e.g., loan defaults after 12 months). Each has different trade-offs in volume, quality, and timeliness.

2. **How does active learning reduce labeling costs, and when is it most effective?**
   > Active learning selects the most informative samples for labeling — typically those where the model is most uncertain (near the decision boundary). Instead of randomly labeling thousands of samples, you label hundreds of carefully selected ones and achieve similar model improvement. It's most effective when labeling is expensive (expert review) and there's a large pool of unlabeled data.

3. **What is the data flywheel, and why does it create a competitive moat?**
   > The data flywheel is a virtuous cycle: more users generate more data, which trains better models, which improve the product, which attracts more users. It creates a competitive moat because competitors can copy code and algorithms, but they can't replicate the accumulated data and user feedback. The advantage compounds over time — each cycle makes the model harder to catch up with.

4. **Why must an automated retraining pipeline include validation gates?**
   > Without validation gates, a model retrained on noisy or biased feedback data could be worse than the current model. Validation gates compare the new model against the incumbent on a holdout set, ensuring accuracy actually improves. They also check for regressions on specific segments (e.g., the new model might be better overall but worse for a critical customer group). Deploying a worse model breaks user trust and erodes the flywheel.

5. **How would you design a feedback loop for a fraud detection model where ground truth takes 30-90 days?**
   > (1) Use a delayed feedback collector that matches predictions with chargeback outcomes 30-90 days later. (2) In the interim, supplement with rule-based labels (known fraud patterns) and investigator decisions. (3) Implement a HITL workflow where the model flags uncertain cases for human investigation, and the investigator's decision serves as a proxy label. (4) Use active learning to prioritize the most uncertain cases for investigation. (5) Schedule retraining monthly once sufficient delayed labels accumulate, always validating against recent holdout data before deployment.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Logging and Debugging](./05-logging-and-debugging.md) | [📂 Model Monitoring](./README.md) | [Cloud Platforms for ML →](../09-Infrastructure/01-cloud-platforms-for-ml.md) |

---

*Unit 8 · Chapter 6 · Feedback Loops — MLOps Course Notes*
