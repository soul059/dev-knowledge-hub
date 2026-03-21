# Data Versioning (DVC)

## Overview

**Data Version Control (DVC)** is an open-source tool that brings Git-like versioning to datasets and ML models. Since Git can't handle large files efficiently, DVC stores data externally (S3, GCS, Azure, local) while tracking metadata in Git. This enables full reproducibility — every experiment is tied to an exact data snapshot.

---

## Why Version Data?

```
  Without data versioning:

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Monday:  train model on dataset_v1 → acc: 0.85  │
  │  Tuesday: someone updates the dataset             │
  │  Wednesday: retrain model → acc: 0.78             │
  │                                                    │
  │  "What changed? The code is the same!"           │
  │  → The DATA changed, but nobody tracked it!      │
  │                                                    │
  │  Problems:                                       │
  │  • Can't reproduce past results                  │
  │  • Can't compare experiments fairly              │
  │  • Can't audit what data trained a deployed model│
  │  • Can't rollback to a previous dataset          │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## How DVC Works

```
  DVC = Git for data

  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  Git tracks:                    DVC tracks:                  │
  │  • Source code (.py, .yaml)    • Large files (data, models) │
  │  • DVC metafiles (.dvc)        • Stored in remote storage   │
  │  • Pipeline definitions        • Linked via hash (MD5/SHA)  │
  │                                                                │
  │  ┌──────────────────┐         ┌──────────────────┐          │
  │  │   Git Repo       │         │  DVC Remote      │          │
  │  │                  │         │  (S3, GCS, etc.) │          │
  │  │  train.py        │         │                  │          │
  │  │  data.csv.dvc ───┼────────►│  data.csv (2GB)  │          │
  │  │  model.pkl.dvc ──┼────────►│  model.pkl       │          │
  │  │  dvc.yaml        │         │                  │          │
  │  │  dvc.lock        │         │                  │          │
  │  └──────────────────┘         └──────────────────┘          │
  │                                                                │
  │  .dvc file contains:                                         │
  │  - MD5 hash of the actual data file                          │
  │  - File size                                                 │
  │  - Remote storage location                                   │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Getting Started with DVC

```bash
# Install DVC
pip install dvc dvc-s3   # or dvc-gs, dvc-azure

# Initialize DVC in an existing Git repo
cd my-ml-project
dvc init

# Add a data file to DVC tracking
dvc add data/training_data.csv

# This creates:
#   data/training_data.csv.dvc   (metadata, tracked by Git)
#   data/.gitignore               (ignores actual data from Git)

# Commit the metadata to Git
git add data/training_data.csv.dvc data/.gitignore
git commit -m "Track training data with DVC"

# Configure remote storage
dvc remote add -d myremote s3://my-bucket/dvc-storage

# Push data to remote
dvc push

# On another machine, pull the data
git clone <repo>
dvc pull    # downloads exact data version from remote
```

---

## DVC Pipelines

```
  Define reproducible ML pipelines in dvc.yaml:

  # dvc.yaml
  stages:
    prepare:
      cmd: python src/prepare.py
      deps:
        - src/prepare.py
        - data/raw
      outs:
        - data/prepared

    train:
      cmd: python src/train.py
      deps:
        - src/train.py
        - data/prepared
      outs:
        - models/model.pkl
      metrics:
        - metrics.json:
            cache: false

    evaluate:
      cmd: python src/evaluate.py
      deps:
        - src/evaluate.py
        - models/model.pkl
        - data/prepared
      metrics:
        - eval_metrics.json:
            cache: false

  ┌──────────┐     ┌──────────┐     ┌──────────┐
  │ Prepare  │────►│  Train   │────►│ Evaluate │
  │ data/raw │     │ prepared │     │ model.pkl│
  └──────────┘     └──────────┘     └──────────┘

  Run the full pipeline:
  $ dvc repro

  DVC tracks which stages need rerunning based on
  changes to deps (code, data). Unchanged stages are skipped!
```

---

## Data Versioning Workflow

```python
# Typical workflow with DVC:

# 1. Initial data version
# $ dvc add data/train.csv
# $ git add data/train.csv.dvc
# $ git commit -m "v1: initial training data"
# $ git tag data-v1
# $ dvc push

# 2. Update data (add new samples, fix labels)
# $ dvc add data/train.csv       (new hash computed)
# $ git add data/train.csv.dvc
# $ git commit -m "v2: added 5000 new samples"
# $ git tag data-v2
# $ dvc push

# 3. Switch between data versions
# $ git checkout data-v1
# $ dvc checkout       → downloads v1 data
# $ git checkout data-v2
# $ dvc checkout       → downloads v2 data

# 4. Compare metrics across data versions
# $ dvc metrics diff data-v1 data-v2
```

---

## DVC vs. Alternatives

```
  ┌─────────────────┬──────────┬──────────┬──────────┬──────────┐
  │ Feature         │ DVC      │ LakeFS   │ Delta    │ Git LFS  │
  │                 │          │          │ Lake     │          │
  ├─────────────────┼──────────┼──────────┼──────────┼──────────┤
  │ File-based data │ ✓        │ ✓        │          │ ✓        │
  │ Tabular data    │ ✓        │ ✓        │ ✓        │ ✓        │
  │ Git integration │ ✓✓       │ ✓        │          │ ✓✓       │
  │ Branching       │ via Git  │ Native   │ Time     │ via Git  │
  │                 │          │          │ travel   │          │
  │ Pipeline def.   │ ✓        │          │          │          │
  │ Experiment trk  │ ✓        │          │          │          │
  │ Scale           │ Medium   │ Large    │ Large    │ Small    │
  │ Storage backend │ S3/GCS/  │ S3/GCS   │ S3/ADLS  │ Git host │
  │                 │ Azure/SSH│          │          │          │
  │ Learning curve  │ Low      │ Medium   │ Medium   │ Very low │
  └─────────────────┴──────────┴──────────┴──────────┴──────────┘

  DVC is the most popular for ML teams due to its Git integration
  and built-in pipeline support.
```

---

## Python: DVC API

```python
import dvc.api

# Get URL to a specific version of a file
url = dvc.api.get_url(
    path="data/train.csv",
    repo="https://github.com/myteam/ml-project",
    rev="data-v2"     # Git tag, branch, or commit
)

# Read file content directly (without checkout)
with dvc.api.open(
    "data/train.csv",
    repo="https://github.com/myteam/ml-project",
    rev="data-v1",
    mode="r"
) as f:
    import pandas as pd
    df = pd.read_csv(f)
    print(f"Data v1: {len(df)} rows")

# Get parameters from params.yaml
params = dvc.api.params_show()
print(f"Learning rate: {params['train']['lr']}")
print(f"Batch size: {params['train']['batch_size']}")

# Compare metrics across experiments
# $ dvc metrics show
# $ dvc plots show
```

---

## Revision Questions

1. **Why can't Git handle large ML datasets directly?**
2. **What does a .dvc file contain and what is its purpose?**
3. **How does `dvc repro` know which pipeline stages to rerun?**
4. **How do you switch between different data versions using DVC?**
5. **Compare DVC with Git LFS — when would you use each?**

---

[Previous: ../01-Introduction-to-MLOps/05-challenges-in-ml-production.md](../01-Introduction-to-MLOps/05-challenges-in-ml-production.md) | [Next: 02-data-validation.md](02-data-validation.md)
