# Feature Stores

## Overview

A **Feature Store** is a centralized system for managing, storing, and serving ML features. It solves the critical problem of **training-serving skew** by ensuring the exact same feature computation logic is used for both training and real-time inference. Popular feature stores include **Feast** (open-source), **Tecton**, **Hopsworks**, and built-in offerings from cloud providers.

---

## The Problem Feature Stores Solve

```
  Without a feature store:

  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  Data Scientist (Training):                                  │
  │    df["avg_purchase"] = df.groupby("user")["amount"].mean() │
  │    # Uses pandas, computes on full history                   │
  │                                                                │
  │  ML Engineer (Serving):                                      │
  │    SELECT AVG(amount) FROM purchases                         │
  │    WHERE user_id = ? AND timestamp > NOW() - INTERVAL 30 DAY│
  │    # Uses SQL, computes on last 30 days only!                │
  │                                                                │
  │  DIFFERENT LOGIC → DIFFERENT VALUES → MODEL BREAKS!          │
  │  This is "training-serving skew"                             │
  │                                                                │
  │  With a feature store:                                       │
  │    Both training and serving fetch features from the SAME    │
  │    source, computed by the SAME transformation code.         │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Feature Store Architecture

```
  ┌──────────────────────────────────────────────────────────────┐
  │                     FEATURE STORE                              │
  │                                                                │
  │  ┌────────────────────────────────────────────────────┐      │
  │  │            Feature Transformation                  │      │
  │  │  (Same code for batch + streaming)                │      │
  │  └──────────────────┬───────────────┬────────────────┘      │
  │                     │               │                         │
  │                     ▼               ▼                         │
  │  ┌──────────────────────┐  ┌──────────────────────┐         │
  │  │   Offline Store      │  │   Online Store       │         │
  │  │   (Historical)       │  │   (Low-latency)      │         │
  │  │                      │  │                      │         │
  │  │   Data warehouse     │  │   Redis / DynamoDB   │         │
  │  │   (BigQuery, S3)     │  │   (< 10ms reads)     │         │
  │  │                      │  │                      │         │
  │  │   Used for:          │  │   Used for:          │         │
  │  │   • Training data    │  │   • Real-time        │         │
  │  │   • Batch scoring    │  │     inference        │         │
  │  │   • Backfilling      │  │   • Online serving   │         │
  │  └──────────────────────┘  └──────────────────────┘         │
  │                                                                │
  │  ┌────────────────────────────────────────────────────┐      │
  │  │            Feature Registry (Metadata)             │      │
  │  │  • Feature names, types, descriptions             │      │
  │  │  • Owner, freshness SLA, data source              │      │
  │  │  • Feature lineage and dependencies               │      │
  │  └────────────────────────────────────────────────────┘      │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Offline vs. Online Store

```
  ┌──────────────────┬──────────────────────┬──────────────────────┐
  │ Aspect           │ Offline Store        │ Online Store         │
  ├──────────────────┼──────────────────────┼──────────────────────┤
  │ Purpose          │ Training data        │ Real-time serving    │
  ├──────────────────┼──────────────────────┼──────────────────────┤
  │ Latency          │ Seconds to minutes   │ < 10ms               │
  ├──────────────────┼──────────────────────┼──────────────────────┤
  │ Data scope       │ Full history         │ Latest values only   │
  ├──────────────────┼──────────────────────┼──────────────────────┤
  │ Storage          │ Data warehouse       │ Key-value store      │
  │                  │ (Parquet, BigQuery)  │ (Redis, DynamoDB)    │
  ├──────────────────┼──────────────────────┼──────────────────────┤
  │ Access pattern   │ Bulk reads (full     │ Point lookups        │
  │                  │ dataset for training)│ (single entity)      │
  ├──────────────────┼──────────────────────┼──────────────────────┤
  │ Update frequency │ Batch (hourly/daily) │ Near real-time       │
  └──────────────────┴──────────────────────┴──────────────────────┘
```

---

## Feast (Open-Source Feature Store)

