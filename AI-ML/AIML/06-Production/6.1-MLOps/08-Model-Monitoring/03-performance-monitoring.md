# ⚡ Performance Monitoring

> **Unit 8 · Chapter 3** — Tracking model accuracy, latency, and throughput in production: metrics dashboards, Prometheus/Grafana setup, alerting thresholds, and SLA management.

---

## 📋 Chapter Overview

A model that passes all offline tests can still fail in production. **Performance monitoring** closes the gap between offline evaluation and real-world behavior by continuously tracking model quality, serving speed, and system health. Without it, you're flying blind — unable to distinguish between a model that's working perfectly and one that's silently making bad predictions.

Production performance monitoring operates on three axes: **model quality** (are predictions correct?), **serving latency** (are predictions fast enough?), and **throughput** (can the system handle the load?). Together, they define your model's Service Level Objectives (SLOs).

This chapter covers:

- The three pillars of ML performance monitoring
- Key metrics to track and how to compute them
- Prometheus + Grafana setup for ML systems
- Setting effective alerting thresholds
- Dashboard design for ML teams

---

## 🏗️ Three Pillars of ML Performance Monitoring

```
  ┌──────────────────────────────────────────────────────────────────┐
  │              ML PERFORMANCE MONITORING                           │
  │                                                                  │
  │    ┌──────────────────┐  ┌──────────────┐  ┌────────────────┐   │
  │    │  MODEL QUALITY   │  │   LATENCY    │  │  THROUGHPUT    │   │
  │    │  ──────────────  │  │  ──────────  │  │  ────────────  │   │
  │    │                  │  │              │  │                │   │
  │    │  • Accuracy      │  │  • p50       │  │  • QPS         │   │
  │    │  • Precision     │  │  • p95       │  │  • Batch size  │   │
  │    │  • Recall / F1   │  │  • p99       │  │  • Queue depth │   │
  │    │  • AUC-ROC       │  │  • Timeout % │  │  • Saturation  │   │
  │    │  • RMSE / MAE    │  │              │  │                │   │
  │    │                  │  │              │  │                │   │
  │    │  "Is the model   │  │  "Is it fast │  │  "Can it       │   │
  │    │   correct?"      │  │   enough?"   │  │   handle load?"│   │
  │    └──────────────────┘  └──────────────┘  └────────────────┘   │
  │                                                                  │
  │    ┌─────────────────────────────────────────────────────────┐   │
  │    │                SYSTEM HEALTH                            │   │
  │    │  CPU / GPU utilization, memory, disk I/O, error rates  │   │
  │    └─────────────────────────────────────────────────────────┘   │
  └──────────────────────────────────────────────────────────────────┘
```

### Metrics by Model Type

| Metric | Classification | Regression | Ranking | Generative |
|--------|---------------|------------|---------|------------|
| **Primary quality** | Accuracy, F1 | RMSE, MAE | NDCG, MRR | BLEU, perplexity |
| **Latency target** | < 50ms | < 50ms | < 100ms | < 2s (streaming) |
| **Throughput** | 1000+ QPS | 1000+ QPS | 500+ QPS | 10-100 QPS |
| **Key alert** | Accuracy < threshold | RMSE spike | NDCG drop | Toxicity spike |

---

## 📈 Metrics Deep Dive

### Latency Distribution

```
  Requests
  │
  │ ██
  │ ████
  │ ██████
  │ ████████
  │ ██████████                      Latency Percentiles:
  │ ████████████                    ──────────────────
  │ ██████████████                  p50  =  12ms  (median)
  │ ████████████████                p90  =  45ms
  │ ██████████████████              p95  =  89ms
  │ ██████████████████████          p99  = 234ms  ⚠️ SLO: <200ms
  │ ████████████████████████████    p999 = 512ms
  └──────────────────────────────── Latency (ms)
    0   20   50   100  200  500

  ⚠️  Mean latency (28ms) looks fine, but p99 violates SLO!
     Always monitor tail latencies, not just averages.
```

### Quality Metrics Over Time

