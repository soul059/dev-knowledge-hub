# 📜 Regulatory Compliance

> **Unit 11 · Responsible AI in Production · Chapter 5**
>
> How to navigate the regulatory landscape for production ML systems — GDPR right to explanation, EU AI Act risk categories, FDA guidelines for ML in healthcare, SOC 2 for ML systems, and a practical compliance checklist for ML teams.

---

## Table of Contents

1. [Chapter Overview](#chapter-overview)
2. [Regulatory Landscape Overview](#regulatory-landscape-overview)
3. [GDPR and the Right to Explanation](#gdpr-and-the-right-to-explanation)
4. [EU AI Act](#eu-ai-act)
5. [FDA Guidelines for ML in Healthcare](#fda-guidelines-for-ml-in-healthcare)
6. [SOC 2 for ML Systems](#soc-2-for-ml-systems)
7. [Compliance Checklist for ML Teams](#compliance-checklist-for-ml-teams)
8. [Building a Compliance Pipeline](#building-a-compliance-pipeline)
9. [Summary](#summary)
10. [Revision Questions](#revision-questions)
11. [Navigation](#navigation)

---

## Chapter Overview

ML systems don't operate in a regulatory vacuum. A model that scores consumers must comply with GDPR, fair lending laws, and potentially the EU AI Act. A clinical decision-support model needs FDA clearance. An ML system handling customer data needs SOC 2 controls. Yet most ML teams discover compliance requirements late — after the model is built, sometimes after it's deployed. This chapter provides a practical guide to the **key regulations** affecting production ML, what they require of your models and infrastructure, and a **compliance checklist** you can embed into your development process to avoid last-minute surprises.

```
Regulatory Coverage for ML Systems
┌───────────────────────────────────────────────────────────────┐
│                                                               │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐              │
│  │   GDPR     │  │  EU AI Act │  │    FDA     │              │
│  │            │  │            │  │            │              │
│  │ Data       │  │ Risk-based │  │ Medical    │              │
│  │ protection │  │ AI rules   │  │ device     │              │
│  │ + right to │  │ for EU     │  │ regulation │              │
│  │ explanation│  │ market     │  │ for ML     │              │
│  └────────────┘  └────────────┘  └────────────┘              │
│                                                               │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐              │
│  │   SOC 2   │  │ ECOA /     │  │  NYC LL    │              │
│  │            │  │ Reg B      │  │  144       │              │
│  │ Security   │  │            │  │            │              │
│  │ controls   │  │ Fair       │  │ Automated  │              │
│  │ for SaaS   │  │ lending    │  │ employment │              │
│  │ + ML       │  │ (US)       │  │ decisions  │              │
│  └────────────┘  └────────────┘  └────────────┘              │
│                                                               │
│  Most ML systems are affected by 2+ regulations              │
└───────────────────────────────────────────────────────────────┘
```

---

## Regulatory Landscape Overview

### Key Regulations by Domain

| Regulation | Jurisdiction | Scope | ML-Specific? | Status |
|---|---|---|---|---|
| **GDPR** | EU / EEA | All personal data processing | Partially (Art. 22) | In force (2018) |
| **EU AI Act** | EU / EEA | AI systems by risk level | ✅ Yes | In force (2024) |
| **ECOA / Reg B** | US | Credit decisions | No (applies to ML) | In force (1974) |
| **FCRA** | US | Consumer reports / credit | No (applies to ML) | In force (1970) |
| **NYC LL 144** | NYC | Automated employment decisions | ✅ Yes | In force (2023) |
| **FDA SaMD** | US | Software as Medical Device | Partially | Evolving |
| **SOC 2** | Global | Service org security controls | No (applies to ML) | Industry standard |
| **SR 11-7** | US Banking | Model risk management | Partially | In force (2011) |
| **HIPAA** | US | Protected health information | No (applies to ML) | In force (1996) |

### Regulation Applicability Matrix

```
┌───────────────────────────────────────────────────────────────┐
│       Which Regulations Apply to Your ML System?              │
│                                                               │
│  Question                                 → Regulation        │
│  ─────────────────────────────────────────────────────────    │
│  Process personal data of EU residents?   → GDPR              │
│  Make automated decisions about people?   → GDPR Art. 22      │
│  Deployed in the EU market?               → EU AI Act          │
│  Make credit decisions?                   → ECOA, FCRA, SR11-7│
│  Make hiring decisions in NYC?            → NYC LL 144         │
│  Medical diagnosis or treatment?          → FDA SaMD, HIPAA    │
│  Handle customer data as SaaS?            → SOC 2              │
│  Used by a US bank?                       → SR 11-7, OCC       │
│  Insurance pricing or underwriting?       → State insurance law│
│  Criminal justice risk assessment?        → Varies by state    │
└───────────────────────────────────────────────────────────────┘
```

---

## GDPR and the Right to Explanation

### Key GDPR Articles for ML

| Article | Requirement | ML Impact |
|---|---|---|
| **Art. 5** | Data minimization, purpose limitation | Only use necessary features; don't repurpose training data |
| **Art. 13/14** | Transparency about data use | Disclose that ML is used; describe logic involved |
| **Art. 15** | Right of access | Users can request what data you hold and how it's used |
| **Art. 17** | Right to erasure ("right to be forgotten") | Must delete user data from training sets on request |
| **Art. 22** | Automated decision-making | Right to human review; right to explanation of logic |
| **Art. 25** | Data protection by design | Privacy built into ML pipeline, not bolted on |
| **Art. 35** | Data Protection Impact Assessment (DPIA) | Required for high-risk automated processing |

### Art. 22 — Automated Individual Decision-Making

```
┌───────────────────────────────────────────────────────────────┐
│            GDPR Art. 22 Decision Flow                         │
│                                                               │
│  Is the decision solely automated?                            │
│       │                                                       │
│       ├── No → Art. 22 does NOT apply                         │
│       │        (human in the loop = exemption)                │
│       │                                                       │
│       └── Yes → Does it produce legal or significant effects? │
│                   │                                           │
│                   ├── No → Art. 22 does NOT apply             │
│                   │                                           │
│                   └── Yes → Art. 22 APPLIES:                  │
│                              │                                │
│                              ├── Right to human review        │
│                              ├── Right to contest decision    │
│                              ├── Right to explanation         │
│                              └── Must not be based on         │
│                                  special-category data        │
│                                  (race, health, etc.)         │
│                                  unless explicit consent      │
└───────────────────────────────────────────────────────────────┘
```

### GDPR Compliance Implementation

```python
# compliance/gdpr.py
"""GDPR compliance utilities for ML systems."""
import json
import hashlib
from datetime import datetime
from dataclasses import dataclass, field, asdict
from pathlib import Path
from typing import Optional


@dataclass
class DataSubjectRecord:
    """Track what data we hold and how it's used for a data subject."""
    subject_id: str
    data_collected: list[str] = field(default_factory=list)
    purposes: list[str] = field(default_factory=list)
    consent_date: Optional[str] = None
    used_in_training: bool = False
    training_dataset_versions: list[str] = field(default_factory=list)
    predictions_made: int = 0
    erasure_requested: bool = False
    erasure_completed: bool = False
    erasure_date: Optional[str] = None


@dataclass
class AutomatedDecisionRecord:
    """Record for GDPR Art. 22 compliance — automated decisions."""
    decision_id: str
    subject_id: str
    timestamp: str
    model_name: str
    model_version: str
    decision: str
    has_legal_effect: bool
    explanation: dict
    human_review_available: bool = True
    human_review_requested: bool = False
    human_review_outcome: Optional[str] = None


class GDPRComplianceManager:
    """Manage GDPR compliance for ML systems."""

    def __init__(self, storage_dir: str):
        self.storage_dir = Path(storage_dir)
        self.storage_dir.mkdir(parents=True, exist_ok=True)

    def record_automated_decision(
        self,
        subject_id: str,
        model_name: str,
        model_version: str,
        decision: str,
        explanation: dict,
        has_legal_effect: bool = True,
    ) -> AutomatedDecisionRecord:
        """Record an automated decision for Art. 22 compliance."""
        record = AutomatedDecisionRecord(
            decision_id=hashlib.sha256(
                f"{subject_id}:{datetime.utcnow().isoformat()}".encode()
            ).hexdigest()[:16],
            subject_id=subject_id,
            timestamp=datetime.utcnow().isoformat(),
            model_name=model_name,
            model_version=model_version,
            decision=decision,
            has_legal_effect=has_legal_effect,
            explanation=explanation,
        )

        # Persist record
        path = self.storage_dir / f"decisions/{subject_id}"
        path.mkdir(parents=True, exist_ok=True)
        (path / f"{record.decision_id}.json").write_text(
            json.dumps(asdict(record), indent=2)
        )

        return record

    def handle_explanation_request(self, subject_id: str) -> list[dict]:
        """Handle a data subject's request for explanation (Art. 22)."""
        path = self.storage_dir / f"decisions/{subject_id}"
        if not path.exists():
            return []

        decisions = []
        for file in path.glob("*.json"):
            record = json.loads(file.read_text())
            decisions.append({
                "decision_id": record["decision_id"],
                "date": record["timestamp"],
                "model": record["model_name"],
                "decision": record["decision"],
                "explanation": record["explanation"],
                "human_review_available": record["human_review_available"],
            })

        return decisions

    def handle_erasure_request(self, subject_id: str) -> dict:
        """Handle right to erasure request (Art. 17)."""
        actions = []

        # Remove from prediction logs
        decision_path = self.storage_dir / f"decisions/{subject_id}"
        if decision_path.exists():
            import shutil
            shutil.rmtree(decision_path)
            actions.append("Deleted decision records")

        # Flag for removal from training data
        erasure_log = self.storage_dir / "erasure_requests.jsonl"
        with open(erasure_log, "a") as f:
            f.write(json.dumps({
                "subject_id": subject_id,
                "requested_at": datetime.utcnow().isoformat(),
                "status": "pending_training_data_removal",
            }) + "\n")
        actions.append("Flagged for training data removal")

        return {
            "subject_id": subject_id,
            "actions_taken": actions,
            "note": "Model retraining required to fully remove data influence",
        }


# ── Usage ────────────────────────────────────────────────
if __name__ == "__main__":
    gdpr = GDPRComplianceManager("compliance/gdpr_records")

    # Record an automated decision
    record = gdpr.record_automated_decision(
        subject_id="user-12345",
        model_name="loan_approval",
        model_version="3.2.1",
        decision="DENIED",
        explanation={
            "primary_factors": [
                {"feature": "debt_to_income", "impact": "negative", "contribution": -0.25},
                {"feature": "credit_history_length", "impact": "negative", "contribution": -0.15},
            ],
            "human_readable": "Application declined primarily due to high debt-to-income ratio and short credit history.",
        },
    )
    print(f"Decision recorded: {record.decision_id}")

    # Handle explanation request
    explanations = gdpr.handle_explanation_request("user-12345")
    print(f"\nExplanations for user-12345: {len(explanations)} decision(s)")
```

---

## EU AI Act

### Risk Categories

```
┌───────────────────────────────────────────────────────────────┐
│               EU AI Act Risk Pyramid                          │
│                                                               │
│                    ╱╲                                          │
│                   ╱  ╲          PROHIBITED                     │
│                  ╱    ╲         • Social scoring               │
│                 ╱ BANNED╲       • Real-time biometric ID       │
│                ╱──────────╲     • Manipulation / exploitation  │
│               ╱            ╲                                   │
│              ╱   HIGH RISK  ╲   HEAVY REGULATION               │
│             ╱                ╲  • Credit scoring               │
│            ╱  Must register,  ╲ • Hiring / recruitment         │
│           ╱   monitor, audit   ╲• Law enforcement              │
│          ╱──────────────────────╲• Medical devices              │
│         ╱                        ╲                             │
│        ╱     LIMITED RISK         ╲ TRANSPARENCY OBLIGATIONS   │
│       ╱                            ╲• Chatbots                 │
│      ╱   Must disclose AI use       ╲• Emotion recognition     │
│     ╱────────────────────────────────╲• Deepfakes              │
│    ╱                                  ╲                        │
│   ╱         MINIMAL RISK              ╲ NO OBLIGATIONS         │
│  ╱   Spam filters, game AI, etc.       ╲                       │
│ ╱──────────────────────────────────────╲                      │
└───────────────────────────────────────────────────────────────┘
```

### High-Risk AI Requirements

| Requirement | Description | Implementation |
|---|---|---|
| **Risk management** | Continuous risk identification and mitigation | Risk assessment framework (Chapter 4) |
| **Data governance** | Training data quality, relevance, completeness | Data validation pipelines |
| **Technical documentation** | Detailed system description | Model cards + architecture docs |
| **Record-keeping** | Automatic logging of system events | Audit trail (Chapter 4) |
| **Transparency** | Clear instructions for deployers | User-facing documentation |
| **Human oversight** | Human can understand, monitor, and override | Kill switch + human-in-the-loop |
| **Accuracy & robustness** | Appropriate accuracy levels, cybersecurity | Testing + adversarial robustness |
| **Quality management** | QMS covering entire lifecycle | Governance workflows |

### EU AI Act Compliance Checker

```python
# compliance/eu_ai_act.py
"""EU AI Act compliance assessment for ML systems."""
from dataclasses import dataclass, field
from enum import Enum
from typing import Optional


class AIActRiskLevel(Enum):
    PROHIBITED = "prohibited"
    HIGH_RISK = "high_risk"
    LIMITED_RISK = "limited_risk"
    MINIMAL_RISK = "minimal_risk"


# Annex III high-risk categories
HIGH_RISK_CATEGORIES = [
    "biometric_identification",
    "critical_infrastructure",
    "education_training",
    "employment_hr",
    "essential_services",    # credit, insurance, social benefits
    "law_enforcement",
    "migration_border",
    "justice_democratic",
]

PROHIBITED_USES = [
    "social_scoring",
    "realtime_biometric_public",
    "subliminal_manipulation",
    "exploitation_vulnerability",
]


@dataclass
class AIActAssessment:
    system_name: str
    description: str
    use_category: str
    risk_level: AIActRiskLevel = AIActRiskLevel.MINIMAL_RISK
    requirements: list[str] = field(default_factory=list)
    gaps: list[str] = field(default_factory=list)
    compliant: bool = False


def assess_risk_level(use_category: str) -> AIActRiskLevel:
    """Determine EU AI Act risk level from use category."""
    if use_category in PROHIBITED_USES:
        return AIActRiskLevel.PROHIBITED
    elif use_category in HIGH_RISK_CATEGORIES:
        return AIActRiskLevel.HIGH_RISK
    elif use_category in ["chatbot", "emotion_recognition", "deepfake_generation"]:
        return AIActRiskLevel.LIMITED_RISK
    else:
        return AIActRiskLevel.MINIMAL_RISK


HIGH_RISK_REQUIREMENTS = {
    "risk_management_system": "Implement continuous risk identification and mitigation",
    "data_governance": "Ensure training data quality, relevance, and representativeness",
    "technical_documentation": "Comprehensive system description and model cards",
    "record_keeping": "Automatic logging of all system events and decisions",
    "transparency": "Clear instructions for downstream deployers",
    "human_oversight": "Allow effective human oversight and override capability",
    "accuracy_robustness": "Appropriate accuracy, cybersecurity measures",
    "quality_management": "Quality management system covering full lifecycle",
    "conformity_assessment": "Self-assessment or third-party assessment",
    "eu_database_registration": "Register in EU public database of high-risk AI",
    "post_market_monitoring": "Continuous monitoring after deployment",
}


def assess_compliance(
    system_name: str,
    description: str,
    use_category: str,
    existing_controls: list[str],
) -> AIActAssessment:
    """Assess EU AI Act compliance for an ML system."""
    risk_level = assess_risk_level(use_category)

    assessment = AIActAssessment(
        system_name=system_name,
        description=description,
        use_category=use_category,
        risk_level=risk_level,
    )

    if risk_level == AIActRiskLevel.PROHIBITED:
        assessment.requirements = ["SYSTEM PROHIBITED — cannot be deployed in EU"]
        assessment.gaps = ["PROHIBITED"]
        assessment.compliant = False
        return assessment

    if risk_level == AIActRiskLevel.HIGH_RISK:
        assessment.requirements = list(HIGH_RISK_REQUIREMENTS.keys())
        assessment.gaps = [
            req for req in HIGH_RISK_REQUIREMENTS
            if req not in existing_controls
        ]
    elif risk_level == AIActRiskLevel.LIMITED_RISK:
        assessment.requirements = ["transparency_disclosure"]
        assessment.gaps = [
            r for r in assessment.requirements if r not in existing_controls
        ]
    else:
        assessment.requirements = []
        assessment.gaps = []

    assessment.compliant = len(assessment.gaps) == 0
    return assessment


# ── Usage ────────────────────────────────────────────────
if __name__ == "__main__":
    existing = [
        "risk_management_system",
        "data_governance",
        "technical_documentation",
        "record_keeping",
        "accuracy_robustness",
    ]

    result = assess_compliance(
        system_name="CreditScore AI",
        description="Automated credit scoring for consumer loans",
        use_category="essential_services",
        existing_controls=existing,
    )

    print(f"System: {result.system_name}")
    print(f"Risk level: {result.risk_level.value}")
    print(f"Compliant: {'✅' if result.compliant else '❌'}")
    if result.gaps:
        print(f"\nGaps to address:")
        for gap in result.gaps:
            print(f"  ❌ {gap}: {HIGH_RISK_REQUIREMENTS.get(gap, '')}")
```

---

## FDA Guidelines for ML in Healthcare

### FDA Framework for AI/ML-Based SaMD

```
┌───────────────────────────────────────────────────────────────┐
│          FDA AI/ML Framework for SaMD                         │
│                                                               │
│  SaMD = Software as a Medical Device                          │
│                                                               │
│  Classification by risk:                                      │
│                                                               │
│  ┌───────────────────────────────────────────────────────┐    │
│  │ Class I  — Low risk                                    │    │
│  │  General controls only                                 │    │
│  │  e.g., wellness apps, fitness trackers                 │    │
│  ├───────────────────────────────────────────────────────┤    │
│  │ Class II — Moderate risk                               │    │
│  │  510(k) clearance (substantially equivalent)           │    │
│  │  e.g., clinical decision support, imaging CAD          │    │
│  ├───────────────────────────────────────────────────────┤    │
│  │ Class III — High risk                                  │    │
│  │  Pre-market approval (PMA)                             │    │
│  │  e.g., autonomous diagnosis, life-critical decisions   │    │
│  └───────────────────────────────────────────────────────┘    │
│                                                               │
│  Key FDA concept: "Predetermined Change Control Plan"         │
│  • Pre-specify what model changes are allowed                 │
│  • Define re-validation requirements for each change type     │
│  • Enables continuous learning without full re-submission     │
└───────────────────────────────────────────────────────────────┘
```

### FDA Compliance Requirements for ML

| Requirement | Description | Implementation |
|---|---|---|
| **Intended use** | Clear statement of medical purpose | Documented in model card |
| **Clinical validation** | Evidence of clinical effectiveness | Prospective or retrospective study |
| **Analytical validation** | Technical performance (sensitivity, specificity) | Test on held-out clinical data |
| **Algorithm description** | How the model works, what data it uses | Technical documentation |
| **Training data** | Description of training population | Demographics, inclusion/exclusion criteria |
| **Performance by subgroup** | Fairness across demographics | Per-subgroup performance metrics |
| **Change control plan** | How model updates will be managed | Predetermined Change Control Plan |
| **Real-world monitoring** | Post-market surveillance | Continuous performance monitoring |
| **Labeling** | Warnings, limitations, instructions for use | User-facing documentation |

### Healthcare ML Compliance Tracker

```python
# compliance/fda_tracker.py
"""Track FDA compliance requirements for ML-based medical devices."""
from dataclasses import dataclass, field
from typing import Optional


@dataclass
class FDAComplianceStatus:
    """Track FDA compliance for a healthcare ML system."""
    device_name: str
    device_class: str  # "I", "II", "III"
    intended_use: str
    submission_type: str  # "exempt", "510k", "PMA"

    # Documentation requirements
    algorithm_description: bool = False
    intended_use_documented: bool = False
    training_data_described: bool = False
    clinical_validation: bool = False
    analytical_validation: bool = False
    subgroup_performance: bool = False
    change_control_plan: bool = False
    labeling_complete: bool = False
    post_market_plan: bool = False

    # Validation results
    sensitivity: Optional[float] = None
    specificity: Optional[float] = None
    auc: Optional[float] = None
    subgroup_results: dict = field(default_factory=dict)

    def compliance_score(self) -> float:
        """Calculate percentage of requirements met."""
        reqs = [
            self.algorithm_description,
            self.intended_use_documented,
            self.training_data_described,
            self.clinical_validation,
            self.analytical_validation,
            self.subgroup_performance,
            self.change_control_plan,
            self.labeling_complete,
            self.post_market_plan,
        ]
        return sum(reqs) / len(reqs) * 100

    def gaps(self) -> list[str]:
        """List unmet requirements."""
        checks = {
            "Algorithm description": self.algorithm_description,
            "Intended use documented": self.intended_use_documented,
            "Training data described": self.training_data_described,
            "Clinical validation": self.clinical_validation,
            "Analytical validation": self.analytical_validation,
            "Subgroup performance": self.subgroup_performance,
            "Change control plan": self.change_control_plan,
            "Labeling complete": self.labeling_complete,
            "Post-market monitoring plan": self.post_market_plan,
        }
        return [name for name, met in checks.items() if not met]


# ── Usage ────────────────────────────────────────────────
if __name__ == "__main__":
    device = FDAComplianceStatus(
        device_name="ChestXR-AI",
        device_class="II",
        intended_use="Computer-aided detection of pneumothorax on chest X-rays",
        submission_type="510k",
        algorithm_description=True,
        intended_use_documented=True,
        training_data_described=True,
        clinical_validation=False,
        analytical_validation=True,
        subgroup_performance=True,
        change_control_plan=False,
        labeling_complete=False,
        post_market_plan=True,
        sensitivity=0.94,
        specificity=0.92,
        auc=0.97,
    )

    print(f"Device: {device.device_name}")
    print(f"Class: {device.device_class} ({device.submission_type})")
    print(f"Compliance: {device.compliance_score():.0f}%")
    if device.gaps():
        print(f"\nGaps:")
        for gap in device.gaps():
            print(f"  ❌ {gap}")
```

---

## SOC 2 for ML Systems

### SOC 2 Trust Service Criteria + ML

```
┌───────────────────────────────────────────────────────────────┐
│          SOC 2 Trust Service Criteria for ML Systems           │
│                                                               │
│  Criteria         │ Standard Requirement  │ ML-Specific        │
│  ─────────────────┼───────────────────────┼──────────────────  │
│  Security         │ Access controls,      │ Model access       │
│                   │ encryption, logging   │ controls, training │
│                   │                       │ data encryption    │
│  ─────────────────┼───────────────────────┼──────────────────  │
│  Availability     │ SLAs, monitoring,     │ Model serving      │
│                   │ disaster recovery     │ uptime, fallback   │
│                   │                       │ models             │
│  ─────────────────┼───────────────────────┼──────────────────  │
│  Processing       │ Complete, accurate,   │ Model accuracy,    │
│  Integrity        │ authorized processing │ input validation,  │
│                   │                       │ prediction logging │
│  ─────────────────┼───────────────────────┼──────────────────  │
│  Confidentiality  │ Protect confidential  │ Training data      │
│                   │ information           │ isolation, PII     │
│                   │                       │ in features        │
│  ─────────────────┼───────────────────────┼──────────────────  │
│  Privacy          │ Personal data per     │ Data minimization  │
│                   │ privacy notice        │ in features,       │
│                   │                       │ model inversion    │
│                   │                       │ protection         │
└───────────────────────────────────────────────────────────────┘
```

### SOC 2 ML Controls

```python
# compliance/soc2_controls.py
"""SOC 2 control verification for ML systems."""
from dataclasses import dataclass, field


@dataclass
class SOC2MLControl:
    control_id: str
    category: str  # security, availability, processing_integrity, confidentiality, privacy
    description: str
    implemented: bool = False
    evidence: str = ""
    last_tested: str = ""


SOC2_ML_CONTROLS = [
    # ── Security ─────────────────────────────────────────
    SOC2MLControl("SEC-ML-01", "security",
                  "Model artifacts stored in access-controlled registry"),
    SOC2MLControl("SEC-ML-02", "security",
                  "Training data encrypted at rest and in transit"),
    SOC2MLControl("SEC-ML-03", "security",
                  "Model serving endpoints require authentication"),
    SOC2MLControl("SEC-ML-04", "security",
                  "Audit logging for all model access and predictions"),
    SOC2MLControl("SEC-ML-05", "security",
                  "Model input validation prevents injection attacks"),

    # ── Availability ─────────────────────────────────────
    SOC2MLControl("AVL-ML-01", "availability",
                  "Model serving has defined SLA (uptime, latency)"),
    SOC2MLControl("AVL-ML-02", "availability",
                  "Fallback model or rule-based system exists"),
    SOC2MLControl("AVL-ML-03", "availability",
                  "Model rollback can be executed within defined RTO"),
    SOC2MLControl("AVL-ML-04", "availability",
                  "Model serving auto-scales based on load"),

    # ── Processing Integrity ─────────────────────────────
    SOC2MLControl("INT-ML-01", "processing_integrity",
                  "Model accuracy monitored against defined thresholds"),
    SOC2MLControl("INT-ML-02", "processing_integrity",
                  "Input data validated before model inference"),
    SOC2MLControl("INT-ML-03", "processing_integrity",
                  "All predictions logged with inputs and outputs"),
    SOC2MLControl("INT-ML-04", "processing_integrity",
                  "Model version tracked for every prediction"),

    # ── Confidentiality ──────────────────────────────────
    SOC2MLControl("CON-ML-01", "confidentiality",
                  "Training data classified and access-restricted"),
    SOC2MLControl("CON-ML-02", "confidentiality",
                  "PII removed or encrypted in feature pipelines"),
    SOC2MLControl("CON-ML-03", "confidentiality",
                  "Model not susceptible to data extraction attacks"),

    # ── Privacy ──────────────────────────────────────────
    SOC2MLControl("PRI-ML-01", "privacy",
                  "Data minimization: only necessary features used"),
    SOC2MLControl("PRI-ML-02", "privacy",
                  "Data retention policy for training and prediction data"),
    SOC2MLControl("PRI-ML-03", "privacy",
                  "Data subject requests (access, deletion) supported"),
]


def generate_soc2_report(controls: list[SOC2MLControl]) -> dict:
    """Generate SOC 2 compliance report for ML controls."""
    total = len(controls)
    implemented = sum(1 for c in controls if c.implemented)

    by_category = {}
    for c in controls:
        if c.category not in by_category:
            by_category[c.category] = {"total": 0, "implemented": 0, "gaps": []}
        by_category[c.category]["total"] += 1
        if c.implemented:
            by_category[c.category]["implemented"] += 1
        else:
            by_category[c.category]["gaps"].append(c.control_id)

    return {
        "total_controls": total,
        "implemented": implemented,
        "compliance_pct": round(implemented / total * 100, 1),
        "by_category": by_category,
    }


# ── Usage ────────────────────────────────────────────────
if __name__ == "__main__":
    # Simulate some controls being implemented
    for ctrl in SOC2_ML_CONTROLS[:12]:
        ctrl.implemented = True
        ctrl.evidence = "Verified in Q2 audit"
        ctrl.last_tested = "2024-06-15"

    report = generate_soc2_report(SOC2_ML_CONTROLS)
    print(f"SOC 2 ML Compliance: {report['compliance_pct']}%")
    print(f"Implemented: {report['implemented']}/{report['total_controls']}")
    print(f"\nBy category:")
    for cat, data in report["by_category"].items():
        status = "✅" if not data["gaps"] else "❌"
        print(f"  {status} {cat}: {data['implemented']}/{data['total']}")
        for gap in data["gaps"]:
            print(f"      Gap: {gap}")
```

---

## Compliance Checklist for ML Teams

### Pre-Development Checklist

```
┌───────────────────────────────────────────────────────────────┐
│         ML Compliance Checklist — Pre-Development             │
│                                                               │
│  ☐ Identify applicable regulations                            │
│  ☐ Classify AI risk level (EU AI Act)                         │
│  ☐ Conduct Data Protection Impact Assessment (if GDPR)        │
│  ☐ Define intended use and out-of-scope uses                  │
│  ☐ Review data sources for privacy / consent                  │
│  ☐ Assess model risk tier                                     │
│  ☐ Identify sensitive attributes for fairness testing         │
│  ☐ Document data minimization strategy                        │
│  ☐ Plan for human oversight / override                        │
│  ☐ Engage legal / compliance team early                       │
└───────────────────────────────────────────────────────────────┘
```

### Pre-Deployment Checklist

```
┌───────────────────────────────────────────────────────────────┐
│         ML Compliance Checklist — Pre-Deployment              │
│                                                               │
│  ☐ Model card complete                                        │
│  ☐ Fairness audit passed (all metrics within thresholds)      │
│  ☐ Explainability solution implemented                        │
│  ☐ Approval workflow completed (per risk tier)                │
│  ☐ Audit trail active                                         │
│  ☐ Model registered in inventory                              │
│  ☐ Monitoring dashboards configured                           │
│  ☐ Alert rules set for performance + fairness                 │
│  ☐ Rollback plan documented and tested                        │
│  ☐ Data subject request handling implemented (if GDPR)        │
│  ☐ Adverse action notice process (if credit/hiring)           │
│  ☐ SOC 2 controls verified                                    │
│  ☐ Security review (input validation, access controls)        │
│  ☐ Re-validation schedule set                                 │
└───────────────────────────────────────────────────────────────┘
```

### Post-Deployment Checklist

```
┌───────────────────────────────────────────────────────────────┐
│         ML Compliance Checklist — Post-Deployment             │
│                                                               │
│  ☐ Performance monitoring active and alerting                 │
│  ☐ Fairness monitoring active and alerting                    │
│  ☐ Prediction logging with explanations                       │
│  ☐ Data subject requests being fulfilled (GDPR)              │
│  ☐ Periodic re-validation on schedule                         │
│  ☐ Compliance reports generated on cadence                    │
│  ☐ Incident response plan tested                              │
│  ☐ Model inventory kept up to date                            │
│  ☐ Regulatory changes tracked and assessed                    │
└───────────────────────────────────────────────────────────────┘
```

---

## Building a Compliance Pipeline

### Architecture

```
┌───────────────────────────────────────────────────────────────┐
│            Compliance Pipeline Architecture                    │
│                                                               │
│  Code Commit                                                  │
│       │                                                       │
│       ▼                                                       │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐           │
│  │ Unit Tests  │──►│ Bias Tests  │──►│ Security    │           │
│  │             │  │ (Fairlearn) │  │ Scan        │           │
│  └─────────────┘  └──────┬──────┘  └──────┬──────┘           │
│                          │                │                   │
│                          ▼                ▼                   │
│                   ┌──────────────────────────────┐            │
│                   │  Compliance Gate              │            │
│                   │  • Model card exists?         │            │
│                   │  • Fairness thresholds met?   │            │
│                   │  • SOC 2 controls verified?   │            │
│                   │  • DPIA completed (GDPR)?     │            │
│                   │  • Approval workflow done?     │            │
│                   └──────────────┬───────────────┘            │
│                                  │                            │
│                          ┌───────┴───────┐                    │
│                          │               │                    │
│                     ❌ Block        ✅ Deploy                  │
│                     + Report        + Log to                  │
│                     gaps            audit trail               │
│                                         │                     │
│                                         ▼                     │
│                               ┌──────────────────┐            │
│                               │ Post-Deploy      │            │
│                               │ • Monitoring     │            │
│                               │ • Compliance     │            │
│                               │   reports        │            │
│                               │ • Re-validation  │            │
│                               │   scheduler      │            │
│                               └──────────────────┘            │
└───────────────────────────────────────────────────────────────┘
```

### Compliance Gate Implementation

```python
# compliance/compliance_gate.py
"""Automated compliance gate for ML deployment pipeline."""
import json
import sys
from dataclasses import dataclass, field
from pathlib import Path
from typing import Optional


@dataclass
class ComplianceCheck:
    name: str
    required: bool
    passed: bool
    details: str = ""


@dataclass
class ComplianceGateResult:
    model_name: str
    model_version: str
    regulations: list[str]
    checks: list[ComplianceCheck] = field(default_factory=list)
    overall_passed: bool = False


def run_compliance_gate(
    model_name: str,
    model_version: str,
    applicable_regulations: list[str],
    artifacts_dir: str,
) -> ComplianceGateResult:
    """Run all compliance checks for a model before deployment."""
    result = ComplianceGateResult(
        model_name=model_name,
        model_version=model_version,
        regulations=applicable_regulations,
    )

    artifacts = Path(artifacts_dir)

    # ── Check 1: Model card exists ───────────────────────
    model_card_exists = (artifacts / "model_card.json").exists()
    result.checks.append(ComplianceCheck(
        name="model_card",
        required=True,
        passed=model_card_exists,
        details="Model card found" if model_card_exists else "Model card MISSING",
    ))

    # ── Check 2: Fairness audit passed ───────────────────
    fairness_path = artifacts / "fairness_report.json"
    fairness_passed = False
    if fairness_path.exists():
        report = json.loads(fairness_path.read_text())
        fairness_passed = report.get("overall_compliance") == "COMPLIANT"
    result.checks.append(ComplianceCheck(
        name="fairness_audit",
        required=True,
        passed=fairness_passed,
        details="Fairness audit passed" if fairness_passed else "Fairness audit FAILED or MISSING",
    ))

    # ── Check 3: Approval workflow completed ─────────────
    approval_path = artifacts / "approval.json"
    approval_passed = False
    if approval_path.exists():
        approval = json.loads(approval_path.read_text())
        approval_passed = approval.get("status") == "approved"
    result.checks.append(ComplianceCheck(
        name="approval_workflow",
        required=True,
        passed=approval_passed,
        details="Approval granted" if approval_passed else "Approval PENDING or MISSING",
    ))

    # ── Check 4: GDPR DPIA (if applicable) ───────────────
    if "gdpr" in applicable_regulations:
        dpia_exists = (artifacts / "dpia.json").exists()
        result.checks.append(ComplianceCheck(
            name="gdpr_dpia",
            required=True,
            passed=dpia_exists,
            details="DPIA completed" if dpia_exists else "DPIA REQUIRED but MISSING",
        ))

    # ── Check 5: EU AI Act registration (if applicable) ──
    if "eu_ai_act" in applicable_regulations:
        registration_exists = (artifacts / "ai_act_registration.json").exists()
        result.checks.append(ComplianceCheck(
            name="eu_ai_act_registration",
            required=True,
            passed=registration_exists,
            details="AI Act registration found" if registration_exists else "Registration MISSING",
        ))

    # ── Check 6: Monitoring plan exists ──────────────────
    monitoring_exists = (artifacts / "monitoring_plan.json").exists()
    result.checks.append(ComplianceCheck(
        name="monitoring_plan",
        required=True,
        passed=monitoring_exists,
        details="Monitoring plan found" if monitoring_exists else "Monitoring plan MISSING",
    ))

    # ── Overall result ───────────────────────────────────
    required_checks = [c for c in result.checks if c.required]
    result.overall_passed = all(c.passed for c in required_checks)

    return result


# ── Usage ────────────────────────────────────────────────
if __name__ == "__main__":
    result = run_compliance_gate(
        model_name="loan_model",
        model_version="3.2.1",
        applicable_regulations=["gdpr", "eu_ai_act"],
        artifacts_dir="models/loan_model/3.2.1/",
    )

    print(f"=== Compliance Gate: {result.model_name} v{result.model_version} ===")
    print(f"Regulations: {', '.join(result.regulations)}")
    print()

    for check in result.checks:
        status = "✅" if check.passed else "❌"
        req = "(required)" if check.required else "(optional)"
        print(f"  {status} {check.name} {req}: {check.details}")

    print(f"\n{'✅ GATE PASSED' if result.overall_passed else '❌ GATE FAILED'}")

    if not result.overall_passed:
        sys.exit(1)
```

---

## Summary

| Concept | Key Takeaway |
|---|---|
| **GDPR Art. 22** | Automated decisions with legal effect require explanation, human review, and right to contest |
| **Right to erasure** | Must delete user data from training sets on request; may require model retraining |
| **EU AI Act** | Risk-based: prohibited → high-risk → limited → minimal; high-risk needs registration, monitoring, audits |
| **FDA SaMD** | ML medical devices need clinical validation, predetermined change control plans, post-market surveillance |
| **SOC 2 for ML** | Extends trust service criteria to model access, training data security, prediction logging, fallback models |
| **Compliance checklist** | Pre-development, pre-deployment, and post-deployment checklists catch gaps before they become violations |
| **Compliance gate** | Automated CI/CD gate that blocks deployment if regulatory requirements are not met |

> **Rule of thumb:** Compliance is not a one-time checklist — it's a continuous process. Embed regulatory requirements into your ML pipeline (as automated gates) rather than treating them as a separate auditing exercise. The cost of building compliance in from the start is a fraction of the cost of retrofitting it after a regulatory finding.

---

## Revision Questions

1. **A user** in the EU submits a right-to-erasure request under GDPR Art. 17. Your model was trained on their data. What steps must you take, and what are the implications for your deployed model?

2. **Your company** is deploying a credit scoring model in the EU. Which regulations apply? For each, list the three most important requirements and how you would implement them.

3. **Compare** the EU AI Act's requirements for a high-risk AI system with the FDA's requirements for a Class II SaMD. Where do they overlap, and where do they differ?

4. **Design** a SOC 2 control framework for an ML platform that serves multiple models. What additional controls beyond standard SOC 2 are needed, and how would you test them?

5. **Your compliance gate** reveals that a model lacks a fairness audit and a DPIA. The business team wants to deploy urgently. How would you handle this conflict? What risk does deploying without these controls create, and what temporary measures might you put in place?

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Model Governance](./04-model-governance.md) | [📂 Responsible AI](./README.md) | **End of Course** 🎓 |

---

> **End of Unit 11 — Responsible AI in Production.** This concludes the MLOps course. You now have the tools and frameworks to build ML systems that are not only technically sound but also fair, explainable, governed, and compliant.
