# 🧪 Unit Testing ML Code

> **Unit 10 · ML Testing · Chapter 1**
>
> How to apply traditional unit testing to ML code — pytest for ML projects, testing preprocessing functions, verifying prediction shapes, testing feature engineering logic, mocking models for fast tests, and building reusable test fixtures.

---

## Table of Contents

1. [Chapter Overview](#chapter-overview)
2. [Why Unit Test ML Code?](#why-unit-test-ml-code)
3. [pytest Fundamentals for ML](#pytest-fundamentals-for-ml)
4. [Testing Preprocessing Functions](#testing-preprocessing-functions)
5. [Testing Model Prediction Shapes](#testing-model-prediction-shapes)
6. [Testing Feature Engineering](#testing-feature-engineering)
7. [Mocking Models](#mocking-models)
8. [Fixtures for Test Data](#fixtures-for-test-data)
9. [Structuring an ML Test Suite](#structuring-an-ml-test-suite)
10. [Summary](#summary)
11. [Revision Questions](#revision-questions)
12. [Navigation](#navigation)

---

## Chapter Overview

ML code is notoriously under-tested. Data scientists prototype in notebooks, push code to production, and hope for the best. But ML systems contain **ordinary software** — data transformations, feature pipelines, API handlers — that can and should be tested with the same rigour as any web service. This chapter covers how to use **pytest** to build a reliable test suite for ML code: testing data transformations for correctness, verifying tensor/array shapes through the model graph, testing feature engineering logic with known inputs, mocking expensive model calls, and creating reusable fixtures that provide deterministic test data.

```
Without Tests                          With Unit Tests
┌──────────────────────────┐           ┌──────────────────────────┐
│  "It works on my laptop" │           │  pytest tests/           │
│                          │           │  ├── test_preprocess.py  │
│  Push to prod            │           │  ├── test_features.py    │
│  ┌─────────┐             │           │  ├── test_model.py       │
│  │ 🔥 Bug  │ ← found    │           │  └── test_pipeline.py    │
│  │ in prod │   at 3 AM   │           │                          │
│  └─────────┘             │           │  ✅ 47 passed in 2.3s    │
│  Rollback + hotfix       │           │  Bugs caught before merge│
└──────────────────────────┘           └──────────────────────────┘
```

---

## Why Unit Test ML Code?

### The Testing Gap

Most ML teams test models (accuracy, F1) but **not** the code around them. Yet the majority of production ML bugs come from:

| Bug Source | Example | Caught by Unit Tests? |
|---|---|---|
| **Data transformation** | Wrong column scaling, off-by-one | ✅ Yes |
| **Feature engineering** | Missing interaction term, wrong aggregation | ✅ Yes |
| **Shape mismatches** | Model expects (batch, 128) but gets (batch, 129) | ✅ Yes |
| **Edge cases** | NaN input, empty batch, single-row DataFrame | ✅ Yes |
| **Model accuracy regression** | F1 dropped from 0.92 → 0.88 | ❌ Integration test |
| **Data distribution shift** | New category in production data | ❌ Data test |

### What to Test vs What Not to Test

```
┌─────────────────────────────────────────────────────┐
│                Unit Test (this chapter)              │
│  • Pure functions: transform(x) → y                 │
│  • Shape contracts: model(input) → (batch, classes)  │
│  • Feature logic: extract_features(row) → vector    │
│  • Edge cases: nulls, empty, large values            │
├─────────────────────────────────────────────────────┤
│           DO NOT unit test                           │
│  • Whether the model "learned" correctly             │
│  • Hyperparameter choices                            │
│  • Training convergence                              │
└─────────────────────────────────────────────────────┘
```

---

## pytest Fundamentals for ML

### Project Layout

```
ml-project/
├── src/
│   ├── preprocess.py
│   ├── features.py
│   ├── model.py
│   └── pipeline.py
├── tests/
│   ├── conftest.py            # Shared fixtures
│   ├── test_preprocess.py
│   ├── test_features.py
│   ├── test_model.py
│   └── test_pipeline.py
├── pyproject.toml
└── pytest.ini
```

### Minimal pytest Configuration

```ini
# pytest.ini
[pytest]
testpaths = tests
markers =
    slow: marks tests as slow (model loading, GPU)
    gpu: marks tests requiring GPU
addopts = -v --tb=short
```

### Running Tests

```bash
# Run all tests
pytest

# Run a single file
pytest tests/test_preprocess.py

# Run tests matching a keyword
pytest -k "test_normalize"

# Skip slow tests (e.g., in CI fast lane)
pytest -m "not slow"

# Run with coverage
pytest --cov=src --cov-report=term-missing
```

### Key pytest Features for ML

```python
# tests/test_basics.py
import pytest
import numpy as np

# --- Parametrize: test many inputs in one function ---
@pytest.mark.parametrize("input_val, expected", [
    (0.0, 0.0),
    (1.0, 1.0),
    (-1.0, 0.0),      # ReLU clamps negatives
])
def test_relu(input_val, expected):
    result = max(0, input_val)
    assert result == expected


# --- Approximate equality for floats ---
def test_softmax_sums_to_one():
    logits = np.array([2.0, 1.0, 0.1])
    probs = np.exp(logits) / np.exp(logits).sum()
    assert probs.sum() == pytest.approx(1.0, abs=1e-6)


# --- Expected failures ---
@pytest.mark.xfail(reason="Known issue with empty input")
def test_empty_input():
    from src.preprocess import normalize
    normalize(np.array([]))
```

---

## Testing Preprocessing Functions

Preprocessing is the **highest-value** target for unit tests — it's pure functions operating on data.

### Example: Testing a Normalizer

```python
# src/preprocess.py
import numpy as np
import pandas as pd

def normalize(values: np.ndarray) -> np.ndarray:
    """Min-max normalize to [0, 1]."""
    min_val = values.min()
    max_val = values.max()
    if max_val == min_val:
        return np.zeros_like(values)
    return (values - min_val) / (max_val - min_val)

def clean_text(text: str) -> str:
    """Lowercase, strip whitespace, remove special chars."""
    import re
    text = text.lower().strip()
    text = re.sub(r"[^a-z0-9\s]", "", text)
    return text

def encode_categorical(df: pd.DataFrame, column: str) -> pd.DataFrame:
    """One-hot encode a categorical column."""
    dummies = pd.get_dummies(df[column], prefix=column)
    return pd.concat([df.drop(columns=[column]), dummies], axis=1)
```

### Tests

```python
# tests/test_preprocess.py
import numpy as np
import pandas as pd
import pytest
from src.preprocess import normalize, clean_text, encode_categorical


class TestNormalize:
    def test_basic_normalization(self):
        arr = np.array([1.0, 2.0, 3.0, 4.0, 5.0])
        result = normalize(arr)
        assert result.min() == pytest.approx(0.0)
        assert result.max() == pytest.approx(1.0)

    def test_already_normalized(self):
        arr = np.array([0.0, 0.5, 1.0])
        result = normalize(arr)
        np.testing.assert_array_almost_equal(result, arr)

    def test_constant_values(self):
        """All same values should return zeros, not NaN."""
        arr = np.array([5.0, 5.0, 5.0])
        result = normalize(arr)
        assert not np.any(np.isnan(result))
        np.testing.assert_array_equal(result, [0.0, 0.0, 0.0])

    def test_negative_values(self):
        arr = np.array([-10.0, 0.0, 10.0])
        result = normalize(arr)
        assert result[0] == pytest.approx(0.0)
        assert result[2] == pytest.approx(1.0)

    def test_output_shape_matches_input(self):
        arr = np.random.randn(100)
        result = normalize(arr)
        assert result.shape == arr.shape


class TestCleanText:
    @pytest.mark.parametrize("raw, expected", [
        ("Hello World!", "hello world"),
        ("  SPACES  ", "spaces"),
        ("Price: $100", "price 100"),
        ("café", "caf"),
        ("", ""),
    ])
    def test_clean_text(self, raw, expected):
        assert clean_text(raw) == expected


class TestEncodeCategorical:
    def test_one_hot_columns_created(self):
        df = pd.DataFrame({"color": ["red", "blue", "red"]})
        result = encode_categorical(df, "color")
        assert "color_red" in result.columns
        assert "color_blue" in result.columns
        assert "color" not in result.columns

    def test_row_count_preserved(self):
        df = pd.DataFrame({"color": ["red", "blue", "green"]})
        result = encode_categorical(df, "color")
        assert len(result) == 3
```

---

## Testing Model Prediction Shapes

You can't unit test whether a model is "correct," but you **can** test that it produces the right shapes, types, and value ranges.

### Shape Contract Pattern

```
Input Contract                    Output Contract
┌─────────────────────┐          ┌─────────────────────────┐
│ shape: (batch, 128)  │   ───►  │ shape: (batch, num_cls) │
│ dtype: float32       │         │ dtype: float32           │
│ range: [-1, 1]       │         │ range: [0, 1] (softmax)  │
│ no NaN               │         │ sum per row ≈ 1.0        │
└─────────────────────┘          └─────────────────────────┘
```

### Tests

```python
# tests/test_model.py
import numpy as np
import pytest
import torch
import torch.nn as nn


class SimpleClassifier(nn.Module):
    def __init__(self, input_dim=128, num_classes=10):
        super().__init__()
        self.fc1 = nn.Linear(input_dim, 64)
        self.fc2 = nn.Linear(64, num_classes)

    def forward(self, x):
        x = torch.relu(self.fc1(x))
        return torch.softmax(self.fc2(x), dim=-1)


@pytest.fixture
def model():
    m = SimpleClassifier(input_dim=128, num_classes=10)
    m.eval()
    return m


class TestModelShapes:
    def test_output_shape(self, model):
        batch = torch.randn(16, 128)
        output = model(batch)
        assert output.shape == (16, 10)

    def test_single_sample(self, model):
        """Model should handle batch_size=1."""
        single = torch.randn(1, 128)
        output = model(single)
        assert output.shape == (1, 10)

    def test_output_is_probability(self, model):
        batch = torch.randn(8, 128)
        output = model(batch)
        # Each row sums to 1.0
        row_sums = output.sum(dim=1)
        assert torch.allclose(row_sums, torch.ones(8), atol=1e-5)

    def test_output_non_negative(self, model):
        batch = torch.randn(8, 128)
        output = model(batch)
        assert (output >= 0).all()

    def test_no_nan_in_output(self, model):
        batch = torch.randn(8, 128)
        output = model(batch)
        assert not torch.isnan(output).any()

    @pytest.mark.parametrize("batch_size", [1, 8, 32, 128])
    def test_variable_batch_size(self, model, batch_size):
        batch = torch.randn(batch_size, 128)
        output = model(batch)
        assert output.shape == (batch_size, 10)
```

---

## Testing Feature Engineering

Feature engineering functions are pure logic — ideal for unit testing.

### Example: Feature Extraction

```python
# src/features.py
import pandas as pd
import numpy as np
from datetime import datetime

def extract_time_features(df: pd.DataFrame, col: str) -> pd.DataFrame:
    """Extract hour, day-of-week, is_weekend from a datetime column."""
    df = df.copy()
    dt = pd.to_datetime(df[col])
    df[f"{col}_hour"] = dt.dt.hour
    df[f"{col}_dow"] = dt.dt.dayofweek       # 0=Monday
    df[f"{col}_is_weekend"] = (dt.dt.dayofweek >= 5).astype(int)
    return df

def compute_interaction(df: pd.DataFrame, col_a: str, col_b: str) -> pd.Series:
    """Multiply two columns as an interaction feature."""
    return df[col_a] * df[col_b]

def bin_age(age: int) -> str:
    """Bin an age into a category."""
    if age < 0:
        raise ValueError("Age cannot be negative")
    if age < 18:
        return "minor"
    elif age < 35:
        return "young_adult"
    elif age < 60:
        return "adult"
    else:
        return "senior"
```

### Tests

```python
# tests/test_features.py
import pandas as pd
import numpy as np
import pytest
from src.features import extract_time_features, compute_interaction, bin_age


class TestTimeFeatures:
    @pytest.fixture
    def sample_df(self):
        return pd.DataFrame({
            "event_time": [
                "2024-01-15 08:30:00",   # Monday morning
                "2024-01-20 22:00:00",   # Saturday night
                "2024-01-21 14:00:00",   # Sunday afternoon
            ]
        })

    def test_columns_created(self, sample_df):
        result = extract_time_features(sample_df, "event_time")
        assert "event_time_hour" in result.columns
        assert "event_time_dow" in result.columns
        assert "event_time_is_weekend" in result.columns

    def test_hour_values(self, sample_df):
        result = extract_time_features(sample_df, "event_time")
        assert list(result["event_time_hour"]) == [8, 22, 14]

    def test_weekend_detection(self, sample_df):
        result = extract_time_features(sample_df, "event_time")
        assert list(result["event_time_is_weekend"]) == [0, 1, 1]

    def test_original_column_preserved(self, sample_df):
        result = extract_time_features(sample_df, "event_time")
        assert "event_time" in result.columns


class TestInteraction:
    def test_basic_interaction(self):
        df = pd.DataFrame({"a": [1, 2, 3], "b": [4, 5, 6]})
        result = compute_interaction(df, "a", "b")
        assert list(result) == [4, 10, 18]

    def test_with_zeros(self):
        df = pd.DataFrame({"a": [0, 1], "b": [100, 0]})
        result = compute_interaction(df, "a", "b")
        assert list(result) == [0, 0]


class TestBinAge:
    @pytest.mark.parametrize("age, expected", [
        (5, "minor"),
        (17, "minor"),
        (18, "young_adult"),
        (34, "young_adult"),
        (35, "adult"),
        (59, "adult"),
        (60, "senior"),
        (90, "senior"),
    ])
    def test_age_bins(self, age, expected):
        assert bin_age(age) == expected

    def test_negative_age_raises(self):
        with pytest.raises(ValueError, match="negative"):
            bin_age(-1)
```

---

## Mocking Models

Loading a real model in every test is **slow** (seconds to minutes). Mock the model to keep tests fast.

### When to Mock vs When to Load

```
┌────────────────────────────────┬──────────────────────────────┐
│        Mock the Model          │      Load the Real Model     │
├────────────────────────────────┼──────────────────────────────┤
│ Testing code around the model  │ Testing model behavior       │
│ Preprocessing → model → post   │ Shape contracts (small model)│
│ API handler logic              │ Smoke test: does it load?    │
│ Error handling                 │ Integration tests            │
│ Fast: < 1 second              │ Slow: 5-60 seconds           │
│ Run on every commit            │ Run nightly / before deploy  │
└────────────────────────────────┴──────────────────────────────┘
```

### Mocking with unittest.mock

```python
# tests/test_with_mocks.py
import numpy as np
import pytest
from unittest.mock import MagicMock, patch


# --- Example: function that uses a model ---
# src/pipeline.py
def predict_and_postprocess(model, features: np.ndarray) -> list[str]:
    """Run prediction and map to labels."""
    labels = ["cat", "dog", "bird"]
    probs = model.predict(features)            # expensive call
    indices = probs.argmax(axis=1)
    return [labels[i] for i in indices]


class TestPredictAndPostprocess:
    def test_returns_labels(self):
        mock_model = MagicMock()
        mock_model.predict.return_value = np.array([
            [0.1, 0.8, 0.1],     # dog
            [0.9, 0.05, 0.05],   # cat
        ])

        result = predict_and_postprocess(mock_model, np.zeros((2, 128)))
        assert result == ["dog", "cat"]
        mock_model.predict.assert_called_once()

    def test_handles_single_sample(self):
        mock_model = MagicMock()
        mock_model.predict.return_value = np.array([[0.0, 0.0, 1.0]])

        result = predict_and_postprocess(mock_model, np.zeros((1, 128)))
        assert result == ["bird"]


# --- Mocking an expensive model load ---
class TestWithPatchedModel:
    @patch("src.pipeline.load_model")
    def test_pipeline_uses_model(self, mock_load):
        fake_model = MagicMock()
        fake_model.predict.return_value = np.array([[0.5, 0.3, 0.2]])
        mock_load.return_value = fake_model

        from src.pipeline import predict_and_postprocess
        result = predict_and_postprocess(fake_model, np.zeros((1, 128)))
        assert result == ["cat"]
```

---

## Fixtures for Test Data

Fixtures create **deterministic, reusable test data** that keeps tests isolated and reproducible.

### conftest.py — Shared Fixtures

```python
# tests/conftest.py
import numpy as np
import pandas as pd
import pytest
import torch


@pytest.fixture
def seed():
    """Fix random seeds for reproducibility."""
    np.random.seed(42)
    torch.manual_seed(42)


@pytest.fixture
def sample_tabular_data():
    """Realistic tabular dataset for testing."""
    np.random.seed(42)
    n = 100
    return pd.DataFrame({
        "age": np.random.randint(18, 80, n),
        "income": np.random.lognormal(10, 1, n),
        "category": np.random.choice(["A", "B", "C"], n),
        "timestamp": pd.date_range("2024-01-01", periods=n, freq="h"),
    })


@pytest.fixture
def sample_batch():
    """A batch of feature vectors."""
    torch.manual_seed(42)
    return torch.randn(16, 128)


@pytest.fixture
def sample_image_batch():
    """Batch of fake images (batch, channels, height, width)."""
    torch.manual_seed(42)
    return torch.randn(8, 3, 224, 224)


@pytest.fixture
def sample_with_nulls():
    """DataFrame with deliberate missing values for edge-case tests."""
    return pd.DataFrame({
        "age": [25, None, 35, 40, None],
        "income": [50000, 60000, None, 80000, 90000],
        "category": ["A", "B", None, "A", "C"],
    })


# --- Factory fixture for flexible test data ---
@pytest.fixture
def make_dataframe():
    """Factory fixture: call with custom args to build DataFrames."""
    def _make(n_rows=50, include_nulls=False):
        np.random.seed(42)
        df = pd.DataFrame({
            "feature_1": np.random.randn(n_rows),
            "feature_2": np.random.randn(n_rows),
            "label": np.random.randint(0, 2, n_rows),
        })
        if include_nulls:
            mask = np.random.random(n_rows) < 0.1
            df.loc[mask, "feature_1"] = np.nan
        return df
    return _make
```

### Using Fixtures in Tests

```python
# tests/test_with_fixtures.py
def test_tabular_data_shape(sample_tabular_data):
    assert sample_tabular_data.shape == (100, 4)
    assert "age" in sample_tabular_data.columns

def test_batch_shape(sample_batch):
    assert sample_batch.shape == (16, 128)

def test_factory_fixture(make_dataframe):
    small = make_dataframe(n_rows=10)
    assert len(small) == 10

    with_nulls = make_dataframe(n_rows=100, include_nulls=True)
    assert with_nulls["feature_1"].isna().sum() > 0

def test_null_handling(sample_with_nulls):
    assert sample_with_nulls["age"].isna().sum() == 2
```

---

## Structuring an ML Test Suite

### Test Pyramid for ML

```
                    ╱╲
                   ╱  ╲
                  ╱ E2E╲            Few: full pipeline on real data
                 ╱______╲           (slow, run nightly)
                ╱        ╲
               ╱Integration╲       Some: model + preprocessing
              ╱______________╲     (medium, run before merge)
             ╱                ╲
            ╱   Unit Tests     ╲   Many: pure functions, shapes
           ╱____________________╲  (fast, run on every commit)

    Speed:    Slow ◄──────────────► Fast
    Count:    Few  ◄──────────────► Many
    Scope:    Broad ◄─────────────► Narrow
```

### Marker-Based Test Organization

```python
# Run fast tests on every commit
# pytest -m "not slow and not gpu"

@pytest.mark.slow
def test_full_model_load():
    """Takes 30 seconds — skip in CI fast lane."""
    ...

@pytest.mark.gpu
def test_gpu_inference():
    """Requires GPU — skip on CPU runners."""
    ...

# No marker = fast test, always runs
def test_normalize_basic():
    ...
```

### CI Configuration Tiers

| Tier | When | Markers | Duration |
|---|---|---|---|
| **Fast lane** | Every commit / PR | `not slow and not gpu` | < 30 seconds |
| **Full suite** | Before merge | `not gpu` | 2–5 minutes |
| **Nightly** | Scheduled daily | All tests | 10–60 minutes |
| **GPU suite** | Weekly / pre-deploy | `gpu` | Variable |

---

## Summary

| Concept | Key Takeaway |
|---|---|
| **Why unit test ML** | Most production bugs are in data/feature code, not model weights |
| **pytest for ML** | Use `parametrize`, `approx`, fixtures, and markers (`slow`, `gpu`) |
| **Preprocessing tests** | Test normalization, encoding, cleaning with known input→output pairs |
| **Shape tests** | Verify output shapes, dtypes, value ranges — don't test accuracy |
| **Feature engineering** | Pure functions → parametrized tests with edge cases |
| **Mocking models** | Use `MagicMock` to avoid loading real models in fast tests |
| **Fixtures** | Shared via `conftest.py`; use factory fixtures for flexibility |
| **Test pyramid** | Many fast unit tests, fewer integration, rare E2E |

> **Rule of thumb:** If a function takes data in and returns data out without training a model, it should have a unit test. These tests are fast, cheap, and catch the bugs that actually hit production.

---

## Revision Questions

1. **Explain** why testing preprocessing functions provides more value per test than testing model accuracy. Give two examples of preprocessing bugs that unit tests would catch.

2. **Design** a set of pytest tests for a function `impute_missing(df, strategy="median")` that fills NaN values. Include edge cases like all-NaN columns and single-row DataFrames.

3. **Compare** mocking a model with `MagicMock` vs loading a small real model in tests. When is each approach appropriate? What are the risks of over-mocking?

4. **A team** has 200 unit tests that take 8 minutes because every test loads a PyTorch model. Describe how to restructure the suite using fixtures and mocking to bring it under 30 seconds.

5. **Write** a `conftest.py` factory fixture that generates synthetic classification datasets with configurable number of features, samples, class balance, and missing-value rate. Explain how parametrize would use this fixture.

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Scaling Strategies](../09-Infrastructure/05-scaling-strategies.md) | [📂 ML Testing](./README.md) | [Data Testing →](./02-data-testing.md) |