```
  ┌─────────────────────────────────────────────────────────────┐
  │  Model Accuracy (7-day rolling)                             │
  │                                                             │
  │  1.0 ┤                                                      │
  │      │                                                      │
  │  0.95┤ ────────────────────────────── SLO: 0.93             │
  │      │ ╭───────────╮                                        │
  │  0.90┤─╯           ╰──────────╮                             │
  │      │                        ╰──────╮                      │
  │  0.85┤                               ╰───── ⚠️ Below SLO   │
  │      │                                                      │
  │  0.80┤                                                      │
  │      └──────────────────────────────────────── time          │
  │        Mon  Tue  Wed  Thu  Fri  Sat  Sun                    │
  └─────────────────────────────────────────────────────────────┘
```

---

## 🐍 Python Implementation

### Prometheus Metrics for ML Serving

```python
from prometheus_client import (
    Counter,
    Histogram,
    Gauge,
    Summary,
    start_http_server,
    CollectorRegistry,
)
import time
import numpy as np
from functools import wraps

# --- Define ML-specific Prometheus metrics ---

REGISTRY = CollectorRegistry()

# Prediction latency
PREDICTION_LATENCY = Histogram(
    "ml_prediction_latency_seconds",
    "Time spent processing prediction requests",
    labelnames=["model_name", "model_version"],
    buckets=[0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1.0, 2.5],
    registry=REGISTRY,
)

# Prediction counter
PREDICTION_TOTAL = Counter(
    "ml_predictions_total",
    "Total number of predictions served",
    labelnames=["model_name", "model_version", "status"],
    registry=REGISTRY,
)

# Prediction value distribution
PREDICTION_VALUE = Histogram(
    "ml_prediction_value",
    "Distribution of prediction values (for regression / probability)",
    labelnames=["model_name"],
    buckets=[0.0, 0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 1.0],
    registry=REGISTRY,
)

# Feature value gauges (for drift monitoring)
FEATURE_VALUE = Summary(
    "ml_feature_value",
    "Summary statistics of input feature values",
    labelnames=["model_name", "feature_name"],
    registry=REGISTRY,
)

# Model accuracy (updated periodically)
MODEL_ACCURACY = Gauge(
    "ml_model_accuracy",
    "Current model accuracy (rolling window)",
    labelnames=["model_name", "model_version"],
    registry=REGISTRY,
)

# Active model info
MODEL_INFO = Gauge(
    "ml_model_info",
    "Currently loaded model metadata",
    labelnames=["model_name", "model_version", "framework"],
    registry=REGISTRY,
)


def track_prediction(model_name: str, model_version: str):
    """Decorator to track prediction metrics."""
    def decorator(predict_fn):
        @wraps(predict_fn)
        def wrapper(*args, **kwargs):
            start_time = time.time()
            try:
                result = predict_fn(*args, **kwargs)
                duration = time.time() - start_time

                PREDICTION_LATENCY.labels(
                    model_name=model_name,
                    model_version=model_version,
                ).observe(duration)

                PREDICTION_TOTAL.labels(
                    model_name=model_name,
                    model_version=model_version,
                    status="success",
                ).inc()

                # Track prediction value distribution
                if isinstance(result, (int, float)):
                    PREDICTION_VALUE.labels(
                        model_name=model_name,
                    ).observe(result)

                return result

            except Exception as e:
                PREDICTION_TOTAL.labels(
                    model_name=model_name,
                    model_version=model_version,
                    status="error",
                ).inc()
                raise

        return wrapper
    return decorator


# --- Example ML service with metrics ---

class MonitoredMLService:
    """ML service with built-in Prometheus monitoring."""

    def __init__(self, model, model_name: str, model_version: str):
        self.model = model
        self.model_name = model_name
        self.model_version = model_version

        MODEL_INFO.labels(
            model_name=model_name,
            model_version=model_version,
            framework="sklearn",
        ).set(1)

    @track_prediction("fraud_detector", "v3")
    def predict(self, features: dict) -> float:
        """Make a prediction with full monitoring."""
        # Track input feature distributions
        for name, value in features.items():
            if isinstance(value, (int, float)):
                FEATURE_VALUE.labels(
                    model_name=self.model_name,
                    feature_name=name,
                ).observe(value)

        return self.model.predict([list(features.values())])[0]


# Start Prometheus metrics server
# start_http_server(8000, registry=REGISTRY)
# print("Metrics server running on :8000/metrics")
```

