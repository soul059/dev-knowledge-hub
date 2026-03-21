# 🔬 Model Testing

> **Unit 10 · ML Testing · Chapter 3**
>
> How to test model behavior beyond accuracy — behavioral testing with CheckList, invariance tests, directional expectation tests, minimum functionality tests, testing for bias and fairness, and stress testing models under adversarial and edge-case inputs.

---

## Table of Contents

1. [Chapter Overview](#chapter-overview)
2. [Beyond Accuracy Metrics](#beyond-accuracy-metrics)
3. [Behavioral Testing with CheckList](#behavioral-testing-with-checklist)
4. [Invariance Tests](#invariance-tests)
5. [Directional Expectation Tests](#directional-expectation-tests)
6. [Minimum Functionality Tests](#minimum-functionality-tests)
7. [Testing for Bias and Fairness](#testing-for-bias-and-fairness)
8. [Stress Testing Models](#stress-testing-models)
9. [Putting It Together: Model Test Suite](#putting-it-together-model-test-suite)
10. [Summary](#summary)
11. [Revision Questions](#revision-questions)
12. [Navigation](#navigation)

---

## Chapter Overview

A model can have 95% accuracy and still fail catastrophically in production. Aggregate metrics hide **specific failures** — the model might ace common cases but break on negation, synonyms, or minority groups. This chapter covers **behavioral testing** — a paradigm inspired by software testing but adapted for ML. Rather than testing code paths, we test **capabilities**: Does the model handle negation? Is it robust to irrelevant changes? Does it perform equally across demographic groups? We cover the **CheckList** framework (from Microsoft Research), invariance and directional tests, minimum functionality tests (MFTs), bias auditing, and stress testing under adversarial conditions.

```
Traditional Model Evaluation           Behavioral Model Testing
┌──────────────────────────┐           ┌──────────────────────────────┐
│                          │           │                              │
│  Test set accuracy: 94%  │           │  ✅ Invariance: 98% pass     │
│  F1 score: 0.91          │           │  ✅ Directional: 95% pass    │
│  AUC: 0.96               │           │  ✅ MFT (negation): 91%      │
│                          │           │  ❌ MFT (temporal): 62% ←    │
│  "Ship it!" 🚀           │           │  ❌ Bias (gender): gap 12%   │
│                          │           │  ✅ Stress (typos): 89%      │
│  (What could go wrong?)  │           │                              │
│                          │           │  "Fix temporal and bias      │
│                          │           │   before shipping"           │
└──────────────────────────┘           └──────────────────────────────┘
```

---

## Beyond Accuracy Metrics

### Why Accuracy Isn't Enough

| Problem | Example | Aggregate Metric |
|---|---|---|
| **Class imbalance** | 99% accuracy on fraud detection — predicts "not fraud" for everything | Accuracy: 99% ✅ (useless) |
| **Spurious correlations** | Model learned "hospital" → positive for disease | AUC: 0.95 ✅ (brittle) |
| **Subgroup failures** | Works for English, fails for code-switched text | F1: 0.90 ✅ (hides gaps) |
| **Negation blindness** | "This is great" → positive, "This is not great" → positive | Accuracy: 88% ✅ (wrong) |

### The Behavioral Testing Philosophy

```
Software Testing Analogy:

  Unit test:     "Does login() return a token for valid credentials?"
  Not:           "What is the average success rate of all API calls?"

Model Testing:

  Behavioral test: "Does the model correctly flip sentiment for negation?"
  Not:             "What is the average F1 on the test set?"

  ┌──────────────────────────────────────────────────────┐
  │  Test CAPABILITIES, not aggregate PERFORMANCE        │
  │  Test SPECIFIC behaviors on CRAFTED inputs           │
  │  Each test targets ONE linguistic/logical phenomenon │
  └──────────────────────────────────────────────────────┘
```

---

## Behavioral Testing with CheckList

**CheckList** (Ribeiro et al., 2020) is a framework that structures model testing into three test types, applied across multiple **capabilities**.

### CheckList Framework

```
┌───────────────────────────────────────────────────────────────┐
│                        CheckList                              │
│                                                               │
│  Test Types:                                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌──────────────────────┐  │
│  │     MFT     │  │     INV     │  │        DIR           │  │
│  │  Minimum    │  │ Invariance  │  │   Directional        │  │
│  │ Functionality│  │   Test     │  │   Expectation        │  │
│  │             │  │             │  │                      │  │
│  │ "Simple     │  │ "Change X,  │  │ "Change X,           │  │
│  │  cases the  │  │  output     │  │  output should       │  │
│  │  model MUST │  │  should NOT │  │  change in           │  │
│  │  get right" │  │  change"    │  │  direction Y"        │  │
│  └─────────────┘  └─────────────┘  └──────────────────────┘  │
│                                                               │
│  Applied across Capabilities:                                 │
│  • Vocabulary    • Negation    • Robustness                   │
│  • Taxonomy      • Fairness   • Temporal                      │
│  • Named Entities • Coreference • Logic                       │
└───────────────────────────────────────────────────────────────┘
```

### Setting Up CheckList Tests

```python
# tests/test_model_behavior.py
import pytest
import numpy as np


@pytest.fixture
def sentiment_model():
    """Load sentiment analysis model."""
    from src.model import load_sentiment_model
    return load_sentiment_model("models/sentiment_v2.pt")


def predict_sentiment(model, text: str) -> dict:
    """Returns {'label': 'positive'|'negative', 'score': float}."""
    return model.predict(text)
```

---

## Invariance Tests

**Invariance tests (INV):** changing the input in a way that **should not** change the output.

### Concept

```
Original Input                 Perturbed Input              Expected
─────────────                  ─────────────────            ────────
"The food was great"           "The food was great!!!"      Same label
"I love this movie"            "I LOVE this movie"          Same label
"John enjoyed the hotel"       "Mary enjoyed the hotel"     Same label
"Great service in NYC"         "Great service in London"    Same label
```

### Implementation

```python
# tests/test_invariance.py
import pytest


class TestInvariance:
    """Changes that should NOT affect model output."""

    @pytest.mark.parametrize("original, perturbed", [
        # Name changes should not affect sentiment
        ("John loved the restaurant", "Maria loved the restaurant"),
        ("David hated the service", "Sarah hated the service"),
        # Location changes
        ("Great hotel in Paris", "Great hotel in Tokyo"),
        ("Terrible food in NYC", "Terrible food in London"),
    ])
    def test_name_location_invariance(self, sentiment_model, original, perturbed):
        orig_pred = predict_sentiment(sentiment_model, original)
        pert_pred = predict_sentiment(sentiment_model, perturbed)
        assert orig_pred["label"] == pert_pred["label"], \
            f"'{original}' → {orig_pred['label']}, " \
            f"'{perturbed}' → {pert_pred['label']}"

    @pytest.mark.parametrize("original, perturbed", [
        # Typos should not flip prediction
        ("This product is excellent", "This prodcut is excelent"),
        ("Horrible experience", "Horible experiance"),
    ])
    def test_typo_invariance(self, sentiment_model, original, perturbed):
        orig_pred = predict_sentiment(sentiment_model, original)
        pert_pred = predict_sentiment(sentiment_model, perturbed)
        assert orig_pred["label"] == pert_pred["label"]

    @pytest.mark.parametrize("original, perturbed", [
        # Adding neutral context should not change sentiment
        ("The movie was boring", "I watched it on Tuesday. The movie was boring"),
        ("Amazing food!", "We went at 7pm. Amazing food!"),
    ])
    def test_neutral_context_invariance(self, sentiment_model, original, perturbed):
        orig_pred = predict_sentiment(sentiment_model, original)
        pert_pred = predict_sentiment(sentiment_model, perturbed)
        assert orig_pred["label"] == pert_pred["label"]


def generate_invariance_pairs(templates, names):
    """Generate test pairs by swapping names in templates."""
    pairs = []
    for template in templates:
        for i, name_a in enumerate(names):
            for name_b in names[i+1:]:
                original = template.format(name=name_a)
                perturbed = template.format(name=name_b)
                pairs.append((original, perturbed))
    return pairs

# Example usage
TEMPLATES = [
    "{name} loved the food",
    "{name} was disappointed by the service",
]
NAMES = ["James", "Aisha", "Chen", "Fatima"]
INVARIANCE_PAIRS = generate_invariance_pairs(TEMPLATES, NAMES)
```

---

## Directional Expectation Tests

**Directional tests (DIR):** changing the input in a way that **should** change the output in a known direction.

### Concept

```
Original Input                 Perturbation                 Expected Direction
─────────────                  ─────────────                ──────────────────
"The food was good"            Add "but service was awful"  Sentiment ↓ (more negative)
"Decent hotel"                 Add "with amazing views"     Sentiment ↑ (more positive)
"Average product"              Add "broke after one day"    Sentiment ↓ (more negative)
```

### Implementation

```python
# tests/test_directional.py
import pytest


class TestDirectional:
    """Changes that SHOULD affect model output in a predictable direction."""

    @pytest.mark.parametrize("base, addition, direction", [
        # Adding negative phrases should decrease sentiment
        ("The food was good", "but the service was terrible", "decrease"),
        ("Nice hotel", "but the room was filthy", "decrease"),
        ("Good product", "but it broke after one day", "decrease"),
        # Adding positive phrases should increase sentiment
        ("Average restaurant", "with an incredible view", "increase"),
        ("Okay movie", "with outstanding performances", "increase"),
    ])
    def test_sentiment_direction(self, sentiment_model, base, addition, direction):
        base_pred = predict_sentiment(sentiment_model, base)
        modified = f"{base}, {addition}"
        mod_pred = predict_sentiment(sentiment_model, modified)

        if direction == "decrease":
            assert mod_pred["score"] < base_pred["score"], \
                f"Expected decrease: '{base}' ({base_pred['score']:.3f}) " \
                f"→ '{modified}' ({mod_pred['score']:.3f})"
        else:
            assert mod_pred["score"] > base_pred["score"], \
                f"Expected increase: '{base}' ({base_pred['score']:.3f}) " \
                f"→ '{modified}' ({mod_pred['score']:.3f})"

    @pytest.mark.parametrize("base, negated", [
        ("I love this product", "I do not love this product"),
        ("The movie was great", "The movie was not great"),
        ("Excellent service", "Not excellent service"),
    ])
    def test_negation_flips_sentiment(self, sentiment_model, base, negated):
        """Negation should flip or significantly reduce sentiment."""
        base_pred = predict_sentiment(sentiment_model, base)
        neg_pred = predict_sentiment(sentiment_model, negated)
        assert neg_pred["score"] < base_pred["score"] - 0.3, \
            f"Negation didn't reduce sentiment enough: " \
            f"{base_pred['score']:.3f} → {neg_pred['score']:.3f}"
```

### Directional Tests for Tabular Models

```python
# tests/test_directional_tabular.py
import numpy as np
import pytest


class TestDirectionalTabular:
    """For a house price model, certain changes should predictably
    increase or decrease predictions."""

    @pytest.fixture
    def base_features(self):
        return {
            "sqft": 1500,
            "bedrooms": 3,
            "bathrooms": 2,
            "year_built": 2000,
            "lot_size": 5000,
        }

    @pytest.mark.parametrize("feature, delta, direction", [
        ("sqft", +500, "increase"),         # More sqft → higher price
        ("bedrooms", +1, "increase"),       # More bedrooms → higher price
        ("year_built", +20, "increase"),    # Newer → higher price
        ("lot_size", +2000, "increase"),    # Bigger lot → higher price
    ])
    def test_feature_direction(
        self, house_model, base_features, feature, delta, direction
    ):
        import pandas as pd
        base_df = pd.DataFrame([base_features])
        modified = base_features.copy()
        modified[feature] += delta
        mod_df = pd.DataFrame([modified])

        base_price = house_model.predict(base_df)[0]
        mod_price = house_model.predict(mod_df)[0]

        if direction == "increase":
            assert mod_price > base_price, \
                f"Increasing {feature} by {delta}: " \
                f"${base_price:,.0f} → ${mod_price:,.0f} (expected increase)"
```

---

## Minimum Functionality Tests

**Minimum functionality tests (MFT):** simple, unambiguous cases the model **must** get right. If it fails these, it has a fundamental problem.

### Concept

```
MFTs are the "sanity checks" of ML testing:

  ┌────────────────────────────────────────────────────────┐
  │  "If the model can't get THESE right, nothing else     │
  │   matters."                                            │
  │                                                        │
  │  • "I love this!" → Positive (obvious positive)        │
  │  • "This is terrible" → Negative (obvious negative)    │
  │  • "I am happy" → Positive (basic emotion word)        │
  │  • "I am sad" → Negative (basic emotion word)          │
  │  • "Not good" → Negative (simple negation)             │
  └────────────────────────────────────────────────────────┘
```

### Implementation

```python
# tests/test_mft.py
import pytest


class TestMFTSentiment:
    """Minimum functionality tests — unambiguous cases."""

    @pytest.mark.parametrize("text, expected_label", [
        # Basic positive
        ("I love this!", "positive"),
        ("This is amazing!", "positive"),
        ("Best product ever", "positive"),
        ("Absolutely wonderful", "positive"),
        ("Highly recommended", "positive"),
        # Basic negative
        ("I hate this", "negative"),
        ("This is terrible", "negative"),
        ("Worst experience ever", "negative"),
        ("Absolutely awful", "negative"),
        ("Do not buy this", "negative"),
    ])
    def test_obvious_sentiment(self, sentiment_model, text, expected_label):
        pred = predict_sentiment(sentiment_model, text)
        assert pred["label"] == expected_label, \
            f"'{text}' predicted as {pred['label']} (expected {expected_label})"

    @pytest.mark.parametrize("text, expected_label", [
        # Negation
        ("This is not good", "negative"),
        ("I don't like it", "negative"),
        ("Not recommended", "negative"),
        ("It's not bad", "positive"),       # double negation
        ("I can't complain", "positive"),
    ])
    def test_negation(self, sentiment_model, text, expected_label):
        pred = predict_sentiment(sentiment_model, text)
        assert pred["label"] == expected_label

    @pytest.mark.parametrize("text, expected_label", [
        # Temporal expressions
        ("The food used to be good but now it's terrible", "negative"),
        ("It was bad but they improved", "positive"),
    ])
    def test_temporal(self, sentiment_model, text, expected_label):
        pred = predict_sentiment(sentiment_model, text)
        assert pred["label"] == expected_label


class TestMFTTabular:
    """MFTs for a credit risk model."""

    def test_high_income_low_debt_is_low_risk(self, credit_model):
        features = {"income": 200000, "debt": 5000, "credit_score": 800}
        risk = credit_model.predict_proba(features)
        assert risk < 0.1, f"Obviously low-risk customer scored {risk:.2f}"

    def test_zero_income_high_debt_is_high_risk(self, credit_model):
        features = {"income": 0, "debt": 100000, "credit_score": 300}
        risk = credit_model.predict_proba(features)
        assert risk > 0.8, f"Obviously high-risk customer scored {risk:.2f}"
```

---

## Testing for Bias and Fairness

### Types of Bias to Test

```
┌───────────────────────────────────────────────────────────────┐
│                    Bias Testing Dimensions                    │
│                                                               │
│  Demographic Parity:   P(ŷ=1 | group=A) ≈ P(ŷ=1 | group=B) │
│  Equal accuracy across groups                                 │
│                                                               │
│  Equalized Odds:       TPR and FPR equal across groups        │
│  Same error rates for all groups                              │
│                                                               │
│  Counterfactual:       Changing a protected attribute          │
│                        should not change the prediction        │
│                                                               │
│  Protected Attributes: gender, race, age, disability,         │
│                        religion, nationality                   │
└───────────────────────────────────────────────────────────────┘
```

### Implementation

```python
# tests/test_bias.py
import pandas as pd
import numpy as np
import pytest


class TestFairness:
    @pytest.fixture
    def predictions_by_group(self, model, test_data):
        """Get predictions split by demographic group."""
        test_data["prediction"] = model.predict(test_data.drop(columns=["label"]))
        test_data["correct"] = test_data["prediction"] == test_data["label"]
        return test_data

    def test_demographic_parity(self, predictions_by_group):
        """Positive prediction rate should be similar across groups."""
        max_gap = 0.1   # 10% max allowed gap
        groups = predictions_by_group.groupby("gender")["prediction"].mean()
        gap = groups.max() - groups.min()
        assert gap < max_gap, \
            f"Demographic parity gap: {gap:.3f}\n{groups}"

    def test_equalized_odds(self, predictions_by_group):
        """True positive rate should be similar across groups."""
        max_gap = 0.1
        df = predictions_by_group
        tpr_by_group = {}
        for group in df["gender"].unique():
            subset = df[df["gender"] == group]
            positives = subset[subset["label"] == 1]
            if len(positives) > 0:
                tpr_by_group[group] = positives["correct"].mean()

        tpr_values = list(tpr_by_group.values())
        gap = max(tpr_values) - min(tpr_values)
        assert gap < max_gap, \
            f"TPR gap: {gap:.3f}\n{tpr_by_group}"

    def test_accuracy_across_groups(self, predictions_by_group):
        """Accuracy should not vary dramatically by group."""
        max_gap = 0.05  # 5%
        acc_by_group = predictions_by_group.groupby("gender")["correct"].mean()
        gap = acc_by_group.max() - acc_by_group.min()
        assert gap < max_gap, \
            f"Accuracy gap: {gap:.3f}\n{acc_by_group}"


class TestCounterfactualFairness:
    """Changing protected attributes should not change predictions."""

    @pytest.mark.parametrize("original, counterfactual", [
        # Gender swaps
        ("John is a qualified candidate", "Jane is a qualified candidate"),
        ("He has 10 years of experience", "She has 10 years of experience"),
        # Name-based proxies
        ("Muhammad applied for a loan", "Michael applied for a loan"),
        ("Chen submitted the application", "Smith submitted the application"),
    ])
    def test_counterfactual_name(self, model, original, counterfactual):
        orig_pred = model.predict(original)
        cf_pred = model.predict(counterfactual)
        assert abs(orig_pred["score"] - cf_pred["score"]) < 0.1, \
            f"Counterfactual gap: '{original}' ({orig_pred['score']:.3f}) " \
            f"vs '{counterfactual}' ({cf_pred['score']:.3f})"
```

### Fairness Metrics Comparison

| Metric | Formula | What It Measures | Threshold |
|---|---|---|---|
| **Demographic Parity** | \|P(ŷ=1\|A) − P(ŷ=1\|B)\| | Equal positive rates | < 0.10 |
| **Equalized Odds** | \|TPR_A − TPR_B\| and \|FPR_A − FPR_B\| | Equal error rates | < 0.10 |
| **Predictive Parity** | \|PPV_A − PPV_B\| | Equal precision | < 0.10 |
| **Counterfactual** | \|f(x_A) − f(x_B)\| | Swap-and-check | < 0.05 |
| **Calibration** | P(Y=1\|ŷ=p, A) ≈ p | Equal calibration | p ± 0.05 |

---

## Stress Testing Models

Stress testing pushes the model beyond normal operating conditions to find breaking points.

### Stress Test Categories

```
┌──────────────────────────────────────────────────────────────┐
│                    Stress Test Types                         │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐ │
│  │  Adversarial │  │  Edge Cases  │  │  Out-of-Domain     │ │
│  │              │  │              │  │                    │ │
│  │  Crafted     │  │  Extremely   │  │  Inputs from a     │ │
│  │  inputs to   │  │  long/short  │  │  distribution the  │ │
│  │  fool the    │  │  Empty input │  │  model has never   │ │
│  │  model       │  │  Unicode     │  │  seen              │ │
│  │              │  │  All zeros   │  │                    │ │
│  └──────────────┘  └──────────────┘  └────────────────────┘ │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐                         │
│  │  Load Stress │  │  Data Noise  │                         │
│  │              │  │              │                         │
│  │  Large batch │  │  Gaussian    │                         │
│  │  Concurrent  │  │  noise added │                         │
│  │  requests    │  │  to features │                         │
│  └──────────────┘  └──────────────┘                         │
└──────────────────────────────────────────────────────────────┘
```

### Implementation

```python
# tests/test_stress.py
import numpy as np
import torch
import pytest


class TestEdgeCases:
    """Test model behavior on unusual inputs."""

    def test_empty_string(self, sentiment_model):
        """Model should not crash on empty input."""
        result = predict_sentiment(sentiment_model, "")
        assert result["label"] in ("positive", "negative", "neutral")

    def test_very_long_input(self, sentiment_model):
        """Model should handle or gracefully truncate long input."""
        long_text = "This is great. " * 10000
        result = predict_sentiment(sentiment_model, long_text)
        assert result is not None

    def test_unicode_input(self, sentiment_model):
        """Model should not crash on Unicode/emoji input."""
        result = predict_sentiment(sentiment_model, "Great product! 🎉👍")
        assert result is not None

    def test_only_punctuation(self, sentiment_model):
        result = predict_sentiment(sentiment_model, "!?!?!?...")
        assert result is not None

    def test_numeric_only(self, sentiment_model):
        result = predict_sentiment(sentiment_model, "12345 67890")
        assert result is not None


class TestAdversarial:
    """Test model robustness to adversarial perturbations."""

    @pytest.mark.parametrize("original, adversarial", [
        ("This movie is great", "This movie is gr8"),
        ("Terrible product", "T3rribl3 pr0duct"),
        ("I love it", "I l0ve it"),
    ])
    def test_leetspeak_robustness(self, sentiment_model, original, adversarial):
        orig_pred = predict_sentiment(sentiment_model, original)
        adv_pred = predict_sentiment(sentiment_model, adversarial)
        assert orig_pred["label"] == adv_pred["label"]

    def test_character_insertion(self, sentiment_model):
        """Inserting invisible characters should not change prediction."""
        original = "This is a great product"
        perturbed = "This is a gre\u200bat product"   # zero-width space
        orig_pred = predict_sentiment(sentiment_model, original)
        pert_pred = predict_sentiment(sentiment_model, perturbed)
        assert orig_pred["label"] == pert_pred["label"]


class TestNumericalStress:
    """Stress tests for tabular/numerical models."""

    def test_all_zeros(self, tabular_model):
        zeros = np.zeros((1, 128))
        result = tabular_model.predict(zeros)
        assert not np.isnan(result).any()

    def test_extreme_values(self, tabular_model):
        extreme = np.full((1, 128), 1e10)
        result = tabular_model.predict(extreme)
        assert not np.isnan(result).any()
        assert not np.isinf(result).any()

    def test_nan_input_handling(self, tabular_model):
        """Model should either handle NaN or raise a clear error."""
        nan_input = np.array([[np.nan] * 128])
        try:
            result = tabular_model.predict(nan_input)
            assert not np.isnan(result).any(), "NaN propagated to output"
        except (ValueError, RuntimeError):
            pass   # Raising is acceptable — silent NaN propagation is not

    def test_large_batch(self, tabular_model):
        """Model should handle a large batch without OOM."""
        large_batch = np.random.randn(10000, 128).astype(np.float32)
        result = tabular_model.predict(large_batch)
        assert result.shape[0] == 10000

    @pytest.mark.parametrize("noise_level", [0.01, 0.1, 0.5])
    def test_gaussian_noise_robustness(self, tabular_model):
        """Small noise should not dramatically change predictions."""
        base = np.random.randn(100, 128).astype(np.float32)
        base_preds = tabular_model.predict(base)

        noise = np.random.randn(*base.shape).astype(np.float32) * 0.01
        noisy_preds = tabular_model.predict(base + noise)

        # Predictions should be close with tiny noise
        diff = np.abs(base_preds - noisy_preds).mean()
        assert diff < 0.1, f"Mean prediction difference: {diff:.4f}"
```

---

## Putting It Together: Model Test Suite

### Complete Test Organization

```
tests/
├── conftest.py               # Model fixtures, data loading
├── test_unit/                 # Unit tests (Chapter 1)
│   ├── test_preprocess.py
│   └── test_features.py
├── test_data/                 # Data tests (Chapter 2)
│   └── test_schema.py
├── test_model/                # Model behavioral tests
│   ├── test_mft.py            # Minimum functionality
│   ├── test_invariance.py     # INV tests
│   ├── test_directional.py    # DIR tests
│   ├── test_bias.py           # Fairness tests
│   └── test_stress.py         # Stress tests
└── test_integration/          # Integration tests (Chapter 4)
    └── test_pipeline.py
```

### CheckList Summary Matrix

| Capability | MFT | INV | DIR |
|---|---|---|---|
| **Vocabulary** | Basic sentiment words | Synonym swap | Add intensifier ("very") |
| **Negation** | "not good" → negative | — | Negate → flip sentiment |
| **Robustness** | — | Typos, casing | — |
| **Named Entities** | — | Name swap | — |
| **Fairness** | — | Gender/race swap | — |
| **Temporal** | "used to be good, now bad" → neg | — | Add recency qualifier |
| **Logic** | "good and good" → positive | — | "good but bad" → lower |

---

## Summary

| Concept | Key Takeaway |
|---|---|
| **Beyond accuracy** | Aggregate metrics hide specific failures; test capabilities, not averages |
| **CheckList framework** | Three test types: MFT (must-pass), INV (invariance), DIR (directional) |
| **Invariance tests** | Irrelevant changes (names, locations, typos) should not change output |
| **Directional tests** | Meaningful changes (negation, intensifiers) should change output predictably |
| **MFTs** | Obvious cases the model must get right — sanity checks |
| **Bias testing** | Demographic parity, equalized odds, counterfactual fairness |
| **Stress testing** | Edge cases (empty, long, Unicode), adversarial inputs, numerical extremes |
| **Test organization** | Separate MFT / INV / DIR / bias / stress into dedicated test files |

> **Rule of thumb:** If a human would find the test case trivially easy, and the model gets it wrong, that's a showstopper bug. MFTs catch these. If swapping a name changes the output, that's a fairness bug. INV tests catch these. Both should block deployment.

---

## Revision Questions

1. **Explain** the three CheckList test types (MFT, INV, DIR) with a concrete example for each. Why is this more informative than a single accuracy number?

2. **Design** a set of invariance tests for a resume-screening model. What attributes should be invariant, and what are the risks if they are not?

3. **A sentiment model** achieves 93% accuracy but fails on "The food was not bad" (predicts negative). Explain which CheckList test type catches this, and write 5 additional test cases for the same capability.

4. **Compare** demographic parity and equalized odds as fairness metrics. Give an example where satisfying one violates the other. Which is more appropriate for a loan approval model?

5. **Design** a stress testing suite for an image classification model deployed as an API. Include edge cases, adversarial perturbations, load tests, and numerical boundary conditions.

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Data Testing](./02-data-testing.md) | [📂 ML Testing](./README.md) | [Integration Testing →](./04-integration-testing.md) |
