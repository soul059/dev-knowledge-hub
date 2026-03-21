# 🏛️ Model Governance

> **Unit 11 · Responsible AI in Production · Chapter 4**
>
> How to govern ML models across their lifecycle — model documentation requirements, approval workflows, audit trails, model risk management frameworks, maintaining a model inventory, and defining roles and responsibilities for responsible AI.

---

## Table of Contents

1. [Chapter Overview](#chapter-overview)
2. [Why Model Governance Matters](#why-model-governance-matters)
3. [Model Documentation Requirements](#model-documentation-requirements)
4. [Approval Workflows](#approval-workflows)
5. [Audit Trails](#audit-trails)
6. [Model Risk Management](#model-risk-management)
7. [Model Inventory](#model-inventory)
8. [Roles and Responsibilities](#roles-and-responsibilities)
9. [Summary](#summary)
10. [Revision Questions](#revision-questions)
11. [Navigation](#navigation)

---

## Chapter Overview

Building a model is the easy part. Governing it — knowing what models are deployed, who approved them, whether they're still fit for purpose, and what happens when they fail — is where most organizations struggle. Without governance, models become invisible risks: undocumented, unmonitored, and unaccountable. This chapter covers the frameworks, processes, and tooling needed for **model governance** at scale: what documentation every model needs, how to build approval workflows that don't bottleneck delivery, maintaining audit trails for regulatory compliance, model risk management (MRM) frameworks, keeping a live model inventory, and defining who is responsible for what.

```
Model Governance Lifecycle
┌───────────────────────────────────────────────────────────────┐
│                                                               │
│  Development  →  Validation  →  Approval  →  Deployment       │
│       │              │            │              │             │
│  Model card     Independent    Sign-off      Registry         │
│  Documentation  review        workflow       + Inventory      │
│  Risk tier                                                    │
│                                                               │
│  Deployment  →  Monitoring  →  Review  →  Retirement          │
│       │              │           │            │                │
│  Audit trail    Performance   Periodic     Deprecation        │
│  Lineage        + Fairness    re-validation plan              │
│                 alerts        (annual)                         │
└───────────────────────────────────────────────────────────────┘
```

---

## Why Model Governance Matters

### The Cost of Ungoverned Models

| Risk | Without Governance | With Governance |
|---|---|---|
| **Regulatory fine** | No evidence of controls → maximum penalty | Documented audit trail → defensible position |
| **Model failure** | Unknown models break silently | Monitored, alerting, rollback plan |
| **Knowledge loss** | Developer leaves, no one knows how model works | Model card + documentation persists |
| **Bias incident** | Discovered by press or lawsuit | Caught by fairness monitoring before harm |
| **Redundant models** | 12 teams build same fraud model | Model inventory prevents duplication |
| **Stale models** | Model trained on 2019 data still in prod in 2025 | Mandatory re-validation schedule |

### Governance Maturity Levels

```
┌───────────────────────────────────────────────────────────────┐
│            Model Governance Maturity Levels                    │
│                                                               │
│  Level 0 — Ad Hoc                                             │
│  • No inventory of deployed models                            │
│  • No approval process                                        │
│  • "Push to prod and pray"                                    │
│                                                               │
│  Level 1 — Basic                                              │
│  • Model registry exists                                      │
│  • Basic documentation (README)                               │
│  • Manual approval via Slack/email                            │
│                                                               │
│  Level 2 — Standardized                                       │
│  • Model cards for all models                                 │
│  • Formal approval workflow                                   │
│  • Automated testing gates                                    │
│  • Model inventory with owners                                │
│                                                               │
│  Level 3 — Managed                                            │
│  • Risk-tiered governance                                     │
│  • Independent model validation                               │
│  • Continuous monitoring + alerting                            │
│  • Periodic re-validation                                     │
│                                                               │
│  Level 4 — Optimized                                          │
│  • Automated compliance reporting                             │
│  • Real-time model inventory                                  │
│  • Governance-as-code                                         │
│  • Cross-org model sharing with controls                      │
└───────────────────────────────────────────────────────────────┘
```

---

## Model Documentation Requirements

### Documentation Checklist

```
┌───────────────────────────────────────────────────────────────┐
│         Model Documentation Checklist                         │
│                                                               │
│  ☐ Model card (see Chapter 3)                                 │
│  ☐ Business justification and intended use                    │
│  ☐ Training data description + lineage                        │
│  ☐ Feature documentation (name, type, source, engineering)    │
│  ☐ Architecture / algorithm description                       │
│  ☐ Hyperparameter choices + tuning methodology                │
│  ☐ Performance metrics (overall + per-group)                  │
│  ☐ Fairness audit results                                     │
│  ☐ Limitations and known failure modes                        │
│  ☐ Monitoring plan (what metrics, what thresholds)            │
│  ☐ Rollback plan                                              │
│  ☐ Data privacy assessment                                    │
│  ☐ Owner and escalation contacts                              │
│  ☐ Re-validation schedule                                     │
│  ☐ Decommissioning criteria                                   │
└───────────────────────────────────────────────────────────────┘
```

### Automated Documentation Generator

```python
# governance/doc_generator.py
"""Auto-generate model documentation from model artifacts."""
import json
from datetime import datetime
from dataclasses import dataclass, field, asdict
from pathlib import Path
from typing import Optional


@dataclass
class ModelDocumentation:
    """Complete model documentation for governance."""

    # ── Identity ─────────────────────────────────────────
    model_name: str = ""
    model_version: str = ""
    model_id: str = ""
    risk_tier: str = ""  # "low", "medium", "high", "critical"
    owner: str = ""
    team: str = ""
    created_date: str = ""
    approved_date: Optional[str] = None
    approved_by: Optional[str] = None

    # ── Business Context ─────────────────────────────────
    business_justification: str = ""
    intended_use: str = ""
    out_of_scope_uses: list[str] = field(default_factory=list)
    downstream_consumers: list[str] = field(default_factory=list)

    # ── Technical Details ────────────────────────────────
    algorithm: str = ""
    framework: str = ""
    features: list[dict] = field(default_factory=list)
    hyperparameters: dict = field(default_factory=dict)
    training_data_source: str = ""
    training_data_size: int = 0
    training_data_date_range: str = ""

    # ── Performance ──────────────────────────────────────
    metrics: dict = field(default_factory=dict)
    per_group_metrics: dict = field(default_factory=dict)
    fairness_audit: dict = field(default_factory=dict)

    # ── Operations ───────────────────────────────────────
    monitoring_plan: dict = field(default_factory=dict)
    rollback_plan: str = ""
    revalidation_schedule: str = ""
    decommission_criteria: list[str] = field(default_factory=list)

    # ── Compliance ───────────────────────────────────────
    data_privacy_assessment: str = ""
    regulatory_requirements: list[str] = field(default_factory=list)
    limitations: list[str] = field(default_factory=list)
    ethical_considerations: list[str] = field(default_factory=list)


def generate_doc_from_artifacts(
    model_path: str,
    metrics_path: str,
    fairness_path: str,
    config_path: str,
) -> ModelDocumentation:
    """
    Generate documentation by pulling from existing artifacts.

    In practice, this reads from model registry metadata,
    experiment tracking, and CI/CD outputs.
    """
    config = json.loads(Path(config_path).read_text())
    metrics = json.loads(Path(metrics_path).read_text())
    fairness = json.loads(Path(fairness_path).read_text())

    doc = ModelDocumentation(
        model_name=config.get("model_name", ""),
        model_version=config.get("version", ""),
        model_id=config.get("model_id", ""),
        risk_tier=config.get("risk_tier", "medium"),
        owner=config.get("owner", ""),
        team=config.get("team", ""),
        created_date=datetime.utcnow().isoformat(),
        algorithm=config.get("algorithm", ""),
        framework=config.get("framework", ""),
        features=config.get("features", []),
        hyperparameters=config.get("hyperparameters", {}),
        training_data_source=config.get("data_source", ""),
        training_data_size=config.get("data_size", 0),
        metrics=metrics,
        fairness_audit=fairness,
        intended_use=config.get("intended_use", ""),
    )

    return doc


def save_documentation(doc: ModelDocumentation, output_dir: str) -> Path:
    """Save documentation as JSON."""
    path = Path(output_dir) / f"{doc.model_name}_{doc.model_version}_doc.json"
    path.parent.mkdir(parents=True, exist_ok=True)
    path.write_text(json.dumps(asdict(doc), indent=2, default=str))
    return path
```

---

## Approval Workflows

### Risk-Tiered Approval

```
┌───────────────────────────────────────────────────────────────┐
│          Risk-Tiered Model Approval Workflow                  │
│                                                               │
│  Risk Tier     │ Approval Required     │ Review Scope          │
│  ──────────────┼───────────────────────┼─────────────────────  │
│  Low           │ Peer review           │ Code review +         │
│  (internal     │ (1 ML engineer)       │ automated tests       │
│   dashboards)  │                       │                       │
│  ──────────────┼───────────────────────┼─────────────────────  │
│  Medium        │ Tech lead +           │ + Model card review   │
│  (customer-    │ product owner         │ + Fairness audit      │
│   facing)      │                       │ + Performance review  │
│  ──────────────┼───────────────────────┼─────────────────────  │
│  High          │ Tech lead +           │ + Independent         │
│  (financial    │ ML governance board   │   validation          │
│   decisions)   │                       │ + Legal review        │
│  ──────────────┼───────────────────────┼─────────────────────  │
│  Critical      │ Governance board +    │ + External audit      │
│  (regulated    │ C-level + legal +     │ + Regulatory filing   │
│   industry)    │ compliance            │ + Board reporting     │
└───────────────────────────────────────────────────────────────┘
```

### Approval Workflow Engine

```python
# governance/approval_workflow.py
"""Model approval workflow engine with risk-tiered reviews."""
from dataclasses import dataclass, field
from datetime import datetime
from enum import Enum
from typing import Optional
import json


class RiskTier(Enum):
    LOW = "low"
    MEDIUM = "medium"
    HIGH = "high"
    CRITICAL = "critical"


class ApprovalStatus(Enum):
    PENDING = "pending"
    IN_REVIEW = "in_review"
    APPROVED = "approved"
    REJECTED = "rejected"
    REVOKED = "revoked"


@dataclass
class ApprovalStep:
    role: str
    status: ApprovalStatus = ApprovalStatus.PENDING
    reviewer: Optional[str] = None
    reviewed_at: Optional[str] = None
    comments: str = ""


@dataclass
class ApprovalRequest:
    model_name: str
    model_version: str
    risk_tier: RiskTier
    requester: str
    created_at: str = field(default_factory=lambda: datetime.utcnow().isoformat())
    status: ApprovalStatus = ApprovalStatus.PENDING
    steps: list[ApprovalStep] = field(default_factory=list)
    automated_checks: dict = field(default_factory=dict)


# Required approval steps per risk tier
APPROVAL_MATRIX = {
    RiskTier.LOW: ["ml_engineer"],
    RiskTier.MEDIUM: ["ml_engineer", "tech_lead", "product_owner"],
    RiskTier.HIGH: ["ml_engineer", "tech_lead", "governance_board", "legal"],
    RiskTier.CRITICAL: [
        "ml_engineer", "tech_lead", "governance_board",
        "legal", "compliance", "executive",
    ],
}

# Automated checks required per risk tier
REQUIRED_CHECKS = {
    RiskTier.LOW: ["unit_tests", "integration_tests"],
    RiskTier.MEDIUM: ["unit_tests", "integration_tests", "fairness_audit", "model_card"],
    RiskTier.HIGH: [
        "unit_tests", "integration_tests", "fairness_audit",
        "model_card", "independent_validation", "security_review",
    ],
    RiskTier.CRITICAL: [
        "unit_tests", "integration_tests", "fairness_audit",
        "model_card", "independent_validation", "security_review",
        "external_audit", "regulatory_review",
    ],
}


def create_approval_request(
    model_name: str,
    model_version: str,
    risk_tier: RiskTier,
    requester: str,
    automated_check_results: dict,
) -> ApprovalRequest:
    """Create an approval request with the right steps for the risk tier."""
    # Verify automated checks passed
    required = REQUIRED_CHECKS[risk_tier]
    missing = [c for c in required if c not in automated_check_results]
    failed = [c for c, v in automated_check_results.items() if not v.get("passed")]

    if missing:
        raise ValueError(f"Missing required checks: {missing}")
    if failed:
        raise ValueError(f"Failed automated checks: {failed}")

    steps = [ApprovalStep(role=role) for role in APPROVAL_MATRIX[risk_tier]]

    return ApprovalRequest(
        model_name=model_name,
        model_version=model_version,
        risk_tier=risk_tier,
        requester=requester,
        steps=steps,
        automated_checks=automated_check_results,
    )


def approve_step(
    request: ApprovalRequest,
    role: str,
    reviewer: str,
    comments: str = "",
) -> ApprovalRequest:
    """Approve a specific step in the workflow."""
    for step in request.steps:
        if step.role == role and step.status == ApprovalStatus.PENDING:
            step.status = ApprovalStatus.APPROVED
            step.reviewer = reviewer
            step.reviewed_at = datetime.utcnow().isoformat()
            step.comments = comments
            break
    else:
        raise ValueError(f"No pending step for role: {role}")

    # Check if all steps are approved
    if all(s.status == ApprovalStatus.APPROVED for s in request.steps):
        request.status = ApprovalStatus.APPROVED

    return request


def reject_step(
    request: ApprovalRequest,
    role: str,
    reviewer: str,
    reason: str,
) -> ApprovalRequest:
    """Reject a step, blocking the entire approval."""
    for step in request.steps:
        if step.role == role:
            step.status = ApprovalStatus.REJECTED
            step.reviewer = reviewer
            step.reviewed_at = datetime.utcnow().isoformat()
            step.comments = reason
            break

    request.status = ApprovalStatus.REJECTED
    return request


# ── Usage ────────────────────────────────────────────────
if __name__ == "__main__":
    checks = {
        "unit_tests": {"passed": True, "details": "47/47 passed"},
        "integration_tests": {"passed": True, "details": "12/12 passed"},
        "fairness_audit": {"passed": True, "dp_diff": 0.07},
        "model_card": {"passed": True, "completeness": "100%"},
    }

    request = create_approval_request(
        model_name="loan_model",
        model_version="3.2.1",
        risk_tier=RiskTier.MEDIUM,
        requester="alice@example.com",
        automated_check_results=checks,
    )

    print(f"Request status: {request.status.value}")
    print(f"Steps required: {[s.role for s in request.steps]}")

    # Simulate approvals
    approve_step(request, "ml_engineer", "bob@example.com", "LGTM")
    approve_step(request, "tech_lead", "carol@example.com", "Metrics look good")
    approve_step(request, "product_owner", "dave@example.com", "Business approved")

    print(f"\nFinal status: {request.status.value}")
```

---

## Audit Trails

### What to Log

```
┌───────────────────────────────────────────────────────────────┐
│               Model Audit Trail Events                        │
│                                                               │
│  Lifecycle Event          │  What to Capture                  │
│  ─────────────────────────┼─────────────────────────────────  │
│  Model created            │  Who, when, from what experiment  │
│  Model registered         │  Registry, version, artifacts     │
│  Approval requested       │  Requester, risk tier, checks     │
│  Approval step completed  │  Reviewer, decision, comments     │
│  Model deployed           │  Environment, strategy, config    │
│  Configuration changed    │  What changed, who, why           │
│  Alert fired              │  Metric, value, threshold, action │
│  Model re-validated       │  Results, pass/fail, reviewer     │
│  Model rolled back        │  Reason, who triggered, to what   │
│  Model decommissioned     │  Reason, replacement, data fate   │
└───────────────────────────────────────────────────────────────┘
```

### Immutable Audit Logger

```python
# governance/audit_trail.py
"""Immutable audit trail for model governance events."""
import hashlib
import json
import time
from dataclasses import dataclass, asdict, field
from pathlib import Path
from typing import Optional


@dataclass
class AuditEvent:
    event_type: str
    model_name: str
    model_version: str
    actor: str
    timestamp: float = field(default_factory=time.time)
    details: dict = field(default_factory=dict)
    previous_hash: str = ""
    event_hash: str = ""

    def compute_hash(self) -> str:
        """Compute hash of this event for tamper detection."""
        payload = json.dumps(
            {
                "event_type": self.event_type,
                "model_name": self.model_name,
                "model_version": self.model_version,
                "actor": self.actor,
                "timestamp": self.timestamp,
                "details": self.details,
                "previous_hash": self.previous_hash,
            },
            sort_keys=True,
        )
        return hashlib.sha256(payload.encode()).hexdigest()


class AuditTrail:
    """Append-only, hash-chained audit trail for model events."""

    def __init__(self, log_path: str):
        self.log_path = Path(log_path)
        self.log_path.parent.mkdir(parents=True, exist_ok=True)
        self._last_hash = "genesis"

    def record(
        self,
        event_type: str,
        model_name: str,
        model_version: str,
        actor: str,
        details: Optional[dict] = None,
    ) -> AuditEvent:
        """Record an immutable audit event."""
        event = AuditEvent(
            event_type=event_type,
            model_name=model_name,
            model_version=model_version,
            actor=actor,
            details=details or {},
            previous_hash=self._last_hash,
        )
        event.event_hash = event.compute_hash()
        self._last_hash = event.event_hash

        # Append to log file (append-only)
        with open(self.log_path, "a") as f:
            f.write(json.dumps(asdict(event), default=str) + "\n")

        return event

    def verify_chain(self) -> bool:
        """Verify the hash chain hasn't been tampered with."""
        events = self.get_events()
        expected_prev = "genesis"

        for event_dict in events:
            if event_dict["previous_hash"] != expected_prev:
                return False
            # Recompute hash
            event = AuditEvent(**{
                k: v for k, v in event_dict.items() if k != "event_hash"
            })
            event.previous_hash = expected_prev
            computed = event.compute_hash()
            if computed != event_dict["event_hash"]:
                return False
            expected_prev = event_dict["event_hash"]

        return True

    def get_events(
        self,
        model_name: Optional[str] = None,
        event_type: Optional[str] = None,
    ) -> list[dict]:
        """Query audit events with optional filters."""
        if not self.log_path.exists():
            return []

        events = []
        for line in self.log_path.read_text().strip().split("\n"):
            if not line:
                continue
            event = json.loads(line)
            if model_name and event["model_name"] != model_name:
                continue
            if event_type and event["event_type"] != event_type:
                continue
            events.append(event)

        return events


# ── Usage ────────────────────────────────────────────────
if __name__ == "__main__":
    trail = AuditTrail("audit/model_events.jsonl")

    trail.record("model_created", "loan_model", "3.2.1", "alice",
                  {"experiment_id": "exp-42", "algorithm": "gbm"})
    trail.record("approval_requested", "loan_model", "3.2.1", "alice",
                  {"risk_tier": "medium"})
    trail.record("approval_granted", "loan_model", "3.2.1", "bob",
                  {"role": "tech_lead", "comments": "LGTM"})
    trail.record("model_deployed", "loan_model", "3.2.1", "ci-pipeline",
                  {"environment": "production", "strategy": "canary"})

    print("=== Audit Trail ===")
    for event in trail.get_events():
        print(f"  [{event['event_type']}] by {event['actor']} — {event['details']}")

    print(f"\nChain integrity: {'✅ Valid' if trail.verify_chain() else '❌ Tampered'}")
```

---

## Model Risk Management

### Risk Assessment Framework

```
┌───────────────────────────────────────────────────────────────┐
│           Model Risk Assessment Matrix                        │
│                                                               │
│                        Impact                                 │
│              Low         Medium        High                   │
│         ┌───────────┬───────────┬───────────┐                 │
│  High   │  Medium   │   High    │ Critical  │                 │
│  Like-  ├───────────┼───────────┼───────────┤                 │
│  lihood │   Low     │  Medium   │   High    │                 │
│  of     ├───────────┼───────────┼───────────┤                 │
│  Failure│   Low     │   Low     │  Medium   │                 │
│         └───────────┴───────────┴───────────┘                 │
│                                                               │
│  Impact factors:                                              │
│  • Financial exposure ($)                                     │
│  • Number of affected users                                   │
│  • Regulatory sensitivity                                     │
│  • Reputational risk                                          │
│                                                               │
│  Likelihood factors:                                          │
│  • Model complexity                                           │
│  • Data quality / volatility                                  │
│  • Domain stability                                           │
│  • Monitoring coverage                                        │
└───────────────────────────────────────────────────────────────┘
```

### Risk Tier Calculator

```python
# governance/risk_assessment.py
"""Automated model risk tier calculation."""
from dataclasses import dataclass
from enum import Enum


class RiskTier(Enum):
    LOW = "low"
    MEDIUM = "medium"
    HIGH = "high"
    CRITICAL = "critical"


@dataclass
class RiskFactors:
    # Impact factors (1-5 scale)
    financial_exposure: int = 1      # 1=minimal, 5=millions
    users_affected: int = 1          # 1=internal, 5=millions of users
    regulatory_sensitivity: int = 1  # 1=none, 5=heavily regulated
    reputational_risk: int = 1       # 1=low, 5=front-page news

    # Likelihood factors (1-5 scale)
    model_complexity: int = 1        # 1=linear, 5=deep learning
    data_volatility: int = 1         # 1=stable, 5=rapidly changing
    domain_stability: int = 1        # 1=mature, 5=novel domain
    monitoring_coverage: int = 5     # 1=none, 5=comprehensive (inverse)


def calculate_risk_tier(factors: RiskFactors) -> dict:
    """Calculate risk tier from risk factors."""
    impact = (
        factors.financial_exposure
        + factors.users_affected
        + factors.regulatory_sensitivity
        + factors.reputational_risk
    ) / 4

    likelihood = (
        factors.model_complexity
        + factors.data_volatility
        + factors.domain_stability
        + (6 - factors.monitoring_coverage)  # Invert: less monitoring = higher risk
    ) / 4

    risk_score = impact * likelihood

    if risk_score >= 16:
        tier = RiskTier.CRITICAL
    elif risk_score >= 9:
        tier = RiskTier.HIGH
    elif risk_score >= 4:
        tier = RiskTier.MEDIUM
    else:
        tier = RiskTier.LOW

    return {
        "impact_score": round(impact, 2),
        "likelihood_score": round(likelihood, 2),
        "risk_score": round(risk_score, 2),
        "risk_tier": tier.value,
        "required_controls": TIER_CONTROLS[tier],
    }


TIER_CONTROLS = {
    RiskTier.LOW: [
        "Code review",
        "Automated tests",
        "Basic monitoring",
    ],
    RiskTier.MEDIUM: [
        "Code review",
        "Automated tests",
        "Model card",
        "Fairness audit",
        "Performance monitoring",
        "Quarterly re-validation",
    ],
    RiskTier.HIGH: [
        "Code review",
        "Automated tests",
        "Model card",
        "Fairness audit",
        "Independent validation",
        "Continuous monitoring",
        "Monthly re-validation",
        "Legal review",
        "Rollback plan",
    ],
    RiskTier.CRITICAL: [
        "Code review",
        "Automated tests",
        "Model card",
        "Fairness audit",
        "Independent validation",
        "External audit",
        "Continuous monitoring",
        "Monthly re-validation",
        "Legal review",
        "Regulatory filing",
        "Board reporting",
        "Tested rollback plan",
        "Incident response plan",
    ],
}


# ── Usage ────────────────────────────────────────────────
if __name__ == "__main__":
    # Credit scoring model — high risk
    credit_factors = RiskFactors(
        financial_exposure=4,
        users_affected=4,
        regulatory_sensitivity=5,
        reputational_risk=4,
        model_complexity=3,
        data_volatility=2,
        domain_stability=2,
        monitoring_coverage=4,
    )

    result = calculate_risk_tier(credit_factors)
    print(f"Risk tier: {result['risk_tier'].upper()}")
    print(f"Risk score: {result['risk_score']}")
    print(f"Required controls:")
    for ctrl in result["required_controls"]:
        print(f"  • {ctrl}")
```

---

## Model Inventory

### What to Track

```
┌───────────────────────────────────────────────────────────────┐
│                  Model Inventory Fields                        │
│                                                               │
│  Identity                                                     │
│  ├── Model ID (unique)                                        │
│  ├── Model name                                               │
│  ├── Version (current + history)                              │
│  ├── Risk tier                                                │
│  └── Status (development/staging/production/deprecated)       │
│                                                               │
│  Ownership                                                    │
│  ├── Owner (person)                                           │
│  ├── Team                                                     │
│  ├── Business sponsor                                         │
│  └── Escalation contact                                       │
│                                                               │
│  Technical                                                    │
│  ├── Algorithm / framework                                    │
│  ├── Serving endpoint                                         │
│  ├── Dependencies (data sources, upstream models)             │
│  ├── Resource requirements (CPU/GPU/memory)                   │
│  └── SLA (latency, availability)                              │
│                                                               │
│  Governance                                                   │
│  ├── Approval status + date                                   │
│  ├── Last validation date                                     │
│  ├── Next validation due                                      │
│  ├── Monitoring dashboard link                                │
│  └── Model card link                                          │
└───────────────────────────────────────────────────────────────┘
```

### Model Inventory System

```python
# governance/model_inventory.py
"""Central model inventory for tracking all deployed models."""
import json
from dataclasses import dataclass, field, asdict
from datetime import datetime, timedelta
from pathlib import Path
from typing import Optional


@dataclass
class ModelEntry:
    model_id: str
    model_name: str
    version: str
    risk_tier: str
    status: str  # "development", "staging", "production", "deprecated"
    owner: str
    team: str
    algorithm: str
    serving_endpoint: Optional[str] = None
    approved_date: Optional[str] = None
    deployed_date: Optional[str] = None
    last_validated: Optional[str] = None
    next_validation_due: Optional[str] = None
    model_card_url: Optional[str] = None
    dashboard_url: Optional[str] = None
    dependencies: list[str] = field(default_factory=list)
    tags: list[str] = field(default_factory=list)


class ModelInventory:
    """Central registry of all models in the organization."""

    def __init__(self, storage_path: str):
        self.storage_path = Path(storage_path)
        self.storage_path.parent.mkdir(parents=True, exist_ok=True)
        self._models: dict[str, ModelEntry] = {}
        self._load()

    def _load(self):
        if self.storage_path.exists():
            data = json.loads(self.storage_path.read_text())
            for model_id, entry in data.items():
                self._models[model_id] = ModelEntry(**entry)

    def _save(self):
        data = {mid: asdict(m) for mid, m in self._models.items()}
        self.storage_path.write_text(json.dumps(data, indent=2, default=str))

    def register(self, entry: ModelEntry) -> str:
        """Register a new model in the inventory."""
        self._models[entry.model_id] = entry
        self._save()
        return entry.model_id

    def update_status(self, model_id: str, status: str):
        """Update model status (e.g., staging → production)."""
        if model_id not in self._models:
            raise KeyError(f"Model not found: {model_id}")
        self._models[model_id].status = status
        if status == "production":
            self._models[model_id].deployed_date = datetime.utcnow().isoformat()
        self._save()

    def get_models_needing_validation(self) -> list[ModelEntry]:
        """Find models whose validation is due or overdue."""
        now = datetime.utcnow()
        overdue = []
        for model in self._models.values():
            if model.status != "production":
                continue
            if model.next_validation_due:
                due = datetime.fromisoformat(model.next_validation_due)
                if now >= due:
                    overdue.append(model)
        return overdue

    def get_production_models(self) -> list[ModelEntry]:
        """List all models currently in production."""
        return [m for m in self._models.values() if m.status == "production"]

    def get_models_by_risk(self, risk_tier: str) -> list[ModelEntry]:
        """List models by risk tier."""
        return [m for m in self._models.values() if m.risk_tier == risk_tier]

    def generate_report(self) -> dict:
        """Generate inventory summary report."""
        models = list(self._models.values())
        return {
            "total_models": len(models),
            "by_status": {
                s: len([m for m in models if m.status == s])
                for s in ["development", "staging", "production", "deprecated"]
            },
            "by_risk_tier": {
                t: len([m for m in models if m.risk_tier == t])
                for t in ["low", "medium", "high", "critical"]
            },
            "needing_validation": len(self.get_models_needing_validation()),
            "without_model_card": len([m for m in models if not m.model_card_url]),
        }


# ── Usage ────────────────────────────────────────────────
if __name__ == "__main__":
    inventory = ModelInventory("governance/inventory.json")

    inventory.register(ModelEntry(
        model_id="loan-model-001",
        model_name="loan_approval",
        version="3.2.1",
        risk_tier="high",
        status="production",
        owner="alice@example.com",
        team="credit-risk",
        algorithm="GradientBoosting",
        serving_endpoint="https://api.example.com/loan/predict",
        approved_date="2024-06-01",
        deployed_date="2024-06-05",
        last_validated="2024-06-01",
        next_validation_due="2024-07-01",
        model_card_url="https://wiki.example.com/models/loan_approval",
    ))

    report = inventory.generate_report()
    print("=== Model Inventory Report ===")
    for k, v in report.items():
        print(f"  {k}: {v}")
```

---

## Roles and Responsibilities

### RACI Matrix for Model Governance

```
┌────────────────────────────────────────────────────────────────┐
│              RACI Matrix: Model Governance                      │
│                                                                │
│  R = Responsible  A = Accountable  C = Consulted  I = Informed │
│                                                                │
│  Activity           │ DS │ MLE │ TL │ PO │ Gov │ Legal│ Exec  │
│  ───────────────────┼────┼─────┼────┼────┼─────┼──────┼─────  │
│  Model development  │ R  │  C  │ A  │ I  │  I  │      │       │
│  Feature selection   │ R  │  C  │ A  │ C  │     │  C   │       │
│  Bias testing        │ R  │  C  │ A  │ I  │  C  │  I   │       │
│  Model documentation│ R  │  C  │ A  │ I  │  C  │      │       │
│  Risk assessment    │ C  │  C  │ R  │ C  │  A  │  C   │  I    │
│  Approval decision  │ I  │  I  │ R  │ R  │  A  │  C   │  I    │
│  Deployment         │ C  │  R  │ A  │ I  │  I  │      │       │
│  Monitoring setup   │ C  │  R  │ A  │ I  │  C  │      │       │
│  Incident response  │ C  │  R  │ A  │ I  │  I  │  C   │  I    │
│  Re-validation      │ R  │  C  │ A  │ I  │  A  │      │       │
│  Decommissioning    │ C  │  R  │ A  │ R  │  A  │  I   │  I    │
│  Regulatory report  │ C  │  C  │ C  │ I  │  R  │  A   │  I    │
│                                                                │
│  DS = Data Scientist   MLE = ML Engineer   TL = Tech Lead      │
│  PO = Product Owner    Gov = Governance Board                  │
└────────────────────────────────────────────────────────────────┘
```

| Role | Key Responsibilities |
|---|---|
| **Data Scientist** | Build models, run bias tests, write model cards, perform re-validation |
| **ML Engineer** | Deploy models, set up monitoring, build CI/CD pipelines, manage infrastructure |
| **Tech Lead** | Technical review, architecture decisions, risk assessment input |
| **Product Owner** | Business justification, intended use definition, accept/reject based on business risk |
| **Governance Board** | Approve high/critical models, set policies, periodic inventory review |
| **Legal / Compliance** | Regulatory requirements, data privacy review, legal risk assessment |
| **Executive Sponsor** | Final sign-off for critical models, accountability for organizational AI ethics |

---

## Summary

| Concept | Key Takeaway |
|---|---|
| **Model documentation** | Every model needs a model card, business justification, monitoring plan, and rollback plan |
| **Approval workflows** | Risk-tiered: low risk needs peer review; critical risk needs governance board + legal + executive |
| **Audit trails** | Hash-chained, append-only log of every governance event for tamper detection and compliance |
| **Risk management** | Score models on impact × likelihood; tier determines required controls and review frequency |
| **Model inventory** | Central registry of all models with status, owner, risk tier, validation dates |
| **RACI matrix** | Clear roles: DS builds, MLE deploys, TL reviews, governance board approves, legal advises |

> **Rule of thumb:** Governance overhead should be proportional to risk. A low-risk internal dashboard model doesn't need a governance board review, but a credit-scoring model affecting millions of users does. Over-governing low-risk models creates bottlenecks; under-governing high-risk models creates liability.

---

## Revision Questions

1. **Design** a model governance framework for a fintech company with 50 ML models in production. How would you tier them, what approval workflows would each tier require, and how would you track compliance?

2. **Explain** why an append-only, hash-chained audit trail is important for model governance. What attacks or errors does it protect against, and what are its limitations?

3. **A data scientist** wants to deploy a new fraud detection model. Walk through the governance process from model development to production, including all documentation, approvals, and monitoring setup required for a high-risk model.

4. **Compare** the governance requirements for three models: (a) an internal dashboard model that estimates team capacity, (b) a customer-facing product recommendation model, and (c) a credit scoring model used for lending decisions. How do documentation, approval, and monitoring requirements differ?

5. **Your model inventory** shows that 15 of 50 production models have overdue re-validations. Design a process to triage and address this backlog. What criteria would you use to prioritize, and how would you prevent this from recurring?

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Explainability in Production](./03-explainability-in-production.md) | [📂 Responsible AI](./README.md) | [Regulatory Compliance →](./05-regulatory-compliance.md) |
