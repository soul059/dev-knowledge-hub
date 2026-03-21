# Model Promotion

## Overview

Model promotion is the process of moving a validated model from development through staging to production. A well-defined promotion workflow includes automated testing gates, approval processes, and safe deployment strategies. This chapter covers promotion patterns, approval workflows, and automation.

---

## Promotion Pipeline

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐ │
  │  │  None    │──►│ Staging  │──►│Production│──►│ Archived │ │
  │  │(registered)  │(testing) │   │(serving) │   │(retired) │ │
  │  └──────────┘   └────┬─────┘   └──────────┘   └──────────┘ │
  │                       │                                      │
  │              ┌────────┴────────────┐                         │
  │              │   Promotion Gates   │                         │
  │              │                      │                         │
  │              │ ✓ Metrics > threshold │                        │
  │              │ ✓ Better than current│                         │
  │              │ ✓ Fairness check     │                         │
  │              │ ✓ Latency check      │                         │
  │              │ ✓ Integration tests  │                         │
  │              │ ✓ Approval (manual)  │                         │
  │              │ ✓ Shadow deployment  │                         │
  │              └─────────────────────┘                         │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Automated Promotion Workflow

```python
import mlflow
from mlflow.tracking import MlflowClient

client = MlflowClient()

MODEL_NAME = "churn_classifier"
F1_MIN = 0.82
LATENCY_MAX_MS = 50


def promote_model(version: int):
    """Automated model promotion with quality gates."""
    
    # Get model version details
    mv = client.get_model_version(MODEL_NAME, str(version))
    run = client.get_run(mv.run_id)
    
    f1 = run.data.metrics.get("f1_score", 0)
    accuracy = run.data.metrics.get("accuracy", 0)
    latency = run.data.metrics.get("inference_latency_ms", 999)
    
    print(f"Evaluating v{version}: F1={f1:.3f}, lat={latency:.0f}ms")
    
    # Gate 1: Performance threshold
    if f1 < F1_MIN:
        print(f"❌ FAIL: F1 {f1:.3f} < {F1_MIN}")
        return False
    
    # Gate 2: Latency constraint
    if latency > LATENCY_MAX_MS:
        print(f"❌ FAIL: Latency {latency}ms > {LATENCY_MAX_MS}ms")
        return False
    
    # Gate 3: Better than current production
    prod_versions = client.get_latest_versions(MODEL_NAME, stages=["Production"])
    if prod_versions:
        prod_run = client.get_run(prod_versions[0].run_id)
        prod_f1 = prod_run.data.metrics.get("f1_score", 0)
        if f1 <= prod_f1:
            print(f"❌ FAIL: Not better than production (prod F1={prod_f1:.3f})")
            return False
    
    # All gates passed — promote!
    client.transition_model_version_stage(
        name=MODEL_NAME,
        version=str(version),
        stage="Staging",
    )
    print(f"✅ v{version} → Staging")
    
    # After staging tests pass (could be in a separate step):
    client.transition_model_version_stage(
        name=MODEL_NAME,
        version=str(version),
        stage="Production",
        archive_existing_versions=True,
    )
    print(f"✅ v{version} → Production")
    return True


# Integrate with CI/CD (GitHub Actions, Jenkins)
# On model registration webhook → run promote_model()
```

---

## Promotion Patterns

```
  ┌──────────────────┬──────────────────────────────────────┐
  │ Pattern          │ Description                          │
  ├──────────────────┼──────────────────────────────────────┤
  │ Fully automated  │ All gates automated, no human review │
  │                  │ Best for: low-risk, frequent updates │
  ├──────────────────┼──────────────────────────────────────┤
  │ Human-in-loop    │ Automated gates + manual approval    │
  │                  │ Best for: high-risk, regulated       │
  ├──────────────────┼──────────────────────────────────────┤
  │ Shadow first     │ Deploy alongside production (no      │
  │                  │ impact), compare before switching    │
  ├──────────────────┼──────────────────────────────────────┤
  │ Canary promotion │ Gradually shift traffic to new model │
  │                  │ 1% → 10% → 50% → 100%              │
  ├──────────────────┼──────────────────────────────────────┤
  │ Champion/        │ Old model (champion) vs new model    │
  │ Challenger       │ (challenger) — A/B test to decide   │
  └──────────────────┴──────────────────────────────────────┘
```

---

## CI/CD Integration

```yaml
# GitHub Actions: auto-promote on model registration
# .github/workflows/model-promotion.yml
name: Model Promotion
on:
  workflow_dispatch:
    inputs:
      model_version:
        description: "Model version to promote"
        required: true

jobs:
  promote:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run promotion gates
        run: |
          python scripts/promotion_gates.py \
            --model churn_classifier \
            --version ${{ github.event.inputs.model_version }}
      
      - name: Run integration tests
        run: pytest tests/integration/ -v
      
      - name: Promote to production
        if: success()
        run: |
          python scripts/promote.py \
            --model churn_classifier \
            --version ${{ github.event.inputs.model_version }} \
            --stage Production
```

---

## Revision Questions

1. **What quality gates should be included in a promotion pipeline?**
2. **What is the champion/challenger promotion pattern?**
3. **When should you use human-in-the-loop vs. fully automated promotion?**
4. **How can CI/CD systems automate model promotion?**

---

[Previous: 04-model-lineage.md](04-model-lineage.md) | [Next: ../06-Model-Serving/01-serving-patterns.md](../06-Model-Serving/01-serving-patterns.md)