```
  Feast = Feature Store for ML

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Key concepts:                                   │
  │                                                    │
  │  Entity:        the "who" (user_id, product_id)  │
  │  Feature View:  group of related features        │
  │  Data Source:   where raw data comes from         │
  │  Online Store:  fast lookup for serving           │
  │  Offline Store: historical data for training     │
  │                                                    │
  │  Workflow:                                       │
  │  1. Define features in Python                    │
  │  2. feast apply → register in Feast              │
  │  3. feast materialize → push to online store     │
  │  4. Get training data: get_historical_features() │
  │  5. Get serving data: get_online_features()      │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Python: Feast Setup and Usage

```python
# feature_repo/feature_definitions.py
from feast import Entity, FeatureView, Field, FileSource
from feast.types import Float32, Int64
from datetime import timedelta

# Define entity (the "who")
user = Entity(
    name="user_id",
    description="Unique user identifier",
)

# Define data source
user_stats_source = FileSource(
    path="data/user_stats.parquet",
    timestamp_field="event_timestamp",
)

# Define feature view
user_features = FeatureView(
    name="user_features",
    entities=[user],
    schema=[
        Field(name="total_purchases", dtype=Int64),
        Field(name="avg_order_value", dtype=Float32),
        Field(name="days_since_last_purchase", dtype=Int64),
        Field(name="account_age_days", dtype=Int64),
    ],
    source=user_stats_source,
    ttl=timedelta(days=1),  # feature freshness
)
```

```python
# Using Feast for training and serving
from feast import FeatureStore
import pandas as pd

store = FeatureStore(repo_path="feature_repo/")

# --- TRAINING: get historical features ---
entity_df = pd.DataFrame({
    "user_id": [1001, 1002, 1003, 1004],
    "event_timestamp": pd.to_datetime(["2025-01-15"] * 4),
})

training_df = store.get_historical_features(
    entity_df=entity_df,
    features=[
        "user_features:total_purchases",
        "user_features:avg_order_value",
        "user_features:days_since_last_purchase",
    ],
).to_df()

print(training_df)
# Train model on training_df...

# --- SERVING: get online features ---
# First, materialize features to online store
# $ feast materialize 2025-01-01 2025-03-15

online_features = store.get_online_features(
    features=[
        "user_features:total_purchases",
        "user_features:avg_order_value",
        "user_features:days_since_last_purchase",
    ],
    entity_rows=[{"user_id": 1001}],
).to_dict()

print(online_features)
# Pass to model for prediction
```

---

## Feature Store Comparison

```
  ┌─────────────────┬──────────┬──────────┬──────────┬───────────┐
  │ Feature         │ Feast    │ Tecton   │Hopsworks │ SageMaker │
  │                 │(open src)│(managed) │(open src)│  FS       │
  ├─────────────────┼──────────┼──────────┼──────────┼───────────┤
  │ Open-source     │ ✓        │          │ ✓        │           │
  │ Online store    │ ✓        │ ✓        │ ✓        │ ✓         │
  │ Offline store   │ ✓        │ ✓        │ ✓        │ ✓         │
  │ Streaming feat. │ Limited  │ ✓✓       │ ✓        │ ✓         │
  │ Feature transf. │ External │ Built-in │ Built-in │ Built-in  │
  │ Monitoring      │ External │ ✓        │ ✓        │ ✓         │
  │ Access control  │ Basic    │ ✓✓       │ ✓        │ ✓ (IAM)   │
  │ Complexity      │ Low      │ Medium   │ Medium   │ Medium    │
  │ Cost            │ Free     │ $$$      │ Free/$   │ $$        │
  └─────────────────┴──────────┴──────────┴──────────┴───────────┘
```

---

## Feature Store Benefits

```
  1. CONSISTENCY
     Same features for training and serving → no skew

  2. REUSABILITY
     Teams share features instead of rebuilding them
     "Has someone already computed 'user_lifetime_value'?"

  3. DISCOVERABILITY
     Feature registry → search and browse available features

  4. FRESHNESS
     SLAs ensure features are up-to-date
     Alerts when materialization is delayed

  5. POINT-IN-TIME CORRECTNESS
     Historical features respect event timestamps
     No data leakage from the future!
```

---

## Revision Questions

1. **What is training-serving skew and how does a feature store prevent it?**
2. **What is the difference between the online and offline store?**
3. **What are the key concepts in Feast (Entity, Feature View, etc.)?**
4. **Why is point-in-time correctness important for training data?**
5. **Compare Feast and Tecton — when would you choose each?**

---

[Previous: 03-data-pipelines.md](03-data-pipelines.md) | [Next: 05-data-quality-monitoring.md](05-data-quality-monitoring.md)
