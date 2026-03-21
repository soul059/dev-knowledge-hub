# 🧪 A/B Testing for ML Models

> **Unit 7 · Chapter 2** — Statistical experimentation for comparing ML model performance in production using controlled traffic splits.

---

## 📋 Chapter Overview

A/B testing in ML goes beyond traditional web experimentation. Instead of comparing button colors, you're comparing **entire prediction models** — each with different architectures, training data, or hyperparameters. The goal: determine which model drives better **business outcomes** with statistical confidence.

This chapter covers:

- Designing A/B tests for ML models
- Traffic splitting and randomization
- Statistical significance testing
- Sample size calculation
- Common pitfalls and best practices

---

## 🏗️ A/B Testing Architecture

```
                          ┌──────────────────────────┐
                          │     Experiment Service    │
                          │  ┌────────────────────┐  │
                          │  │ Traffic Allocator   │  │
                          │  │ ────────────────    │  │
                          │  │ User Hash → Bucket  │  │
                          │  └─────────┬──────────┘  │
                          └────────────┼──────────────┘
                                       │
                         ┌─────────────┼─────────────┐
                         │                           │
                         ▼                           ▼
                ┌─────────────────┐         ┌─────────────────┐
                │  Control (A)    │         │  Treatment (B)  │
                │  ────────────   │         │  ────────────   │
                │  Model v1       │         │  Model v2       │
                │  (50% traffic)  │         │  (50% traffic)  │
                └────────┬────────┘         └────────┬────────┘
                         │                           │
                         ▼                           ▼
                ┌─────────────────────────────────────────────┐
                │           Metrics Collector                 │
                │  ─────────────────────────────────────────  │
                │  • Predictions    • Latency                 │
                │  • Conversions    • Error rates              │
                │  • Revenue        • User satisfaction        │
                └────────────────────┬────────────────────────┘
                                     │
                                     ▼
                          ┌──────────────────────┐
                          │  Statistical Analysis │
                          │  ────────────────────  │
                          │  p-value, CI, effect   │
                          │  size, power analysis  │
                          └──────────────────────┘
```

---

## 📐 Statistical Setup

### Hypothesis Framework

| Element                 | Definition                                             |
|-------------------------|--------------------------------------------------------|
| **Null Hypothesis (H₀)**     | Model B is no better than Model A (μ_B = μ_A)    |
| **Alt Hypothesis (H₁)**      | Model B is better than Model A (μ_B > μ_A)       |
| **Significance Level (α)**   | Probability of false positive (typically 0.05)    |
| **Power (1 − β)**            | Probability of detecting a real effect (≥ 0.80)  |
| **Minimum Detectable Effect** | Smallest meaningful difference to detect          |

### Sample Size Calculation

```
  Required Sample Size per Group:
  ────────────────────────────────

           2 × (Z_α/2 + Z_β)² × σ²
  n  =  ──────────────────────────────
                   δ²

  Where:
    Z_α/2  = 1.96  (for α = 0.05, two-tailed)
    Z_β    = 0.84  (for power = 0.80)
    σ      = standard deviation of metric
    δ      = minimum detectable effect
```

### Python — Sample Size Calculation

```python
import math
from scipy import stats

def calculate_sample_size(
    baseline_rate: float,
    minimum_detectable_effect: float,
    alpha: float = 0.05,
    power: float = 0.80,
) -> int:
    """Calculate required sample size per group for a two-proportion z-test."""
    p1 = baseline_rate
    p2 = baseline_rate + minimum_detectable_effect
    
    # Pooled proportion
    p_bar = (p1 + p2) / 2
    
    z_alpha = stats.norm.ppf(1 - alpha / 2)
    z_beta = stats.norm.ppf(power)
    
    numerator = (z_alpha * math.sqrt(2 * p_bar * (1 - p_bar)) +
                 z_beta * math.sqrt(p1 * (1 - p1) + p2 * (1 - p2))) ** 2
    denominator = (p2 - p1) ** 2
    
    return math.ceil(numerator / denominator)

# Example: baseline CTR = 5%, want to detect 1% lift
n = calculate_sample_size(baseline_rate=0.05, minimum_detectable_effect=0.01)
print(f"Required sample size per group: {n:,}")
# Output: Required sample size per group: 4,720
```

---

## 🔀 Traffic Splitting

### Consistent Hashing for User Assignment

Users must **always** see the same model variant during an experiment to avoid bias.

