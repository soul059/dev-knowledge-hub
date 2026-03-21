# 🔗 Integration Testing

> **Unit 10 · ML Testing · Chapter 4**
>
> How to test ML systems end-to-end — testing the full pipeline from raw data to predictions, API contract testing, verifying model + preprocessing compatibility, staging environment tests, and catch failures that unit tests miss.

---

## Table of Contents

1. [Chapter Overview](#chapter-overview)
2. [Why Integration Testing for ML?](#why-integration-testing-for-ml)
3. [Testing the Full ML Pipeline](#testing-the-full-ml-pipeline)
4. [API Contract Testing](#api-contract-testing)
5. [Testing Model + Preprocessing Together](#testing-model--preprocessing-together)
6. [Staging Environment Tests](#staging-environment-tests)
7. [Test Data Management](#test-data-management)
8. [Integration Test Patterns](#integration-test-patterns)
9. [Summary](#summary)
10. [Revision Questions](#revision-questions)
11. [Navigation](#navigation)

---

## Chapter Overview

Unit tests verify that individual functions work correctly. But **ML systems fail at the seams** — where preprocessing meets the model, where the model meets the API, where the API meets the client. A normalizer can pass all its unit tests, a model can pass all shape tests, and the system still breaks because the normalizer was fit on training data but applied differently in production. **Integration testing** verifies that the components work together as a system: raw data flows through preprocessing, feature engineering, model inference, and postprocessing to produce correct predictions. This chapter covers end-to-end pipeline tests, API contract testing, model + preprocessing compatibility, and staging environment validation.

```
Unit Tests                        Integration Tests
┌───────────┐                     ┌──────────────────────────────────────┐
│ preprocess│  ✅                  │  Raw Data                           │
│ ───────── │                     │    │                                 │
│ features  │  ✅                  │    ▼                                 │
│ ───────── │                     │  Preprocess ──► Features ──► Model   │
│ model     │  ✅                  │    │              │            │     │
│ ───────── │                     │    └──────────────┴────────────┘     │
│ postproc  │  ✅                  │         All connected? ✅ or ❌       │
└───────────┘                     └──────────────────────────────────────┘
Each passes alone                 Tests the connections between them
```

---

## Why Integration Testing for ML?

### Bugs That Unit Tests Miss

| Bug Type | Cause | Unit Tests | Integration Tests |
|---|---|---|---|
| **Feature order mismatch** | Training uses [age, income], serving uses [income, age] | ❌ | ✅ |
| **Scaler not fitted** | Preprocessing in serving uses unfitted scaler | ❌ | ✅ |
| **Missing feature column** | Feature engineering drops column before model | ❌ | ✅ |
| **Type mismatch** | Preprocessing returns float64, model expects float32 | ❌ | ✅ |
| **API serialization** | JSON conversion drops numpy int64 | ❌ | ✅ |
| **Version mismatch** | Model v2 expects 15 features, preprocessor v1 produces 12 | ❌ | ✅ |

### The Integration Testing Gap

```
                    What teams test:
                    ┌──────────────────────────┐
                    │ ████████████████  93%     │  Model accuracy (offline)
                    │ ████████████     72%     │  Unit tests
                    │ ███████          45%     │  API endpoint tests
                    │ ████             22%     │  Pipeline integration ← Gap
                    │ ██               11%     │  Staging validation
                    └──────────────────────────┘

    Most ML bugs hit production because integration
    testing is the least-practiced discipline.
```

---

## Testing the Full ML Pipeline

### End-to-End Pipeline Test

```
┌────────────────────────────────────────────────────────────────┐
│                    End-to-End Pipeline Test                     │
│                                                                │
│  Known Input                Expected Output                    │
│  ┌──────────┐              ┌──────────────┐                    │
│  │ Raw CSV  │───►Pipeline───►│ Predictions │───► Assert        │
│  │ (5 rows) │              │ (5 labels)   │     correct       │
│  └──────────┘              └──────────────┘                    │
│                                                                │
│  Verifies: data loading → cleaning → feature engineering       │
│            → model inference → postprocessing                  │
└────────────────────────────────────────────────────────────────┘
```

### Implementation

```python
# tests/test_pipeline_e2e.py
import pandas as pd
import numpy as np
import pytest
import json
import os


@pytest.fixture(scope="module")
def pipeline():
    """Load the full pipeline once for all tests in this module."""
    from src.pipeline import MLPipeline
    return MLPipeline.load("models/pipeline_v2/")


@pytest.fixture
def golden_data():
    """Known input-output pairs saved during model validation."""
    return pd.read_csv("tests/fixtures/golden_dataset.csv")


class TestEndToEnd:
    """Test the complete pipeline: raw data → predictions."""

    def test_pipeline_produces_predictions(self, pipeline, golden_data):
        """Pipeline should produce one prediction per input row."""
        inputs = golden_data.drop(columns=["expected_label"])
        predictions = pipeline.predict(inputs)
        assert len(predictions) == len(golden_data)

    def test_golden_dataset_accuracy(self, pipeline, golden_data):
        """Pipeline should match known-good outputs on golden dataset."""
        inputs = golden_data.drop(columns=["expected_label"])
        predictions = pipeline.predict(inputs)
        accuracy = (predictions == golden_data["expected_label"]).mean()
        assert accuracy >= 0.95, \
            f"Golden dataset accuracy {accuracy:.2%} below 95% threshold"

    def test_single_row_prediction(self, pipeline):
        """Pipeline should handle single-row input."""
        single = pd.DataFrame([{
            "age": 35, "income": 75000, "category": "B",
            "num_purchases": 12, "days_since_last": 5
        }])
        prediction = pipeline.predict(single)
        assert len(prediction) == 1
        assert prediction[0] in [0, 1]   # binary classification

    def test_output_types(self, pipeline, golden_data):
        """Output predictions should be the correct type."""
        inputs = golden_data.drop(columns=["expected_label"])
        predictions = pipeline.predict(inputs)
        assert isinstance(predictions, np.ndarray)
        assert predictions.dtype in [np.int64, np.int32, np.float64]

    def test_probability_output(self, pipeline, golden_data):
        """Probability outputs should be valid probabilities."""
        inputs = golden_data.drop(columns=["expected_label"])
        probas = pipeline.predict_proba(inputs)
        assert probas.shape == (len(golden_data), 2)
        assert (probas >= 0).all() and (probas <= 1).all()
        row_sums = probas.sum(axis=1)
        np.testing.assert_allclose(row_sums, 1.0, atol=1e-6)


class TestPipelineComponents:
    """Test that pipeline components are wired correctly."""

    def test_feature_count_matches_model(self, pipeline):
        """Preprocessor output columns must match model input features."""
        preprocessor = pipeline.preprocessor
        model = pipeline.model

        # Get expected feature count from model
        expected_features = model.n_features_in_
        sample = pd.DataFrame([{
            "age": 30, "income": 50000, "category": "A",
            "num_purchases": 5, "days_since_last": 10
        }])
        actual_features = preprocessor.transform(sample).shape[1]
        assert actual_features == expected_features, \
            f"Preprocessor outputs {actual_features} features, " \
            f"model expects {expected_features}"

    def test_feature_order_consistency(self, pipeline):
        """Feature order must be consistent between training and serving."""
        expected_order = pipeline.feature_names
        sample = pd.DataFrame([{
            "age": 30, "income": 50000, "category": "A",
            "num_purchases": 5, "days_since_last": 10
        }])
        actual_columns = list(pipeline.preprocessor.get_feature_names_out())
        assert actual_columns == expected_order, \
            f"Feature order mismatch:\n" \
            f"Expected: {expected_order[:5]}...\n" \
            f"Actual:   {actual_columns[:5]}..."

    def test_scaler_is_fitted(self, pipeline):
        """Scaler must be fitted (not default)."""
        scaler = pipeline.preprocessor.named_steps.get("scaler")
        if scaler:
            assert hasattr(scaler, "mean_"), \
                "Scaler is not fitted — will produce wrong results"
```

---

## API Contract Testing

### What to Test

```
Client                    API Server                   Model
  │                          │                            │
  │   POST /predict          │                            │
  │   {"age": 35, ...}       │                            │
  │  ─────────────────────►  │                            │
  │                          │   preprocess + predict      │
  │                          │  ─────────────────────────► │
  │                          │                            │
  │                          │   {"label": 1, "prob": 0.8} │
  │   200 OK                 │  ◄───────────────────────── │
  │   {"label": 1, ...}      │                            │
  │  ◄─────────────────────  │                            │
  │                          │                            │

  Contract tests verify:
  ✅ Request schema is accepted
  ✅ Response schema is correct
  ✅ Error responses are informative
  ✅ Edge cases don't crash the server
```

### Implementation with FastAPI TestClient

```python
# src/app.py
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field
import numpy as np

app = FastAPI()

class PredictionRequest(BaseModel):
    age: int = Field(ge=0, le=120)
    income: float = Field(gt=0)
    category: str = Field(pattern="^[A-C]$")
    num_purchases: int = Field(ge=0)
    days_since_last: int = Field(ge=0)

class PredictionResponse(BaseModel):
    label: int
    probability: float = Field(ge=0, le=1)
    model_version: str

@app.post("/predict", response_model=PredictionResponse)
def predict(request: PredictionRequest):
    from src.pipeline import get_pipeline
    pipeline = get_pipeline()
    features = request.model_dump()
    prediction = pipeline.predict_single(features)
    return PredictionResponse(
        label=int(prediction["label"]),
        probability=float(prediction["probability"]),
        model_version=pipeline.version,
    )
```

```python
# tests/test_api_contract.py
import pytest
from fastapi.testclient import TestClient
from src.app import app


@pytest.fixture
def client():
    return TestClient(app)


class TestAPIContract:
    """Test the API contract — request/response schemas."""

    def test_valid_request_returns_200(self, client):
        response = client.post("/predict", json={
            "age": 35,
            "income": 75000.0,
            "category": "B",
            "num_purchases": 12,
            "days_since_last": 5,
        })
        assert response.status_code == 200

    def test_response_schema(self, client):
        response = client.post("/predict", json={
            "age": 35,
            "income": 75000.0,
            "category": "B",
            "num_purchases": 12,
            "days_since_last": 5,
        })
        data = response.json()
        assert "label" in data
        assert "probability" in data
        assert "model_version" in data
        assert isinstance(data["label"], int)
        assert 0 <= data["probability"] <= 1

    def test_missing_field_returns_422(self, client):
        response = client.post("/predict", json={
            "age": 35,
            # Missing required fields
        })
        assert response.status_code == 422

    def test_invalid_age_returns_422(self, client):
        response = client.post("/predict", json={
            "age": -5,
            "income": 75000.0,
            "category": "B",
            "num_purchases": 12,
            "days_since_last": 5,
        })
        assert response.status_code == 422

    def test_invalid_category_returns_422(self, client):
        response = client.post("/predict", json={
            "age": 35,
            "income": 75000.0,
            "category": "Z",   # Not in [A, B, C]
            "num_purchases": 12,
            "days_since_last": 5,
        })
        assert response.status_code == 422

    def test_batch_prediction_endpoint(self, client):
        response = client.post("/predict/batch", json={
            "instances": [
                {"age": 25, "income": 40000, "category": "A",
                 "num_purchases": 3, "days_since_last": 20},
                {"age": 60, "income": 120000, "category": "C",
                 "num_purchases": 50, "days_since_last": 1},
            ]
        })
        assert response.status_code == 200
        data = response.json()
        assert len(data["predictions"]) == 2


class TestAPIEdgeCases:
    """Test API behavior on edge cases."""

    def test_boundary_age_values(self, client):
        for age in [0, 1, 119, 120]:
            response = client.post("/predict", json={
                "age": age, "income": 50000, "category": "A",
                "num_purchases": 5, "days_since_last": 10,
            })
            assert response.status_code == 200, \
                f"Age {age} should be valid"

    def test_large_income(self, client):
        response = client.post("/predict", json={
            "age": 35, "income": 10_000_000, "category": "A",
            "num_purchases": 5, "days_since_last": 10,
        })
        assert response.status_code == 200

    def test_concurrent_requests(self, client):
        """Multiple rapid requests should all succeed."""
        import concurrent.futures
        payload = {
            "age": 35, "income": 75000, "category": "B",
            "num_purchases": 12, "days_since_last": 5,
        }
        with concurrent.futures.ThreadPoolExecutor(max_workers=10) as executor:
            futures = [
                executor.submit(client.post, "/predict", json=payload)
                for _ in range(20)
            ]
            results = [f.result() for f in futures]
        assert all(r.status_code == 200 for r in results)
```

---

## Testing Model + Preprocessing Together

### The Compatibility Problem

```
Training Time                     Serving Time
┌──────────────────┐             ┌──────────────────┐
│  fit_transform()  │             │  transform()     │
│  on training data │             │  on new data     │
│                  │             │                  │
│  Scaler learns   │             │  Uses saved       │
│  mean=50, std=10 │             │  mean & std       │
│                  │             │                  │
│  Encoder learns  │             │  Must handle      │
│  categories A,B,C│             │  unknown "D"      │
└──────────────────┘             └──────────────────┘
         │                                │
         └──── Must be IDENTICAL ─────────┘

  Integration test catches when they diverge.
```

### Implementation

```python
# tests/test_model_preprocess.py
import pandas as pd
import numpy as np
import pytest
import pickle
import joblib


@pytest.fixture(scope="module")
def training_artifacts():
    """Load artifacts saved during training."""
    return {
        "preprocessor": joblib.load("models/preprocessor.joblib"),
        "model": joblib.load("models/model.joblib"),
        "feature_names": joblib.load("models/feature_names.joblib"),
        "train_sample": pd.read_parquet("tests/fixtures/train_sample.parquet"),
    }


class TestPreprocessorModelCompat:
    """Test that preprocessor and model are compatible."""

    def test_preprocessor_output_feeds_model(self, training_artifacts):
        """Transform output should be accepted by model.predict()."""
        preprocessor = training_artifacts["preprocessor"]
        model = training_artifacts["model"]
        sample = training_artifacts["train_sample"].head(10)

        transformed = preprocessor.transform(sample)
        predictions = model.predict(transformed)  # Should not raise
        assert len(predictions) == 10

    def test_feature_names_match(self, training_artifacts):
        """Feature names from preprocessor must match model expectations."""
        preprocessor = training_artifacts["preprocessor"]
        model = training_artifacts["model"]
        expected = training_artifacts["feature_names"]

        sample = training_artifacts["train_sample"].head(1)
        transformed = preprocessor.transform(sample)

        if hasattr(preprocessor, "get_feature_names_out"):
            actual_names = list(preprocessor.get_feature_names_out())
            assert actual_names == expected, \
                f"Feature name mismatch"

        if hasattr(model, "n_features_in_"):
            assert transformed.shape[1] == model.n_features_in_

    def test_dtype_compatibility(self, training_artifacts):
        """Preprocessor output dtype must be compatible with model."""
        preprocessor = training_artifacts["preprocessor"]
        sample = training_artifacts["train_sample"].head(5)
        transformed = preprocessor.transform(sample)
        assert transformed.dtype in [np.float32, np.float64], \
            f"Unexpected dtype: {transformed.dtype}"

    def test_handles_unseen_category(self, training_artifacts):
        """Preprocessor should handle unseen categories gracefully."""
        preprocessor = training_artifacts["preprocessor"]
        model = training_artifacts["model"]

        sample = training_artifacts["train_sample"].head(1).copy()
        # Inject unseen category
        cat_cols = sample.select_dtypes(include=["object"]).columns
        if len(cat_cols) > 0:
            sample[cat_cols[0]] = "NEVER_SEEN_CATEGORY"
            try:
                transformed = preprocessor.transform(sample)
                predictions = model.predict(transformed)
                assert len(predictions) == 1
            except Exception as e:
                pytest.fail(
                    f"Pipeline crashes on unseen category: {e}\n"
                    f"Use handle_unknown='ignore' in OneHotEncoder"
                )

    def test_reproducibility(self, training_artifacts):
        """Same input should produce same output every time."""
        preprocessor = training_artifacts["preprocessor"]
        model = training_artifacts["model"]
        sample = training_artifacts["train_sample"].head(5)

        pred1 = model.predict(preprocessor.transform(sample))
        pred2 = model.predict(preprocessor.transform(sample))
        np.testing.assert_array_equal(pred1, pred2)
```

---

## Staging Environment Tests

### Staging Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                     Staging Environment                       │
│                                                              │
│  ┌────────────┐     ┌─────────────┐     ┌────────────────┐  │
│  │  Staging    │     │   Model     │     │   Staging      │  │
│  │  Database   │────►│   Server    │────►│   Monitoring   │  │
│  │  (copy of  │     │   (same     │     │   (same        │  │
│  │   prod)    │     │    image)   │     │    dashboards) │  │
│  └────────────┘     └──────┬──────┘     └────────────────┘  │
│                            │                                 │
│                     ┌──────┴──────┐                          │
│                     │ Integration │                          │
│                     │   Tests     │                          │
│                     │   (pytest)  │                          │
│                     └─────────────┘                          │
│                                                              │
│  Must pass ALL staging tests before promoting to production  │
└──────────────────────────────────────────────────────────────┘
```

### Staging Tests

```python
# tests/test_staging.py
"""
Tests that run against the staging environment.
Triggered after deployment to staging, before promotion to prod.
"""
import requests
import pytest
import time
import statistics


STAGING_URL = "https://staging.model-api.internal"


@pytest.fixture(scope="module")
def staging_health():
    """Wait for staging to be healthy before running tests."""
    for attempt in range(30):
        try:
            r = requests.get(f"{STAGING_URL}/health", timeout=5)
            if r.status_code == 200:
                return True
        except requests.ConnectionError:
            pass
        time.sleep(2)
    pytest.fail("Staging environment not healthy after 60 seconds")


class TestStagingSmoke:
    """Smoke tests — does the service respond at all?"""

    def test_health_endpoint(self, staging_health):
        r = requests.get(f"{STAGING_URL}/health")
        assert r.status_code == 200

    def test_model_version(self, staging_health):
        r = requests.get(f"{STAGING_URL}/info")
        assert r.status_code == 200
        info = r.json()
        assert "model_version" in info
        assert info["model_version"] != ""

    def test_predict_returns_result(self, staging_health):
        r = requests.post(f"{STAGING_URL}/predict", json={
            "age": 35, "income": 75000, "category": "B",
            "num_purchases": 12, "days_since_last": 5,
        })
        assert r.status_code == 200
        assert "label" in r.json()


class TestStagingLatency:
    """Latency must meet SLA before promotion."""

    def test_p50_latency(self, staging_health):
        latencies = []
        for _ in range(50):
            start = time.time()
            r = requests.post(f"{STAGING_URL}/predict", json={
                "age": 35, "income": 75000, "category": "B",
                "num_purchases": 12, "days_since_last": 5,
            })
            latencies.append(time.time() - start)
            assert r.status_code == 200

        p50 = statistics.median(latencies) * 1000
        p99 = sorted(latencies)[int(0.99 * len(latencies))] * 1000
        assert p50 < 100, f"p50 latency {p50:.0f}ms exceeds 100ms SLA"
        assert p99 < 500, f"p99 latency {p99:.0f}ms exceeds 500ms SLA"


class TestStagingGoldenDataset:
    """Run golden dataset through staging API."""

    def test_golden_accuracy(self, staging_health):
        import pandas as pd
        golden = pd.read_csv("tests/fixtures/golden_dataset.csv")
        correct = 0
        total = len(golden)

        for _, row in golden.iterrows():
            payload = row.drop("expected_label").to_dict()
            r = requests.post(f"{STAGING_URL}/predict", json=payload)
            assert r.status_code == 200
            if r.json()["label"] == row["expected_label"]:
                correct += 1

        accuracy = correct / total
        assert accuracy >= 0.95, \
            f"Golden dataset accuracy in staging: {accuracy:.2%}"
```

---

## Test Data Management

### Golden Datasets

```
┌──────────────────────────────────────────────────────────────┐
│                    Golden Dataset                             │
│                                                              │
│  A small, curated set of input-output pairs that:            │
│  • Were manually verified or agreed upon                     │
│  • Cover normal cases + edge cases                           │
│  • Are version-controlled alongside the model                │
│  • Serve as regression tests — must always pass              │
│                                                              │
│  tests/fixtures/golden_dataset.csv                           │
│  ┌─────┬────────┬──────────┬────┬────────────────┐          │
│  │ age │ income │ category │ ...│ expected_label  │          │
│  ├─────┼────────┼──────────┼────┼────────────────┤          │
│  │  25 │  40000 │    A     │    │       0         │          │
│  │  60 │ 120000 │    C     │    │       1         │          │
│  │   0 │  10000 │    A     │    │       0         │  edge    │
│  │ 120 │ 500000 │    B     │    │       1         │  cases   │
│  └─────┴────────┴──────────┴────┴────────────────┘          │
│                                                              │
│  Size: 50–200 rows (small enough to run in every CI build)   │
└──────────────────────────────────────────────────────────────┘
```

### Creating Golden Datasets

```python
# scripts/create_golden_dataset.py
"""
Generate a golden dataset from a validated model.
Run once when a model is approved, store in version control.
"""
import pandas as pd
import numpy as np
from src.pipeline import MLPipeline

pipeline = MLPipeline.load("models/pipeline_v2/")

# Select representative samples
test_data = pd.read_parquet("data/test.parquet")

golden_samples = pd.concat([
    test_data.sample(40, random_state=42),          # random
    test_data[test_data["age"] == 0].head(2),       # edge: min age
    test_data[test_data["income"] > 400000].head(3), # edge: high income
    test_data[test_data["category"] == "C"].head(5), # rare category
])

golden_samples["expected_label"] = pipeline.predict(
    golden_samples.drop(columns=["expected_label"], errors="ignore")
)

golden_samples.to_csv("tests/fixtures/golden_dataset.csv", index=False)
print(f"Golden dataset: {len(golden_samples)} rows saved")
```

---

## Integration Test Patterns

### Pattern Summary

| Pattern | Purpose | When to Run |
|---|---|---|
| **Golden dataset** | Regression test with known outputs | Every CI build |
| **Round-trip test** | Serialize → deserialize → predict gives same result | After model packaging |
| **Shadow comparison** | New model outputs vs current production outputs | Before promotion |
| **Canary test** | Small % of real traffic to new model | After staging |
| **Smoke test** | Does the service respond at all? | After every deployment |
| **Contract test** | Request/response schemas match spec | Every CI build |
| **Load test** | Latency and throughput under load | Before promotion |

### Anti-Patterns to Avoid

```
┌──────────────────────────────────────────────────────────────┐
│                Integration Test Anti-Patterns                │
│                                                              │
│  ❌ Testing against live production                          │
│     → Use staging or mocked services                         │
│                                                              │
│  ❌ Giant test with no assertions                            │
│     → "It didn't crash" is not a test                        │
│                                                              │
│  ❌ Hardcoded predictions                                    │
│     → Model updates break all tests                          │
│     → Use golden datasets regenerated with each model        │
│                                                              │
│  ❌ No timeout on staging tests                              │
│     → Hanging tests block the pipeline forever               │
│                                                              │
│  ❌ Sharing state between tests                              │
│     → Tests pass alone, fail together                        │
│                                                              │
│  ✅ Test behaviors, not exact values                         │
│  ✅ Regenerate golden data when model changes                │
│  ✅ Set timeouts on all network calls                        │
│  ✅ Isolate each test                                        │
└──────────────────────────────────────────────────────────────┘
```

---

## Summary

| Concept | Key Takeaway |
|---|---|
| **Why integration test** | ML systems fail at the seams between components; unit tests miss these |
| **End-to-end pipeline** | Test raw data → preprocessing → model → prediction as a single flow |
| **API contract testing** | Validate request/response schemas, error codes, and edge cases |
| **Model + preprocessor** | Verify feature count, order, dtypes, and unseen-category handling |
| **Staging tests** | Smoke test, latency SLA, and golden dataset validation before promotion |
| **Golden datasets** | Small, curated input-output pairs; version-controlled regression tests |
| **Test data management** | Regenerate golden data with each model version; include edge cases |
| **Anti-patterns** | Don't test against prod, don't hardcode predictions, always set timeouts |

> **Rule of thumb:** If your pipeline has N components, you need at least N−1 integration tests — one for every interface between consecutive components. The most critical interface is preprocessor → model, because that's where training/serving skew happens.

---

## Revision Questions

1. **Explain** why a model can pass all unit tests but fail integration tests. Give three specific examples of bugs that only integration tests catch.

2. **Design** an API contract test suite for a text classification API with endpoints `/predict` (single) and `/predict/batch`. Include valid requests, invalid requests, edge cases, and latency checks.

3. **A model** was retrained and deployed to staging. The golden dataset test fails with 88% accuracy (threshold 95%). Describe your debugging process — what would you check first, second, and third?

4. **Compare** golden dataset testing and shadow mode testing (Chapter 5). When would you use each, and what are the limitations of each approach?

5. **Your team** discovers that the production model silently produces wrong predictions because the one-hot encoder drops an unseen category. Write an integration test that would have caught this, and describe how to prevent the issue architecturally.

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Model Testing](./03-model-testing.md) | [📂 ML Testing](./README.md) | [Shadow Mode Testing →](./05-shadow-mode-testing.md) |
