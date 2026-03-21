# Data Validation

## Overview

Data validation ensures that data entering your ML pipeline meets expected quality standards — correct schemas, valid ranges, no unexpected nulls, and consistent distributions. Tools like **Great Expectations**, **TensorFlow Data Validation (TFDV)**, and **Pandera** automate these checks. Catching data issues early prevents garbage-in-garbage-out failures downstream.

---

## Why Validate Data?

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  Without validation, bad data silently breaks your model:    │
  │                                                                │
  │  1. Schema changes:                                          │
  │     Column "age" was int → now string "25 years"            │
  │     → Training crashes or produces garbage                   │
  │                                                                │
  │  2. Missing values:                                          │
  │     New data source has 40% nulls in key feature             │
  │     → Model accuracy drops silently                          │
  │                                                                │
  │  3. Distribution shift:                                      │
  │     "price" mean was $50 → now $5000 (upstream bug)          │
  │     → Model makes wildly wrong predictions                   │
  │                                                                │
  │  4. Label corruption:                                        │
  │     Someone relabeled "positive" as 1 instead of "1"         │
  │     → Training loss doesn't converge                         │
  │                                                                │
  │  Goal: FAIL FAST when data is bad, before wasting compute   │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## What to Validate

```
  ┌──────────────────┬──────────────────────────────────────┐
  │ Check Type       │ Examples                             │
  ├──────────────────┼──────────────────────────────────────┤
  │ Schema           │ Column names, data types, required   │
  │                  │ columns present                      │
  ├──────────────────┼──────────────────────────────────────┤
  │ Completeness     │ No unexpected nulls, minimum row     │
  │                  │ count met                            │
  ├──────────────────┼──────────────────────────────────────┤
  │ Range / Domain   │ age ∈ [0, 120], price > 0,          │
  │                  │ status ∈ {active, inactive}          │
  ├──────────────────┼──────────────────────────────────────┤
  │ Uniqueness       │ No duplicate IDs, unique emails      │
  ├──────────────────┼──────────────────────────────────────┤
  │ Distribution     │ Mean, std, quantiles within expected │
  │                  │ ranges (detect drift)                │
  ├──────────────────┼──────────────────────────────────────┤
  │ Relationships    │ end_date > start_date, foreign key   │
  │                  │ references valid                     │
  ├──────────────────┼──────────────────────────────────────┤
  │ Freshness        │ Data timestamp within last 24 hours  │
  └──────────────────┴──────────────────────────────────────┘
```

---

## Great Expectations

```
  Most popular open-source data validation framework

  Core concepts:
  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Expectation: a single assertion about data      │
  │    e.g., "column age should be between 0 and 120"│
  │                                                    │
  │  Expectation Suite: collection of expectations   │
  │    for a dataset                                  │
  │                                                    │
  │  Checkpoint: run a suite against data and        │
  │    generate validation results                   │
  │                                                    │
  │  Data Docs: auto-generated HTML reports          │
  │    showing validation results                    │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Python: Great Expectations

```python
import great_expectations as gx

# Create a data context
context = gx.get_context()

# Connect to your data
datasource = context.sources.add_pandas("my_datasource")
data_asset = datasource.add_csv_asset(
    "training_data", filepath_or_buffer="data/train.csv"
)
batch_request = data_asset.build_batch_request()

# Create expectations
validator = context.get_validator(
    batch_request=batch_request,
    expectation_suite_name="training_data_suite",
    create_expectation_suite_with_name=True,
)

# Schema checks
validator.expect_table_columns_to_match_ordered_list(
    ["user_id", "age", "income", "purchase_count", "churn"]
)

# Completeness
validator.expect_column_values_to_not_be_null("user_id")
validator.expect_column_values_to_not_be_null("churn")

# Range checks
validator.expect_column_values_to_be_between("age", min_value=0, max_value=120)
validator.expect_column_values_to_be_between("income", min_value=0)

# Domain checks
validator.expect_column_values_to_be_in_set("churn", [0, 1])

# Distribution checks
validator.expect_column_mean_to_be_between("age", min_value=25, max_value=55)
validator.expect_column_stdev_to_be_between("income", min_value=100, max_value=50000)

# Uniqueness
validator.expect_column_values_to_be_unique("user_id")

# Row count
validator.expect_table_row_count_to_be_between(min_value=10000)

# Save the suite
validator.save_expectation_suite(discard_failed_expectations=False)

# Run validation checkpoint
checkpoint = context.add_or_update_checkpoint(
    name="training_data_checkpoint",
    validations=[{
        "batch_request": batch_request,
        "expectation_suite_name": "training_data_suite",
    }],
)

result = checkpoint.run()
print(f"Validation passed: {result.success}")
```

---

## Pandera (Schema Validation for DataFrames)

```python
import pandera as pa
import pandas as pd

# Define schema with Pandera
schema = pa.DataFrameSchema({
    "user_id": pa.Column(int, pa.Check.greater_than(0), unique=True),
    "age": pa.Column(int, [
        pa.Check.in_range(0, 120),
        pa.Check(lambda s: s.mean() > 20, error="Mean age too low"),
    ]),
    "income": pa.Column(float, pa.Check.greater_than(0), nullable=True),
    "purchase_count": pa.Column(int, pa.Check.greater_than_or_equal_to(0)),
    "churn": pa.Column(int, pa.Check.isin([0, 1])),
}, coerce=True)

# Validate data
df = pd.read_csv("data/train.csv")
try:
    validated_df = schema.validate(df)
    print("✅ Data validation passed!")
except pa.errors.SchemaError as e:
    print(f"❌ Validation failed: {e}")

# Use as decorator in pipelines
@pa.check_input(schema)
def train_model(df: pd.DataFrame):
    """Data is guaranteed valid when this function runs."""
    # ... training code
    pass
```

---

## TensorFlow Data Validation (TFDV)

```python
import tensorflow_data_validation as tfdv

# Generate statistics from training data
train_stats = tfdv.generate_statistics_from_csv("data/train.csv")

# Infer schema from training data (baseline)
schema = tfdv.infer_schema(train_stats)

# Validate new data against the schema
new_stats = tfdv.generate_statistics_from_csv("data/new_batch.csv")
anomalies = tfdv.validate_statistics(new_stats, schema)

# Display anomalies
tfdv.display_anomalies(anomalies)

# Detect distribution drift
tfdv.visualize_statistics(
    lhs_statistics=train_stats,    # reference
    rhs_statistics=new_stats,      # new data
    lhs_name="Training",
    rhs_name="New Batch",
)
```

---

## Validation in ML Pipelines

```
  Where to validate:

  ┌──────┐   ┌──────────┐   ┌──────┐   ┌──────┐   ┌──────┐
  │Ingest│──►│Validate! │──►│ Prep │──►│Train │──►│Deploy│
  └──────┘   └────┬─────┘   └──────┘   └──────┘   └──────┘
                   │
              FAIL? → Alert team, halt pipeline
                     Don't train on bad data!

  Validate at MULTIPLE points:
  1. After ingestion (raw data quality)
  2. After preprocessing (transformed data correctness)
  3. Before training (final feature checks)
  4. At serving time (input validation for predictions)
```

---

## Revision Questions

1. **Why is data validation critical in ML pipelines?**
2. **What are the main categories of data checks?**
3. **How does Great Expectations organize validations (concepts)?**
4. **How does Pandera differ from Great Expectations in approach?**
5. **At which points in an ML pipeline should you validate data?**

---

[Previous: 01-data-versioning-dvc.md](01-data-versioning-dvc.md) | [Next: 03-data-pipelines.md](03-data-pipelines.md)