```python
import hashlib

def assign_variant(user_id: str, experiment_id: str, split_ratio: float = 0.5) -> str:
    """Deterministically assign a user to control or treatment."""
    hash_key = f"{experiment_id}:{user_id}"
    hash_value = int(hashlib.sha256(hash_key.encode()).hexdigest(), 16)
    
    # Normalize to [0, 1)
    bucket = (hash_value % 10000) / 10000.0
    
    return "treatment" if bucket < split_ratio else "control"

# Same user always gets the same assignment
print(assign_variant("user_123", "exp_model_v2"))   # "control"
print(assign_variant("user_123", "exp_model_v2"))   # "control" (consistent)
print(assign_variant("user_456", "exp_model_v2"))   # "treatment"
```

### Traffic Split Patterns

```
  50/50 Split (Standard)         90/10 Split (Conservative)
  ┌─────────┬─────────┐         ┌───────────────────┬──┐
  │ Control │Treatment│         │    Control        │T │
  │  (50%)  │  (50%)  │         │     (90%)         │  │
  └─────────┴─────────┘         └───────────────────┴──┘

  Multi-variant (A/B/C)          Holdout + Treatment
  ┌──────┬──────┬──────┐        ┌──────────────┬────┬───┐
  │  A   │  B   │  C   │        │  Production  │Hold│ T │
  │(34%) │(33%) │(33%) │        │    (80%)     │10% │10%│
  └──────┴──────┴──────┘        └──────────────┴────┴───┘
```

---

## 📊 Significance Testing

### Full A/B Test Pipeline — Python Example

```python
import numpy as np
from scipy import stats
from dataclasses import dataclass
from typing import Optional

@dataclass
class ABTestResult:
    control_mean: float
    treatment_mean: float
    lift: float
    p_value: float
    confidence_interval: tuple
    is_significant: bool
    recommendation: str

class ABTestAnalyzer:
    def __init__(self, alpha: float = 0.05):
        self.alpha = alpha

    def analyze_conversion(
        self,
        control_conversions: int,
        control_total: int,
        treatment_conversions: int,
        treatment_total: int,
    ) -> ABTestResult:
        """Analyze A/B test results for conversion rate metrics."""
        p_control = control_conversions / control_total
        p_treatment = treatment_conversions / treatment_total
        
        lift = (p_treatment - p_control) / p_control
        
        # Two-proportion z-test
        p_pooled = (control_conversions + treatment_conversions) / (
            control_total + treatment_total
        )
        se = np.sqrt(p_pooled * (1 - p_pooled) * (1/control_total + 1/treatment_total))
        z_stat = (p_treatment - p_control) / se
        p_value = 2 * (1 - stats.norm.cdf(abs(z_stat)))
        
        # Confidence interval for the difference
        se_diff = np.sqrt(
            p_control * (1 - p_control) / control_total +
            p_treatment * (1 - p_treatment) / treatment_total
        )
        ci = (
            (p_treatment - p_control) - 1.96 * se_diff,
            (p_treatment - p_control) + 1.96 * se_diff,
        )
        
        is_significant = p_value < self.alpha
        
        if is_significant and lift > 0:
            recommendation = "✅ Deploy treatment model — statistically significant improvement"
        elif is_significant and lift < 0:
            recommendation = "❌ Keep control — treatment is significantly worse"
        else:
            recommendation = "⏳ Not significant — continue experiment or increase sample size"
        
        return ABTestResult(
            control_mean=p_control,
            treatment_mean=p_treatment,
            lift=lift,
            p_value=p_value,
            confidence_interval=ci,
            is_significant=is_significant,
            recommendation=recommendation,
        )

    def analyze_continuous(
        self,
        control_values: np.ndarray,
        treatment_values: np.ndarray,
    ) -> ABTestResult:
        """Analyze A/B test for continuous metrics (e.g., revenue, latency)."""
        control_mean = np.mean(control_values)
        treatment_mean = np.mean(treatment_values)
        lift = (treatment_mean - control_mean) / control_mean
        
        # Welch's t-test (unequal variances)
        t_stat, p_value = stats.ttest_ind(
            treatment_values, control_values, equal_var=False
        )
        
        # Confidence interval
        se = np.sqrt(
            np.var(control_values, ddof=1) / len(control_values) +
            np.var(treatment_values, ddof=1) / len(treatment_values)
        )
        diff = treatment_mean - control_mean
        ci = (diff - 1.96 * se, diff + 1.96 * se)
        
        is_significant = p_value < self.alpha
        
        if is_significant and lift > 0:
            recommendation = "✅ Deploy treatment model"
        elif is_significant and lift < 0:
            recommendation = "❌ Keep control model"
        else:
            recommendation = "⏳ Not yet significant"
        
        return ABTestResult(
            control_mean=control_mean,
            treatment_mean=treatment_mean,
            lift=lift,
            p_value=p_value,
            confidence_interval=ci,
            is_significant=is_significant,
            recommendation=recommendation,
        )

# ─── Example Usage ──────────────────────────────────────────

analyzer = ABTestAnalyzer(alpha=0.05)

# Conversion rate test
result = analyzer.analyze_conversion(
    control_conversions=512,
    control_total=10000,
    treatment_conversions=578,
    treatment_total=10000,
)

print(f"Control CR:  {result.control_mean:.4f}")
print(f"Treatment CR:{result.treatment_mean:.4f}")
print(f"Lift:        {result.lift:+.2%}")
print(f"P-value:     {result.p_value:.4f}")
print(f"95% CI:      [{result.confidence_interval[0]:.4f}, {result.confidence_interval[1]:.4f}]")
print(f"Significant: {result.is_significant}")
print(f"→ {result.recommendation}")
```

