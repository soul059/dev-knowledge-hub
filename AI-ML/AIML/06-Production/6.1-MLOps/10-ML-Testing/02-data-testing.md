# 📊 Data Testing

> **Unit 10 · ML Testing · Chapter 2**
>
> How to validate data quality programmatically — Great Expectations for declarative testing, schema validation with Pandera, data contracts between teams, testing for missing values, range checks, and distribution drift tests.

---

## Table of Contents

1. [Chapter Overview](#chapter-overview)
2. [Why Data Testing Matters](#why-data-testing-matters)
3. [Great Expectations](#great-expectations)
4. [Pandera: DataFrame Schema Validation](#pandera-dataframe-schema-validation)
5. [Data Contracts](#data-contracts)
6. [Testing for Missing Values](#testing-for-missing-values)
7. [Range Checks and Outlier Detection](#range-checks-and-outlier-detection)
8. [Distribution Tests](#distribution-tests)
9. [Putting It Together: Data Validation Pipeline](#putting-it-together-data-validation-pipeline)
10. [Summary](#summary)
11. [Revision Questions](#revision-questions)
12. [Navigation](#navigation)

---

## Chapter Overview

**"Garbage in, garbage out"** is the oldest truism in ML, yet most teams have zero automated checks on the data flowing into their models. A model trained on data where `age` is in months instead of years will learn confidently — and be confidently wrong. This chapter covers the tools and techniques for **data testing**: validating schemas, checking for nulls and out-of-range values, detecting distribution shifts, and establishing contracts between data producers and ML consumers. We focus on two leading libraries — **Great Expectations** for rich declarative suites and **Pandera** for lightweight DataFrame validation — plus raw pytest techniques you can use anywhere.

```
Data Pipeline Without Testing          Data Pipeline With Testing
┌───────────┐                          ┌───────────┐
│  Raw Data │                          │  Raw Data │
└─────┬─────┘                          └─────┬─────┘
      │                                      │
      ▼                                      ▼
┌───────────┐                          ┌───────────────────┐
│  Feature  │                          │  Schema Validation │◄── Pandera
│  Pipeline │                          │  Range Checks      │◄── Great Expectations
└─────┬─────┘                          │  Null Checks       │
      │                                │  Distribution Test │
      ▼                                └────────┬──────────┘
┌───────────┐                                   │
│   Model   │  ← silent data bug                ▼ Pass / ❌ Fail + alert
└───────────┘                          ┌───────────┐
                                       │  Feature  │
                                       │  Pipeline │
                                       └─────┬─────┘
                                             │
                                             ▼
                                       ┌───────────┐
                                       │   Model   │  ← clean data
                                       └───────────┘
```

---

## Why Data Testing Matters

### Real-World Data Bugs

| Bug | Cause | Impact | Detection Method |
|---|---|---|---|
| Column renamed upstream | Schema change by data team | Pipeline crash or silent wrong joins | Schema validation |
| Age field in months | Unit change after migration | Model predictions wildly off | Range check |
| 40% nulls in key feature | ETL failure | Model defaults or crashes | Null rate test |
| New category value | Business added "platinum" tier | One-hot encoder crash | Allowed values test |
| Price distribution shifted | Currency changed to cents | Feature scaling breaks | Distribution test |

### Where to Place Data Tests

```
                Data Source
                    │
                    ▼
          ┌─────────────────┐
     ①    │  Ingestion Gate  │  Schema, null %, row count
          └────────┬────────┘
                   │
                   ▼
          ┌─────────────────┐
     ②    │  Post-Transform  │  Range checks, feature stats
          └────────┬────────┘
                   │
                   ▼
          ┌─────────────────┐
     ③    │  Pre-Model Gate  │  Distribution test vs training data
          └────────┬────────┘
                   │
                   ▼
              Model Inference

  ① Catch upstream breakage early
  ② Catch bugs in your own transforms
  ③ Catch drift before it degrades predictions
```

---

## Great Expectations

**Great Expectations** (GE) is the most popular data validation framework. It provides a declarative API for defining "expectations" — assertions about data — and generates documentation and alerts.

### Installation

```bash
pip install great-expectations
```

### Core Concepts

```
┌──────────────────────────────────────────────────────────────┐
│                   Great Expectations                         │
│                                                              │
│  Expectation Suite    A collection of expectations           │
│  ├── expect_column_to_exist("age")                          │
│  ├── expect_column_values_to_be_between("age", 0, 120)     │
│  └── expect_column_values_to_not_be_null("user_id")        │
│                                                              │
│  Validator            Runs suite against a DataFrame         │
│  Checkpoint           Orchestrates validation in pipelines   │
│  Data Docs            Auto-generated HTML reports            │
└──────────────────────────────────────────────────────────────┘
```

### Defining Expectations

```python
# data_tests/validate_users.py
import great_expectations as gx
import pandas as pd

# Create a context
context = gx.get_context()

# Connect to data
data_source = context.data_sources.add_pandas("pandas_source")
data_asset = data_source.add_dataframe_asset(name="user_data")

# Sample data
df = pd.read_csv("data/users.csv")
batch = data_asset.add_batch_definition_whole_dataframe("batch").get_batch(
    dataframe=df
)

# Define expectations
suite = context.suites.add(
    gx.ExpectationSuite(name="user_data_suite")
)

# Column existence
suite.add_expectation(
    gx.expectations.ExpectColumnToExist(column="user_id")
)
suite.add_expectation(
    gx.expectations.ExpectColumnToExist(column="age")
)

# Null checks
suite.add_expectation(
    gx.expectations.ExpectColumnValuesToNotBeNull(column="user_id")
)

# Range checks
suite.add_expectation(
    gx.expectations.ExpectColumnValuesToBeBetween(
        column="age", min_value=0, max_value=120
    )
)

# Allowed values
suite.add_expectation(
    gx.expectations.ExpectColumnDistinctValuesToBeInSet(
        column="tier", value_set=["free", "pro", "enterprise"]
    )
)

# Uniqueness
suite.add_expectation(
    gx.expectations.ExpectColumnValuesToBeUnique(column="user_id")
)

# Run validation
validation_result = batch.validate(suite)
print(f"Success: {validation_result.success}")
```

### Common Expectations Quick Reference

| Expectation | What It Checks |
|---|---|
| `ExpectColumnToExist` | Column exists in DataFrame |
| `ExpectColumnValuesToNotBeNull` | No null values |
| `ExpectColumnValuesToBeBetween` | Values within min/max range |
| `ExpectColumnDistinctValuesToBeInSet` | Only expected categories present |
| `ExpectColumnValuesToBeUnique` | No duplicate values |
| `ExpectColumnMeanToBeBetween` | Mean within expected range |
| `ExpectColumnMedianToBeBetween` | Median within expected range |
| `ExpectTableRowCountToBeBetween` | Row count within bounds |
| `ExpectColumnValuesToMatchRegex` | Values match a regex pattern |
| `ExpectColumnKlDivergenceToBeLessThan` | Distribution similarity check |

---

## Pandera: DataFrame Schema Validation

**Pandera** is a lightweight alternative focused on **type-safe DataFrame validation** with Python type hints. It integrates naturally with pytest.

### Installation

```bash
pip install pandera
```

### Defining Schemas

```python
# src/schemas.py
import pandera as pa
from pandera import Column, Check, Index
import pandas as pd

# --- Declarative schema ---
user_schema = pa.DataFrameSchema(
    columns={
        "user_id": Column(int, Check.gt(0), nullable=False, unique=True),
        "age": Column(int, [
            Check.ge(0),
            Check.le(120),
        ], nullable=False),
        "income": Column(float, Check.gt(0), nullable=True),
        "tier": Column(str, Check.isin(["free", "pro", "enterprise"])),
        "signup_date": Column("datetime64[ns]", nullable=False),
    },
    coerce=True,   # Attempt type coercion before validation
)


# --- Class-based schema (modern Pandera) ---
class UserSchema(pa.DataFrameModel):
    user_id: int = pa.Field(gt=0, unique=True, nullable=False)
    age: int = pa.Field(ge=0, le=120, nullable=False)
    income: float = pa.Field(gt=0, nullable=True)
    tier: str = pa.Field(isin=["free", "pro", "enterprise"])

    class Config:
        coerce = True
        strict = True          # Fail if extra columns present
```

### Using Pandera in Tests

```python
# tests/test_data_schema.py
import pandas as pd
import pandera as pa
import pytest
from src.schemas import user_schema, UserSchema


@pytest.fixture
def valid_df():
    return pd.DataFrame({
        "user_id": [1, 2, 3],
        "age": [25, 40, 55],
        "income": [50000.0, 75000.0, 90000.0],
        "tier": ["free", "pro", "enterprise"],
        "signup_date": pd.to_datetime(["2024-01-01", "2024-02-01", "2024-03-01"]),
    })


def test_valid_data_passes(valid_df):
    """Valid data should pass without errors."""
    validated = user_schema.validate(valid_df)
    assert len(validated) == 3


def test_negative_age_fails(valid_df):
    """Negative age should fail validation."""
    valid_df.loc[0, "age"] = -5
    with pytest.raises(pa.errors.SchemaError):
        user_schema.validate(valid_df)


def test_unknown_tier_fails(valid_df):
    """Unexpected tier value should fail."""
    valid_df.loc[0, "tier"] = "platinum"
    with pytest.raises(pa.errors.SchemaError):
        user_schema.validate(valid_df)


def test_duplicate_user_id_fails(valid_df):
    """Duplicate user_id should fail."""
    valid_df.loc[2, "user_id"] = 1
    with pytest.raises(pa.errors.SchemaError):
        user_schema.validate(valid_df)


# --- Using class-based schema ---
def test_class_based_schema(valid_df):
    validated = UserSchema.validate(valid_df)
    assert len(validated) == 3
```

### Pandera vs Great Expectations

| Feature | Pandera | Great Expectations |
|---|---|---|
| **Philosophy** | Type-safe schemas in code | Declarative expectation suites |
| **Best for** | CI/pytest integration | Data pipelines + reporting |
| **Learning curve** | Low (feels like dataclasses) | Moderate (config-heavy) |
| **Documentation** | — | Auto-generated Data Docs (HTML) |
| **Integration** | pytest, FastAPI, Pydantic | Airflow, Spark, Databricks |
| **Schema definition** | Python classes / dicts | JSON / Python |
| **Performance** | Fast (in-process) | More overhead (checkpoint system) |
| **Ideal team size** | Small–medium | Medium–large |

---

## Data Contracts

A **data contract** is a formal agreement between a data producer (e.g., backend team) and a data consumer (e.g., ML team) about the structure and quality of shared data.

### Contract Components

```
┌──────────────────────────────────────────────────────────────┐
│                     DATA CONTRACT                            │
│                                                              │
│  Owner:        Backend Team (payments service)               │
│  Consumer:     ML Team (fraud detection model)               │
│  Dataset:      transactions                                  │
│  SLA:          Updated every 15 min, < 0.1% nulls            │
│                                                              │
│  Schema:                                                     │
│  ┌─────────────┬──────────┬──────────┬────────────────────┐  │
│  │ Column      │ Type     │ Nullable │ Constraints        │  │
│  ├─────────────┼──────────┼──────────┼────────────────────┤  │
│  │ txn_id      │ string   │ No       │ UUID format        │  │
│  │ amount      │ float    │ No       │ > 0                │  │
│  │ currency    │ string   │ No       │ ISO 4217           │  │
│  │ user_id     │ int      │ No       │ > 0                │  │
│  │ timestamp   │ datetime │ No       │ UTC, no future     │  │
│  │ merchant    │ string   │ Yes      │ —                  │  │
│  └─────────────┴──────────┴──────────┴────────────────────┘  │
│                                                              │
│  Quality Rules:                                              │
│  • Row count > 1000 per batch                                │
│  • amount mean between $10 and $500                          │
│  • No duplicate txn_id                                       │
└──────────────────────────────────────────────────────────────┘
```

### Implementing Contracts as Tests

```python
# tests/test_data_contract.py
"""
Tests that enforce the data contract for the transactions dataset.
If any of these fail, page the data producer — not the ML team.
"""
import pandas as pd
import pytest


@pytest.fixture
def transactions():
    return pd.read_parquet("data/transactions_latest.parquet")


class TestTransactionContract:
    def test_required_columns_exist(self, transactions):
        required = {"txn_id", "amount", "currency", "user_id", "timestamp"}
        assert required.issubset(set(transactions.columns))

    def test_no_null_primary_key(self, transactions):
        assert transactions["txn_id"].isna().sum() == 0

    def test_no_duplicate_primary_key(self, transactions):
        assert transactions["txn_id"].is_unique

    def test_amount_positive(self, transactions):
        assert (transactions["amount"] > 0).all()

    def test_currency_valid(self, transactions):
        valid_currencies = {"USD", "EUR", "GBP", "JPY", "CAD"}
        actual = set(transactions["currency"].unique())
        assert actual.issubset(valid_currencies), \
            f"Unexpected currencies: {actual - valid_currencies}"

    def test_row_count_reasonable(self, transactions):
        assert len(transactions) >= 1000, \
            f"Expected >= 1000 rows, got {len(transactions)}"

    def test_amount_mean_in_range(self, transactions):
        mean_amt = transactions["amount"].mean()
        assert 10 <= mean_amt <= 500, \
            f"Mean amount {mean_amt:.2f} outside expected range"
```

---

## Testing for Missing Values

### Null Check Strategies

```python
# tests/test_nulls.py
import pandas as pd
import numpy as np
import pytest


@pytest.fixture
def df():
    return pd.read_parquet("data/features.parquet")


class TestNulls:
    def test_critical_columns_no_nulls(self, df):
        """Columns the model cannot handle as NaN."""
        critical = ["user_id", "label", "timestamp"]
        for col in critical:
            null_count = df[col].isna().sum()
            assert null_count == 0, \
                f"Column '{col}' has {null_count} nulls"

    def test_null_rate_below_threshold(self, df):
        """Non-critical columns may have some nulls, but not too many."""
        max_null_rate = 0.05   # 5%
        for col in df.columns:
            null_rate = df[col].isna().mean()
            assert null_rate <= max_null_rate, \
                f"Column '{col}' null rate {null_rate:.1%} > {max_null_rate:.1%}"

    def test_no_fully_null_columns(self, df):
        """No column should be entirely null."""
        for col in df.columns:
            assert not df[col].isna().all(), \
                f"Column '{col}' is entirely null"

    def test_null_pattern_not_correlated(self, df):
        """If row is null in col A, it shouldn't always be null in col B."""
        null_matrix = df.isna()
        corr = null_matrix.corr()
        np.fill_diagonal(corr.values, 0)
        max_corr = corr.abs().max().max()
        assert max_corr < 0.9, \
            f"High null correlation ({max_corr:.2f}) — possible systematic issue"
```

---

## Range Checks and Outlier Detection

### Boundary Tests

```python
# tests/test_ranges.py
import pandas as pd
import numpy as np
import pytest


@pytest.fixture
def features():
    return pd.read_parquet("data/features.parquet")


class TestRanges:
    """Test that feature values are within physically plausible ranges."""

    RANGE_SPECS = {
        "age":          (0, 120),
        "income":       (0, 10_000_000),
        "temperature":  (-50, 60),
        "probability":  (0.0, 1.0),
        "num_items":    (0, 10_000),
    }

    @pytest.mark.parametrize("column, bounds", RANGE_SPECS.items())
    def test_values_in_range(self, features, column, bounds):
        if column not in features.columns:
            pytest.skip(f"Column '{column}' not in dataset")
        lo, hi = bounds
        values = features[column].dropna()
        assert (values >= lo).all(), \
            f"{column}: {(values < lo).sum()} values below {lo}"
        assert (values <= hi).all(), \
            f"{column}: {(values > hi).sum()} values above {hi}"

    def test_no_infinity_values(self, features):
        numeric = features.select_dtypes(include=[np.number])
        for col in numeric.columns:
            inf_count = np.isinf(numeric[col]).sum()
            assert inf_count == 0, \
                f"Column '{col}' has {inf_count} infinite values"

    def test_z_score_outliers(self, features):
        """Flag extreme outliers (|z| > 10) — likely data errors."""
        numeric = features.select_dtypes(include=[np.number])
        for col in numeric.columns:
            values = numeric[col].dropna()
            if len(values) < 10:
                continue
            z_scores = (values - values.mean()) / values.std()
            extreme = (z_scores.abs() > 10).sum()
            assert extreme == 0, \
                f"Column '{col}' has {extreme} extreme outliers (|z| > 10)"
```

---

## Distribution Tests

Distribution tests detect **drift** — when the shape of the data changes from what the model was trained on.

### Statistical Tests for Drift

```
Training Data Distribution        Production Data Distribution
┌────────────────────────┐       ┌────────────────────────┐
│       ▄▄▄▄             │       │            ▄▄▄▄        │
│     ▄██████▄           │       │          ▄██████▄      │
│   ▄██████████▄         │       │        ▄██████████▄    │
│ ▄██████████████▄       │       │      ▄██████████████▄  │
│████████████████████    │       │    ████████████████████│
└────────────────────────┘       └────────────────────────┘
     Mean ≈ 50                        Mean ≈ 65  ← SHIFTED!

KS Test:   p-value < 0.01 → Reject null hypothesis → DRIFT DETECTED
```

### Implementation

```python
# tests/test_distribution.py
import pandas as pd
import numpy as np
from scipy import stats
import pytest


@pytest.fixture
def train_stats():
    """Statistics from the training dataset (saved during training)."""
    return {
        "age":    {"mean": 38.5, "std": 13.2, "min": 18, "max": 95},
        "income": {"mean": 62000, "std": 35000, "min": 0, "max": 500000},
    }


@pytest.fixture
def train_data():
    """Reference sample from training data."""
    return pd.read_parquet("data/train_reference.parquet")


@pytest.fixture
def prod_data():
    """Latest production data batch."""
    return pd.read_parquet("data/prod_latest.parquet")


class TestDistribution:
    def test_mean_within_range(self, prod_data, train_stats):
        """Production mean should be within 2 std of training mean."""
        for col, ref in train_stats.items():
            if col not in prod_data.columns:
                continue
            prod_mean = prod_data[col].mean()
            lower = ref["mean"] - 2 * ref["std"]
            upper = ref["mean"] + 2 * ref["std"]
            assert lower <= prod_mean <= upper, \
                f"{col} mean {prod_mean:.1f} outside [{lower:.1f}, {upper:.1f}]"

    def test_ks_test_no_drift(self, train_data, prod_data):
        """Kolmogorov-Smirnov test for distribution shift."""
        numeric_cols = train_data.select_dtypes(include=[np.number]).columns
        for col in numeric_cols:
            if col not in prod_data.columns:
                continue
            stat, p_value = stats.ks_2samp(
                train_data[col].dropna(),
                prod_data[col].dropna()
            )
            assert p_value > 0.01, \
                f"{col}: KS test p={p_value:.4f} — distribution shift detected"

    def test_chi_squared_categorical(self, train_data, prod_data):
        """Chi-squared test for categorical distribution shift."""
        cat_cols = train_data.select_dtypes(include=["object", "category"]).columns
        for col in cat_cols:
            if col not in prod_data.columns:
                continue
            train_counts = train_data[col].value_counts()
            prod_counts = prod_data[col].value_counts()

            # Align categories
            all_cats = set(train_counts.index) | set(prod_counts.index)
            train_freq = [train_counts.get(c, 0) for c in all_cats]
            prod_freq = [prod_counts.get(c, 0) for c in all_cats]

            if sum(prod_freq) == 0:
                continue

            # Normalize to same total
            total = sum(train_freq)
            expected = [f * sum(prod_freq) / total for f in train_freq]

            chi2, p_value = stats.chisquare(prod_freq, f_exp=expected)
            assert p_value > 0.01, \
                f"{col}: chi² p={p_value:.4f} — category distribution shifted"

    def test_new_categories_detected(self, train_data, prod_data):
        """Alert if production data has categories not seen in training."""
        cat_cols = train_data.select_dtypes(include=["object", "category"]).columns
        for col in cat_cols:
            if col not in prod_data.columns:
                continue
            train_cats = set(train_data[col].dropna().unique())
            prod_cats = set(prod_data[col].dropna().unique())
            new_cats = prod_cats - train_cats
            assert len(new_cats) == 0, \
                f"{col}: new categories in production: {new_cats}"
```

---

## Putting It Together: Data Validation Pipeline

### Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                 Data Validation Pipeline                      │
│                                                              │
│  ┌──────────┐   ┌───────────┐   ┌────────────┐   ┌───────┐ │
│  │ Ingest   │──►│  Schema   │──►│   Range    │──►│ Dist  │ │
│  │ Data     │   │  Check    │   │   Check    │   │ Check │ │
│  │          │   │ (Pandera) │   │ (pytest)   │   │ (KS)  │ │
│  └──────────┘   └─────┬─────┘   └─────┬──────┘   └───┬───┘ │
│                       │               │               │     │
│                  ❌ Fail          ❌ Fail          ❌ Fail   │
│                       │               │               │     │
│                       ▼               ▼               ▼     │
│                 ┌──────────────────────────────────────┐     │
│                 │         Alert + Block Pipeline       │     │
│                 └──────────────────────────────────────┘     │
│                                                              │
│                  ✅ All Pass                                  │
│                       │                                      │
│                       ▼                                      │
│                 ┌───────────┐                                │
│                 │  Feature  │                                │
│                 │  Pipeline │──► Model                       │
│                 └───────────┘                                │
└──────────────────────────────────────────────────────────────┘
```

### Implementation

```python
# src/data_validator.py
import pandas as pd
import pandera as pa
from scipy import stats
from dataclasses import dataclass


@dataclass
class ValidationResult:
    passed: bool
    failures: list[str]

    def __str__(self):
        if self.passed:
            return "✅ All data checks passed"
        return "❌ Data validation failed:\n" + "\n".join(
            f"  • {f}" for f in self.failures
        )


def validate_data(
    df: pd.DataFrame,
    schema: pa.DataFrameSchema,
    reference_df: pd.DataFrame | None = None,
    max_null_rate: float = 0.05,
) -> ValidationResult:
    failures = []

    # 1. Schema validation
    try:
        schema.validate(df, lazy=True)
    except pa.errors.SchemaErrors as e:
        for _, row in e.failure_cases.iterrows():
            failures.append(f"Schema: {row['column']} — {row['check']}")

    # 2. Null rate check
    for col in df.columns:
        null_rate = df[col].isna().mean()
        if null_rate > max_null_rate:
            failures.append(
                f"Null rate: {col} is {null_rate:.1%} (max {max_null_rate:.1%})"
            )

    # 3. Distribution check (if reference data provided)
    if reference_df is not None:
        numeric_cols = df.select_dtypes(include=["number"]).columns
        for col in numeric_cols:
            if col not in reference_df.columns:
                continue
            _, p_value = stats.ks_2samp(
                reference_df[col].dropna(), df[col].dropna()
            )
            if p_value < 0.01:
                failures.append(
                    f"Distribution: {col} shifted (KS p={p_value:.4f})"
                )

    return ValidationResult(passed=len(failures) == 0, failures=failures)
```

### Using in an Airflow DAG

```python
# dags/data_pipeline.py
from airflow.decorators import task, dag
from datetime import datetime

@dag(schedule="@hourly", start_date=datetime(2024, 1, 1))
def ml_pipeline():

    @task
    def ingest():
        import pandas as pd
        df = pd.read_parquet("s3://data/raw/latest.parquet")
        df.to_parquet("/tmp/ingested.parquet")

    @task
    def validate():
        import pandas as pd
        from src.data_validator import validate_data
        from src.schemas import user_schema

        df = pd.read_parquet("/tmp/ingested.parquet")
        ref = pd.read_parquet("s3://data/reference/train_sample.parquet")

        result = validate_data(df, user_schema, reference_df=ref)
        if not result.passed:
            raise ValueError(str(result))   # Fails the DAG

    @task
    def transform():
        ...

    ingest() >> validate() >> transform()

ml_pipeline()
```

---

## Summary

| Concept | Key Takeaway |
|---|---|
| **Why data testing** | Most ML bugs are data bugs; catch them before they reach the model |
| **Great Expectations** | Declarative expectation suites with auto-generated docs; best for large teams |
| **Pandera** | Lightweight schema validation with Python type hints; best for CI/pytest |
| **Data contracts** | Formal agreements between data producers and ML consumers |
| **Null checks** | Test critical columns for zero nulls; test others for rate < threshold |
| **Range checks** | Validate physically plausible ranges + detect infinities and extreme outliers |
| **Distribution tests** | KS test for numeric drift, chi-squared for categorical drift |
| **Validation pipeline** | Chain schema → range → distribution checks; block pipeline on failure |

> **Rule of thumb:** Every feature your model consumes should have at least one data test. If the data changes and your tests don't catch it, you'll find out from angry users instead.

---

## Revision Questions

1. **Compare** Great Expectations and Pandera for data validation. Which would you choose for a small team writing pytest-based tests vs a large team with Airflow pipelines?

2. **Design** a data contract for a recommendation system's user-activity dataset. Include schema, null constraints, value ranges, and freshness SLA. What happens when the contract is violated?

3. **Explain** why range checks alone are insufficient for detecting data quality issues. Give an example where data passes range checks but fails a distribution test.

4. **A production model's** accuracy drops 5% over a week. No code changed. Describe the data tests you would add to the pipeline to detect the root cause and prevent recurrence.

5. **Implement** a Pandera schema for a time-series dataset with columns `timestamp`, `sensor_id`, `value`, and `status`. Include checks for monotonically increasing timestamps, valid sensor IDs, and allowed status values.

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Unit Testing ML Code](./01-unit-testing-ml-code.md) | [📂 ML Testing](./README.md) | [Model Testing →](./03-model-testing.md) |
