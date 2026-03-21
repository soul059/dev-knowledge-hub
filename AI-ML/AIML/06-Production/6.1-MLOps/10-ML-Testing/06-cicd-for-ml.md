# 🚀 CI/CD for ML

> **Unit 10 · ML Testing · Chapter 6**
>
> How to automate ML testing and deployment — GitHub Actions for ML pipelines, automated test suites on every commit, model validation gates, promoting models through environments (dev → staging → prod), and CML (Continuous Machine Learning) for experiment tracking in CI.

---

## Table of Contents

1. [Chapter Overview](#chapter-overview)
2. [Why CI/CD for ML Is Different](#why-cicd-for-ml-is-different)
3. [GitHub Actions for ML](#github-actions-for-ml)
4. [Automated Testing Pipelines](#automated-testing-pipelines)
5. [Model Validation Gates](#model-validation-gates)
6. [Promoting Models Through Environments](#promoting-models-through-environments)
7. [CML: Continuous Machine Learning](#cml-continuous-machine-learning)
8. [Complete CI/CD Pipeline](#complete-cicd-pipeline)
9. [Summary](#summary)
10. [Revision Questions](#revision-questions)
11. [Navigation](#navigation)

---

## Chapter Overview

Traditional CI/CD tests code, builds artifacts, and deploys services. ML CI/CD must also test **data**, validate **model quality**, track **experiments**, and manage **large binary artifacts** (model weights). You can't just `git push` a 2 GB model file, and a passing unit test doesn't mean the model is any good. This chapter covers how to build CI/CD pipelines tailored for ML: GitHub Actions workflows that run tests, validate model performance, gate promotions on quality metrics, push models through dev → staging → production, and use **CML** (Continuous Machine Learning) to post training reports directly in pull requests.

```
Traditional CI/CD                     ML CI/CD
┌──────────────────────────┐         ┌──────────────────────────────────┐
│  Code change             │         │  Code change OR data change      │
│    │                     │         │    │                             │
│    ▼                     │         │    ▼                             │
│  Lint + Unit tests       │         │  Lint + Unit tests               │
│    │                     │         │  Data validation tests           │
│    ▼                     │         │    │                             │
│  Build artifact          │         │    ▼                             │
│    │                     │         │  Train model (or load cached)    │
│    ▼                     │         │    │                             │
│  Deploy                  │         │    ▼                             │
│                          │         │  Model validation gate           │
│  Done ✅                 │         │  (accuracy > threshold?)          │
│                          │         │    │                             │
│                          │         │    ▼                             │
│                          │         │  Push to model registry          │
│                          │         │    │                             │
│                          │         │    ▼                             │
│                          │         │  Deploy to staging → shadow → prod│
│                          │         │                                  │
│                          │         │  Done ✅                          │
└──────────────────────────┘         └──────────────────────────────────┘
```

---

## Why CI/CD for ML Is Different

### Additional Challenges

| Challenge | Traditional CI/CD | ML CI/CD |
|---|---|---|
| **What triggers a build** | Code commit | Code commit OR data change OR schedule |
| **Artifacts** | Docker image (~100 MB) | Docker image + model weights (100 MB–10 GB) |
| **Test duration** | Seconds–minutes | Minutes–hours (training + evaluation) |
| **Quality gate** | Tests pass/fail | Tests pass + model metrics above threshold |
| **Reproducibility** | Deterministic builds | Non-deterministic (random seeds, data order) |
| **Hardware** | Standard CPU runners | GPU runners for training/inference |
| **Versioning** | Code version (git) | Code version + data version + model version |
| **Rollback** | Redeploy previous image | Redeploy previous model + compatible code |

### ML CI/CD Dimensions

```
┌───────────────────────────────────────────────────────────────┐
│              Three Pipelines, Not One                         │
│                                                               │
│  ┌─────────────┐   ┌─────────────────┐   ┌───────────────┐   │
│  │  Code CI/CD │   │  Data Pipeline   │   │ Model CI/CD   │   │
│  │             │   │                 │   │               │   │
│  │  Lint       │   │  Validate data  │   │  Train        │   │
│  │  Unit test  │   │  Schema check   │   │  Evaluate     │   │
│  │  Build      │   │  Distribution   │   │  Compare to   │   │
│  │  Deploy     │   │  test           │   │  baseline     │   │
│  │             │   │  Transform      │   │  Register     │   │
│  │             │   │                 │   │  Deploy       │   │
│  └──────┬──────┘   └────────┬────────┘   └───────┬───────┘   │
│         │                   │                     │           │
│         └───────────────────┼─────────────────────┘           │
│                             │                                 │
│                    All three must coordinate                  │
└───────────────────────────────────────────────────────────────┘
```

---

## GitHub Actions for ML

### Basic ML Workflow

```yaml
# .github/workflows/ml-ci.yml
name: ML CI Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 6 * * 1'        # Weekly retrain (Monday 6 AM UTC)

jobs:
  lint-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Cache pip dependencies
        uses: actions/cache@v4
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('requirements.txt') }}

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Lint
        run: |
          ruff check src/ tests/
          mypy src/ --ignore-missing-imports

      - name: Unit tests (fast)
        run: pytest tests/ -m "not slow and not gpu" -v --tb=short

  data-validation:
    runs-on: ubuntu-latest
    needs: lint-and-test
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Download test data
        run: dvc pull data/test.parquet.dvc

      - name: Run data tests
        run: pytest tests/test_data/ -v --tb=short

  model-tests:
    runs-on: ubuntu-latest
    needs: lint-and-test
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Download model
        run: dvc pull models/model.joblib.dvc

      - name: Run model behavioral tests
        run: pytest tests/test_model/ -v --tb=short

      - name: Run integration tests
        run: pytest tests/test_integration/ -v --tb=short
```

### GPU Runner for Training

```yaml
# .github/workflows/train.yml
name: Model Training

on:
  workflow_dispatch:              # Manual trigger
    inputs:
      experiment_name:
        description: 'Experiment name'
        required: true
  schedule:
    - cron: '0 2 * * 0'          # Weekly Sunday 2 AM

jobs:
  train:
    runs-on: [self-hosted, gpu]   # Self-hosted GPU runner
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Pull training data
        run: dvc pull data/train.parquet.dvc

      - name: Train model
        env:
          MLFLOW_TRACKING_URI: ${{ secrets.MLFLOW_URI }}
          EXPERIMENT_NAME: ${{ github.event.inputs.experiment_name || 'scheduled' }}
        run: python scripts/train.py --experiment $EXPERIMENT_NAME

      - name: Evaluate model
        run: python scripts/evaluate.py --output metrics.json

      - name: Upload metrics artifact
        uses: actions/upload-artifact@v4
        with:
          name: model-metrics
          path: metrics.json

      - name: Upload model artifact
        uses: actions/upload-artifact@v4
        with:
          name: trained-model
          path: models/
```

---

## Automated Testing Pipelines

### Test Tier Structure

```
┌───────────────────────────────────────────────────────────────┐
│                  ML Test Pipeline Tiers                       │
│                                                               │
│  Tier 1: Every Commit (< 2 min)                              │
│  ├── Linting (ruff, mypy)                                     │
│  ├── Unit tests (preprocessing, features)                     │
│  └── Schema validation (Pandera)                              │
│                                                               │
│  Tier 2: Every PR (< 10 min)                                  │
│  ├── Data validation tests                                     │
│  ├── Model behavioral tests (MFT, INV, DIR)                   │
│  ├── Integration tests (pipeline end-to-end)                   │
│  └── API contract tests                                        │
│                                                               │
│  Tier 3: Before Merge (< 30 min)                              │
│  ├── Full model evaluation on test set                         │
│  ├── Bias/fairness tests                                       │
│  └── Performance benchmarks (latency)                          │
│                                                               │
│  Tier 4: Nightly / Weekly                                      │
│  ├── Full retrain + evaluation                                 │
│  ├── Stress tests                                              │
│  ├── Shadow mode comparison report                             │
│  └── Data distribution drift tests                             │
└───────────────────────────────────────────────────────────────┘
```

### Multi-Tier Workflow

```yaml
# .github/workflows/ml-test-tiers.yml
name: ML Test Tiers

on:
  push:
    branches: [main, 'feature/**']
  pull_request:
    branches: [main]

jobs:
  # ── Tier 1: Every commit ──────────────────────────────
  tier1-fast:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - run: pip install -r requirements.txt
      - name: Lint
        run: ruff check src/ tests/
      - name: Unit tests
        run: pytest tests/ -m "not slow and not gpu" --tb=short -q

  # ── Tier 2: Every PR ──────────────────────────────────
  tier2-pr:
    if: github.event_name == 'pull_request'
    needs: tier1-fast
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - run: pip install -r requirements.txt
      - name: Pull test data
        run: dvc pull data/test.parquet.dvc
      - name: Data + Model + Integration tests
        run: |
          pytest tests/test_data/ -v --tb=short
          pytest tests/test_model/ -v --tb=short
          pytest tests/test_integration/ -v --tb=short

  # ── Tier 3: Before merge ──────────────────────────────
  tier3-merge-gate:
    if: github.event_name == 'pull_request' && github.base_ref == 'main'
    needs: tier2-pr
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - run: pip install -r requirements.txt
      - name: Pull model and data
        run: dvc pull
      - name: Full model evaluation
        run: python scripts/evaluate.py --output metrics.json
      - name: Check model quality gate
        run: python scripts/check_quality_gate.py metrics.json
      - name: Bias tests
        run: pytest tests/test_model/test_bias.py -v --tb=short
```

---

## Model Validation Gates

### Quality Gate Script

```python
# scripts/check_quality_gate.py
"""
Model validation gate: blocks deployment if model doesn't meet
quality thresholds. Exit 0 = pass, exit 1 = fail.
"""
import json
import sys


THRESHOLDS = {
    "accuracy": 0.90,
    "f1_score": 0.85,
    "auc_roc": 0.90,
    "precision": 0.80,
    "recall": 0.80,
    "latency_p99_ms": 200,
}


def check_gate(metrics_path: str) -> tuple[bool, list[str]]:
    with open(metrics_path) as f:
        metrics = json.load(f)

    failures = []

    for metric, threshold in THRESHOLDS.items():
        if metric not in metrics:
            failures.append(f"Missing metric: {metric}")
            continue

        value = metrics[metric]
        if metric == "latency_p99_ms":
            if value > threshold:
                failures.append(
                    f"{metric}: {value:.1f} > {threshold} (max allowed)"
                )
        else:
            if value < threshold:
                failures.append(
                    f"{metric}: {value:.4f} < {threshold} (minimum required)"
                )

    return len(failures) == 0, failures


def main():
    if len(sys.argv) != 2:
        print("Usage: python check_quality_gate.py <metrics.json>")
        sys.exit(1)

    passed, failures = check_gate(sys.argv[1])

    if passed:
        print("✅ MODEL QUALITY GATE PASSED")
        print("   All metrics meet minimum thresholds.")
        sys.exit(0)
    else:
        print("❌ MODEL QUALITY GATE FAILED")
        for f in failures:
            print(f"   • {f}")
        sys.exit(1)


if __name__ == "__main__":
    main()
```

### Gate Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                Model Validation Gates                       │
│                                                             │
│  ┌───────────┐     ┌───────────┐     ┌───────────────────┐ │
│  │  Gate 1   │     │  Gate 2   │     │     Gate 3        │ │
│  │  Tests    │────►│  Metrics  │────►│  Comparison to    │ │
│  │  Pass?    │     │  Above    │     │  Current Prod     │ │
│  │           │     │  Threshold│     │  Model            │ │
│  └─────┬─────┘     └─────┬─────┘     └────────┬──────────┘ │
│        │                 │                     │            │
│    ❌ Block          ❌ Block             ❌ Block          │
│    CI fails          CI fails             CI fails          │
│                                                             │
│                      All Pass                               │
│                         │                                   │
│                         ▼                                   │
│               ┌───────────────────┐                         │
│               │  Push to Model    │                         │
│               │  Registry         │                         │
│               │  (with metadata)  │                         │
│               └───────────────────┘                         │
└─────────────────────────────────────────────────────────────┘
```

### Comparison Against Baseline

```python
# scripts/compare_to_baseline.py
"""
Compare candidate model to current production model.
Candidate must be at least as good (no regression).
"""
import json
import sys


def compare(candidate_path: str, baseline_path: str) -> tuple[bool, list[str]]:
    with open(candidate_path) as f:
        candidate = json.load(f)
    with open(baseline_path) as f:
        baseline = json.load(f)

    MAX_REGRESSION = 0.02    # Allow max 2% regression
    failures = []

    for metric in ["accuracy", "f1_score", "auc_roc"]:
        if metric in candidate and metric in baseline:
            diff = candidate[metric] - baseline[metric]
            if diff < -MAX_REGRESSION:
                failures.append(
                    f"{metric}: {candidate[metric]:.4f} vs baseline "
                    f"{baseline[metric]:.4f} (regression: {diff:+.4f})"
                )
            else:
                status = "improved" if diff > 0 else "comparable"
                print(f"  {metric}: {candidate[metric]:.4f} vs "
                      f"{baseline[metric]:.4f} ({status}: {diff:+.4f})")

    return len(failures) == 0, failures


if __name__ == "__main__":
    passed, failures = compare(sys.argv[1], sys.argv[2])
    if passed:
        print("✅ No regression vs baseline")
        sys.exit(0)
    else:
        print("❌ REGRESSION DETECTED:")
        for f in failures:
            print(f"   • {f}")
        sys.exit(1)
```

---

## Promoting Models Through Environments

### Environment Promotion Flow

```
┌──────────┐      ┌──────────┐      ┌──────────┐      ┌──────────┐
│   Dev    │      │ Staging  │      │  Shadow  │      │   Prod   │
│          │      │          │      │          │      │          │
│ Auto on  │      │ Auto on  │      │ Manual   │      │ Manual   │
│ PR merge │─────►│ merge to │─────►│ trigger  │─────►│ approval │
│          │      │ main     │      │          │      │          │
│ Tests:   │      │ Tests:   │      │ Tests:   │      │ Tests:   │
│ Tier 1+2 │      │ Tier 3   │      │ Shadow   │      │ Canary   │
│          │      │ + staging│      │ analysis │      │ metrics  │
│          │      │ smoke    │      │          │      │          │
└──────────┘      └──────────┘      └──────────┘      └──────────┘
   Auto              Auto              Manual            Manual
                                       + gates           + approval
```

### Deployment Workflow

```yaml
# .github/workflows/deploy.yml
name: Model Deployment

on:
  workflow_run:
    workflows: ["ML Test Tiers"]
    types: [completed]
    branches: [main]

jobs:
  # ── Deploy to Staging ─────────────────────────────────
  deploy-staging:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/checkout@v4

      - name: Download model artifact
        uses: actions/download-artifact@v4
        with:
          name: trained-model
          run-id: ${{ github.event.workflow_run.id }}

      - name: Push model to registry
        run: |
          python scripts/push_model.py \
            --model-path models/model.joblib \
            --registry ${{ secrets.MODEL_REGISTRY_URL }} \
            --stage staging \
            --version ${{ github.sha }}

      - name: Deploy to staging
        run: |
          kubectl set image deployment/model-server \
            model=registry.example.com/model:${{ github.sha }} \
            --namespace staging

      - name: Run staging tests
        run: |
          sleep 30   # Wait for rollout
          pytest tests/test_staging.py -v --tb=short

  # ── Promote to Production ─────────────────────────────
  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://model-api.example.com
    steps:
      - uses: actions/checkout@v4

      - name: Promote model in registry
        run: |
          python scripts/push_model.py \
            --version ${{ github.sha }} \
            --stage production

      - name: Canary deploy (5%)
        run: |
          kubectl apply -f k8s/canary-5pct.yaml --namespace production

      - name: Wait and check canary metrics
        run: |
          sleep 300   # 5 minutes
          python scripts/check_canary.py \
            --namespace production \
            --error-threshold 0.01 \
            --latency-threshold 200

      - name: Full rollout
        run: |
          kubectl set image deployment/model-server \
            model=registry.example.com/model:${{ github.sha }} \
            --namespace production
```

---

## CML: Continuous Machine Learning

**CML** (by Iterative.ai, the DVC team) posts training metrics, plots, and comparisons directly into pull requests — so reviewers can see model impact without leaving GitHub.

### How CML Works

```
┌───────────────────────────────────────────────────────────────┐
│                        CML Workflow                           │
│                                                               │
│  1. Developer pushes code/data change                        │
│  2. GitHub Actions trains model                               │
│  3. CML generates report:                                     │
│     • Metrics table (accuracy, F1, AUC)                       │
│     • Comparison vs main branch                               │
│     • Confusion matrix image                                  │
│     • ROC curve image                                         │
│  4. CML posts report as PR comment                            │
│                                                               │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │  Pull Request #42: "Update feature engineering"         │  │
│  │                                                         │  │
│  │  🤖 CML Report                                          │  │
│  │  ┌───────────────────────────────────────────────────┐  │  │
│  │  │ Metric     │ main     │ this PR  │ Diff           │  │  │
│  │  │ accuracy   │ 0.9123   │ 0.9245   │ +0.0122 ✅     │  │  │
│  │  │ f1_score   │ 0.8834   │ 0.8901   │ +0.0067 ✅     │  │  │
│  │  │ auc_roc    │ 0.9456   │ 0.9512   │ +0.0056 ✅     │  │  │
│  │  │ latency_ms │ 23       │ 25       │ +2      ✅     │  │  │
│  │  └───────────────────────────────────────────────────┘  │  │
│  │                                                         │  │
│  │  📊 [Confusion Matrix] [ROC Curve] [PR Curve]           │  │
│  └─────────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────────┘
```

### CML Workflow File

```yaml
# .github/workflows/cml-report.yml
name: CML Report

on:
  pull_request:
    branches: [main]

jobs:
  train-and-report:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - uses: iterative/setup-cml@v2

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Pull data with DVC
        run: dvc pull

      - name: Train model
        run: python scripts/train.py --output models/candidate.joblib

      - name: Evaluate model
        run: |
          python scripts/evaluate.py \
            --model models/candidate.joblib \
            --output metrics.json \
            --plots-dir plots/

      - name: Generate CML report
        env:
          REPO_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          # Create report
          echo "## 🤖 Model Performance Report" > report.md
          echo "" >> report.md

          # Metrics table
          echo "### Metrics" >> report.md
          python scripts/format_metrics.py metrics.json >> report.md
          echo "" >> report.md

          # Comparison with main branch
          echo "### Comparison vs Main" >> report.md
          git fetch origin main
          dvc metrics diff main --md >> report.md || echo "No baseline" >> report.md
          echo "" >> report.md

          # Plots
          echo "### Plots" >> report.md
          cml-publish plots/confusion_matrix.png --md >> report.md
          cml-publish plots/roc_curve.png --md >> report.md
          cml-publish plots/feature_importance.png --md >> report.md

          # Post as PR comment
          cml comment create report.md
```

### Metrics Formatting Script

```python
# scripts/format_metrics.py
"""Format metrics.json as a markdown table for CML reports."""
import json
import sys


def format_metrics(metrics_path: str) -> str:
    with open(metrics_path) as f:
        metrics = json.load(f)

    lines = [
        "| Metric | Value |",
        "|--------|-------|",
    ]
    for key, value in sorted(metrics.items()):
        if isinstance(value, float):
            lines.append(f"| **{key}** | {value:.4f} |")
        else:
            lines.append(f"| **{key}** | {value} |")

    return "\n".join(lines)


if __name__ == "__main__":
    print(format_metrics(sys.argv[1]))
```

### CML with GPU Runners

```yaml
# .github/workflows/cml-gpu.yml
name: CML GPU Training

on:
  pull_request:
    branches: [main]

jobs:
  train-gpu:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: iterative/setup-cml@v2

      - name: Launch GPU runner
        env:
          REPO_TOKEN: ${{ secrets.PERSONAL_ACCESS_TOKEN }}
        run: |
          cml runner launch \
            --cloud=aws \
            --cloud-region=us-east-1 \
            --cloud-type=g4dn.xlarge \
            --labels=cml-gpu

  report:
    needs: train-gpu
    runs-on: [self-hosted, cml-gpu]
    steps:
      - uses: actions/checkout@v4

      - name: Train on GPU
        run: python scripts/train.py --device cuda

      - name: Report
        env:
          REPO_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          echo "## GPU Training Report" > report.md
          python scripts/format_metrics.py metrics.json >> report.md
          cml comment create report.md
```

---

## Complete CI/CD Pipeline

### End-to-End Architecture

```
┌──────────────────────────────────────────────────────────────┐
│              Complete ML CI/CD Pipeline                       │
│                                                              │
│  Developer pushes code                                       │
│       │                                                      │
│       ▼                                                      │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ GitHub Actions                                          │ │
│  │                                                         │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐             │ │
│  │  │  Lint    │──►│  Unit    │──►│  Data    │             │ │
│  │  │  + Type  │  │  Tests   │  │  Tests   │             │ │
│  │  └──────────┘  └──────────┘  └─────┬────┘             │ │
│  │                                     │                   │ │
│  │                                     ▼                   │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐             │ │
│  │  │ Model    │──►│ Quality  │──►│  CML     │             │ │
│  │  │ Behavior │  │  Gate    │  │ Report   │             │ │
│  │  │ Tests    │  │          │  │ on PR    │             │ │
│  │  └──────────┘  └──────────┘  └──────────┘             │ │
│  │                      │                                  │ │
│  │                  ❌ = Block PR                           │ │
│  │                  ✅ = Continue                           │ │
│  └──────────────────────┼──────────────────────────────────┘ │
│                         │                                    │
│                    Merge to main                             │
│                         │                                    │
│  ┌──────────────────────┼──────────────────────────────────┐ │
│  │                      ▼                                  │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────┐ │ │
│  │  │ Push to  │──►│ Deploy   │──►│ Shadow   │──►│ Prod   │ │ │
│  │  │ Registry │  │ Staging  │  │ Mode     │  │ Deploy │ │ │
│  │  └──────────┘  └──────────┘  └──────────┘  └────────┘ │ │
│  │                                                         │ │
│  │  Model Registry    Staging       1-2 weeks    Manual    │ │
│  │  (MLflow/DVC)      smoke tests   comparison   approval  │ │
│  └─────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────┘
```

### Pipeline Configuration File

```python
# ci/pipeline_config.py
"""
Central configuration for ML CI/CD pipeline.
All thresholds and environment configs in one place.
"""

QUALITY_GATES = {
    "tier1": {
        "trigger": "every_commit",
        "max_duration_minutes": 2,
        "tests": ["lint", "unit_tests", "schema_validation"],
    },
    "tier2": {
        "trigger": "pull_request",
        "max_duration_minutes": 10,
        "tests": ["data_validation", "model_behavioral", "integration"],
    },
    "tier3": {
        "trigger": "merge_to_main",
        "max_duration_minutes": 30,
        "tests": ["full_evaluation", "bias_tests", "latency_benchmark"],
    },
}

MODEL_THRESHOLDS = {
    "accuracy": {"min": 0.90, "regression_tolerance": 0.02},
    "f1_score": {"min": 0.85, "regression_tolerance": 0.02},
    "auc_roc": {"min": 0.90, "regression_tolerance": 0.01},
    "latency_p99_ms": {"max": 200},
}

ENVIRONMENTS = {
    "dev": {
        "auto_deploy": True,
        "approval_required": False,
        "tests_required": ["tier1", "tier2"],
    },
    "staging": {
        "auto_deploy": True,
        "approval_required": False,
        "tests_required": ["tier1", "tier2", "tier3"],
    },
    "production": {
        "auto_deploy": False,
        "approval_required": True,
        "tests_required": ["tier1", "tier2", "tier3", "shadow_analysis"],
        "canary_percentage": 5,
        "canary_duration_minutes": 60,
    },
}
```

---

## Summary

| Concept | Key Takeaway |
|---|---|
| **ML CI/CD differences** | Must handle data changes, large artifacts, GPU runners, non-determinism |
| **GitHub Actions for ML** | Multi-job workflows with caching, DVC integration, and test tiers |
| **Test tiers** | Tier 1 (every commit, fast) → Tier 2 (PR) → Tier 3 (merge) → Tier 4 (nightly) |
| **Model validation gates** | Block deployment if metrics drop below thresholds or regress vs baseline |
| **Environment promotion** | Dev → Staging → Shadow → Prod; auto-deploy to staging, manual approval for prod |
| **CML** | Posts training metrics, plots, and comparisons directly in PR comments |
| **Complete pipeline** | Lint → Test → Train → Gate → Registry → Staging → Shadow → Prod |

> **Rule of thumb:** Every model change should be as reviewable as every code change. CML puts metrics in the PR, quality gates enforce minimum standards, and environment promotion ensures the model is validated at each stage before reaching users.

---

## Revision Questions

1. **Explain** three ways ML CI/CD differs from traditional software CI/CD. How does each difference affect pipeline design?

2. **Design** a GitHub Actions workflow for a team that trains models on GPU but runs tests on CPU. Include DVC for data versioning, three test tiers, and a quality gate.

3. **A model** passes the quality gate (all metrics above threshold) but the CML report shows a 5% regression in recall compared to the main branch. Should the PR be merged? Design a gate that handles this scenario.

4. **Compare** the following promotion strategies: direct deploy to production, blue/green, canary, and shadow → canary. When would you choose each, and what are the risks?

5. **Your team** uses MLflow for experiment tracking and GitHub Actions for CI. Design a pipeline that automatically registers the best model from each training run, compares it to the current production model, and opens a PR for promotion if it's better.

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Shadow Mode Testing](./05-shadow-mode-testing.md) | [📂 ML Testing](./README.md) | [Bias Detection →](../11-Responsible-AI-in-Production/01-bias-detection.md) |

---

> **End of Unit 10 — ML Testing.** Continue to [Unit 11 — Responsible AI in Production](../11-Responsible-AI-in-Production/01-bias-detection.md) for bias detection and fairness in deployed systems.