### Grafana Dashboard Configuration (JSON)

```python
GRAFANA_DASHBOARD = {
    "title": "ML Model Performance",
    "panels": [
        {
            "title": "Prediction Latency (p50 / p95 / p99)",
            "type": "timeseries",
            "targets": [
                {
                    "expr": 'histogram_quantile(0.50, rate(ml_prediction_latency_seconds_bucket[5m]))',
                    "legendFormat": "p50",
                },
                {
                    "expr": 'histogram_quantile(0.95, rate(ml_prediction_latency_seconds_bucket[5m]))',
                    "legendFormat": "p95",
                },
                {
                    "expr": 'histogram_quantile(0.99, rate(ml_prediction_latency_seconds_bucket[5m]))',
                    "legendFormat": "p99",
                },
            ],
        },
        {
            "title": "Predictions per Second",
            "type": "timeseries",
            "targets": [
                {
                    "expr": 'rate(ml_predictions_total{status="success"}[1m])',
                    "legendFormat": "Success QPS",
                },
                {
                    "expr": 'rate(ml_predictions_total{status="error"}[1m])',
                    "legendFormat": "Error QPS",
                },
            ],
        },
        {
            "title": "Model Accuracy (Rolling)",
            "type": "gauge",
            "targets": [
                {
                    "expr": "ml_model_accuracy",
                    "legendFormat": "{{model_name}} {{model_version}}",
                },
            ],
            "thresholds": [
                {"value": 0, "color": "red"},
                {"value": 0.85, "color": "yellow"},
                {"value": 0.93, "color": "green"},
            ],
        },
        {
            "title": "Error Rate",
            "type": "timeseries",
            "targets": [
                {
                    "expr": (
                        'rate(ml_predictions_total{status="error"}[5m]) / '
                        'rate(ml_predictions_total[5m])'
                    ),
                    "legendFormat": "Error Rate",
                },
            ],
        },
        {
            "title": "Prediction Value Distribution",
            "type": "heatmap",
            "targets": [
                {
                    "expr": "rate(ml_prediction_value_bucket[5m])",
                    "legendFormat": "{{le}}",
                },
            ],
        },
    ],
}
```

### Rolling Accuracy Tracker

```python
from collections import deque
from datetime import datetime
from typing import Optional


class RollingAccuracyTracker:
    """Track model accuracy over a rolling window with SLO alerting."""

    def __init__(
        self,
        window_size: int = 1000,
        slo_threshold: float = 0.93,
        alert_callback=None,
    ):
        self.window_size = window_size
        self.slo_threshold = slo_threshold
        self.alert_callback = alert_callback
        self.results: deque = deque(maxlen=window_size)
        self.total_correct = 0
        self.total_predictions = 0

    def record(self, predicted, actual) -> dict:
        """Record a prediction-actual pair."""
        is_correct = predicted == actual
        self.total_predictions += 1

        if len(self.results) == self.window_size:
            # Remove oldest result from running total
            oldest = self.results[0]
            self.total_correct -= oldest

        self.results.append(int(is_correct))
        self.total_correct += int(is_correct)

        current_accuracy = self.total_correct / len(self.results)

        # Update Prometheus gauge
        MODEL_ACCURACY.labels(
            model_name="fraud_detector",
            model_version="v3",
        ).set(current_accuracy)

        # Check SLO
        if (
            len(self.results) >= self.window_size
            and current_accuracy < self.slo_threshold
        ):
            msg = (
                f"⚠️ Accuracy {current_accuracy:.2%} below SLO "
                f"{self.slo_threshold:.2%}"
            )
            if self.alert_callback:
                self.alert_callback(msg)

        return {
            "current_accuracy": round(current_accuracy, 4),
            "window_size": len(self.results),
            "total_predictions": self.total_predictions,
            "slo_met": current_accuracy >= self.slo_threshold,
        }


# --- Usage ---
tracker = RollingAccuracyTracker(window_size=500, slo_threshold=0.93)

for i in range(1000):
    # Simulate degrading accuracy
    correct_prob = 0.95 if i < 600 else 0.85
    predicted = 1
    actual = 1 if np.random.random() < correct_prob else 0

    result = tracker.record(predicted, actual)
    if i % 200 == 0:
        print(f"Step {i}: {result}")
```