**Sample Output:**
```
Control CR:  0.0512
Treatment CR:0.0578
Lift:        +12.89%
P-value:     0.0421
95% CI:      [0.0002, 0.0130]
Significant: True
→ ✅ Deploy treatment model — statistically significant improvement
```

---

## ⚠️ Common Pitfalls

| Pitfall | Problem | Solution |
|---------|---------|----------|
| **Peeking** | Checking results too early inflates false positives | Pre-define sample size; use sequential testing |
| **Multiple Testing** | Testing many metrics increases error rate | Apply Bonferroni or Benjamini-Hochberg correction |
| **Simpson's Paradox** | Aggregated results hide segment-level differences | Analyze by user segments |
| **Novelty Effect** | Users behave differently just because something is new | Run experiments for ≥ 2 weeks |
| **Interference** | Users in one group affect users in another | Use cluster-based randomization |
| **Survivorship Bias** | Only analyzing users who stayed through the test | Include all users from assignment time |

---

## 📐 Sequential Testing (Optional Early Stopping)

When you can't wait for full sample size, use **sequential analysis**:

```python
from scipy.stats import norm
import numpy as np

def sequential_test(
    cumulative_control: np.ndarray,
    cumulative_treatment: np.ndarray,
    alpha: float = 0.05,
    beta: float = 0.20,
) -> str:
    """O'Brien-Fleming-style boundary check for sequential testing."""
    n = len(cumulative_control)
    
    # Current means
    mean_c = np.mean(cumulative_control)
    mean_t = np.mean(cumulative_treatment)
    
    # Pooled SE
    se = np.sqrt(
        np.var(cumulative_control, ddof=1) / n +
        np.var(cumulative_treatment, ddof=1) / n
    )
    
    if se == 0:
        return "insufficient_data"
    
    z_stat = (mean_t - mean_c) / se
    
    # O'Brien-Fleming spending function (conservative early, liberal late)
    info_fraction = n / 10000  # assuming 10k target per group
    boundary = norm.ppf(1 - alpha / 2) / np.sqrt(info_fraction)
    
    if abs(z_stat) > boundary:
        return "significant" if z_stat > 0 else "significant_negative"
    return "continue"
```

---

## 🔑 Key Takeaways

| # | Takeaway |
|---|----------|
| 1 | A/B tests compare models on **business metrics**, not just ML metrics like accuracy. |
| 2 | **Consistent user hashing** ensures the same user always sees the same model variant. |
| 3 | **Pre-calculate sample size** to know how long the experiment needs to run. |
| 4 | Avoid peeking — use sequential testing if early stopping is needed. |
| 5 | Always check for **practical significance**, not just statistical significance. |

---

## ❓ Revision Questions

1. **Why is consistent hashing important in A/B testing for ML models?**
   > Without consistent hashing, a user might see different model versions across requests, contaminating both groups and making metrics unreliable.

2. **What is the difference between statistical significance and practical significance?**
   > Statistical significance means the difference is unlikely due to chance (p < α). Practical significance means the difference is large enough to matter for the business (e.g., a 0.01% CTR lift may be statistically significant but not worth the deployment effort).

3. **How does the multiple testing problem affect A/B experiments?**
   > Testing many metrics simultaneously increases the chance of finding a false positive. If you test 20 metrics at α = 0.05, you expect ~1 false positive by chance. Corrections like Bonferroni adjust the threshold.

4. **When would you use Welch's t-test vs. a two-proportion z-test?**
   > Use Welch's t-test for continuous metrics (e.g., revenue per user, latency). Use a two-proportion z-test for binary outcomes (e.g., conversion yes/no, click yes/no).

5. **A team sees a 15% lift in their A/B test after just 2 hours. Should they deploy?**
   > No — this is "peeking." Early results are unreliable due to small sample sizes and time-of-day effects. They should wait until the pre-calculated sample size is reached, or use sequential testing with proper boundaries.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Deployment Strategies](./01-deployment-strategies.md) | [📂 Model Deployment](./README.md) | [Canary Deployments →](./03-canary-deployments.md) |

---

*Unit 7 · Chapter 2 · A/B Testing — MLOps Course Notes*