### Docker Compose for Prometheus + Grafana

```python
DOCKER_COMPOSE = """
version: '3.8'

services:
  ml-service:
    build: .
    ports:
      - "8080:8080"   # API
      - "8000:8000"   # Prometheus metrics
    environment:
      - MODEL_NAME=fraud_detector
      - MODEL_VERSION=v3

  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.retention.time=30d'

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana-data:/var/lib/grafana
      - ./dashboards:/etc/grafana/provisioning/dashboards

volumes:
  grafana-data:
"""

PROMETHEUS_CONFIG = """
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'ml-service'
    static_configs:
      - targets: ['ml-service:8000']
    metrics_path: '/metrics'
    scrape_interval: 5s

  - job_name: 'ml-service-batch'
    static_configs:
      - targets: ['ml-service:8001']
    scrape_interval: 60s

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

rule_files:
  - 'alerts.yml'
"""

ALERT_RULES = """
groups:
  - name: ml_model_alerts
    rules:
      - alert: ModelLatencyHigh
        expr: histogram_quantile(0.99, rate(ml_prediction_latency_seconds_bucket[5m])) > 0.5
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Model p99 latency above 500ms"

      - alert: ModelErrorRateHigh
        expr: rate(ml_predictions_total{status="error"}[5m]) / rate(ml_predictions_total[5m]) > 0.05
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Model error rate above 5%"

      - alert: ModelAccuracyLow
        expr: ml_model_accuracy < 0.85
        for: 10m
        labels:
          severity: critical
        annotations:
          summary: "Model accuracy below 85%"

      - alert: ModelThroughputDrop
        expr: rate(ml_predictions_total[5m]) < 10
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Model throughput below 10 QPS"
"""
```

---

## 📊 Alerting Thresholds Reference

| Metric | Warning | Critical | Measurement Window |
|--------|---------|----------|-------------------|
| **p99 Latency** | > 200ms | > 500ms | 5 min rolling |
| **Error Rate** | > 1% | > 5% | 5 min rolling |
| **Model Accuracy** | < 93% | < 85% | 1000-sample window |
| **QPS Drop** | < 50% baseline | < 20% baseline | 5 min rolling |
| **GPU Utilization** | > 80% | > 95% | 1 min average |
| **Memory Usage** | > 75% | > 90% | 1 min average |
| **Prediction Null Rate** | > 0.01% | > 0.1% | 5 min rolling |
| **Feature Missing Rate** | > 0.1% | > 1% | 5 min rolling |

---

## 🏗️ Full Monitoring Stack Architecture

```
  ┌──────────────────────────────────────────────────────────────────┐
  │                    ML MONITORING STACK                           │
  │                                                                  │
  │  ┌────────────┐  ┌────────────┐  ┌────────────┐                │
  │  │ ML Service │  │ ML Service │  │ ML Service │  (replicas)    │
  │  │  :8000     │  │  :8000     │  │  :8000     │                │
  │  └─────┬──────┘  └─────┬──────┘  └─────┬──────┘                │
  │        │               │               │                        │
  │        └───────────────┼───────────────┘                        │
  │                        │ /metrics (Prometheus format)           │
  │                        ▼                                        │
  │              ┌──────────────────┐                               │
  │              │   PROMETHEUS     │                               │
  │              │   ────────────   │                               │
  │              │   Scrapes every  │                               │
  │              │   15 seconds     │                               │
  │              │   Stores 30 days │                               │
  │              │   Evaluates rules│                               │
  │              └────────┬─────────┘                               │
  │                       │                                         │
  │           ┌───────────┼───────────┐                             │
  │           │           │           │                             │
  │           ▼           ▼           ▼                             │
  │  ┌──────────────┐ ┌────────┐ ┌───────────────┐                │
  │  │ ALERTMANAGER │ │GRAFANA │ │ CUSTOM        │                │
  │  │ ──────────── │ │──────  │ │ DRIFT MONITOR │                │
  │  │ Routes alerts│ │Dashbds │ │ ───────────── │                │
  │  │ to channels  │ │& viz   │ │ Evidently /   │                │
  │  └──────┬───────┘ └────────┘ │ custom stats  │                │
  │         │                     └───────────────┘                │
  │         ▼                                                       │
  │  ┌──────────────────────────────┐                              │
  │  │  Slack / PagerDuty / Email   │                              │
  │  └──────────────────────────────┘                              │
  └──────────────────────────────────────────────────────────────────┘
```

---

## 🔑 Key Takeaways

| # | Takeaway |
|---|----------|
| 1 | **Monitor tail latencies, not averages** — p99 reveals the experience of your worst-affected users, which mean latency hides. |
| 2 | **Three pillars matter equally** — model quality, latency, and throughput must all be tracked; a fast but inaccurate model is just as broken as a slow one. |
| 3 | **Prometheus + Grafana is the industry standard** — instrument your ML service with Prometheus client libraries and visualize with Grafana dashboards. |
| 4 | **Set thresholds based on baselines** — establish performance baselines during deployment, then alert on deviations rather than arbitrary numbers. |
| 5 | **Prediction distribution monitoring** is a leading indicator — shifts in output distribution often signal problems before accuracy metrics do. |

---

## ❓ Revision Questions

1. **Why should you monitor p99 latency instead of mean latency for ML services?**
   > Mean latency can be misleadingly low because a small number of very slow requests are averaged out by the majority of fast ones. p99 latency represents the worst 1% of requests — these are often the most important because they may indicate resource contention, model complexity issues, or infrastructure problems. SLAs are typically defined on percentiles, not means.

2. **What Prometheus metric types are most useful for ML monitoring, and when do you use each?**
   > **Counter** for total predictions (monotonically increasing, use `rate()` for QPS). **Histogram** for latency distributions (enables percentile calculation via `histogram_quantile()`). **Gauge** for point-in-time values like current accuracy or GPU utilization. **Summary** for streaming quantile estimation of feature values.

3. **How would you detect a model that returns valid predictions but with degraded accuracy?**
   > Monitor rolling accuracy by comparing predictions against delayed ground truth labels. Track prediction distribution shifts (e.g., if a fraud model suddenly predicts fraud for 15% of traffic instead of 2%, something is wrong). Monitor feature importance drift. Set alerts on accuracy dropping below SLO thresholds measured over a rolling window.

4. **What is the difference between model quality metrics and system health metrics?**
   > Model quality metrics (accuracy, F1, RMSE) measure whether the model's outputs are correct — they require ground truth labels. System health metrics (latency, throughput, error rate, CPU/memory) measure whether the serving infrastructure is performing well — they can be collected without labels. A model can have perfect system health but terrible quality, or vice versa.

5. **How would you design alerting thresholds for a newly deployed model with no historical baseline?**
   > Use shadow mode or canary deployment to establish baselines during the first 1-2 weeks. Measure p50/p95/p99 latency, error rate, and prediction distributions during this baseline period. Set warning thresholds at 1.5× baseline and critical at 2× baseline. For accuracy, use offline evaluation metrics as the initial target. Adjust thresholds as you collect more production data.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Concept Drift](./02-concept-drift.md) | [📂 Model Monitoring](./README.md) | [Alerting Systems →](./04-alerting.md) |

---

*Unit 8 · Chapter 3 · Performance Monitoring — MLOps Course Notes*
