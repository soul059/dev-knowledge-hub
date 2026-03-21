# Unit 3: Threat Detection — Topic 1: Detection Engineering

## Overview

**Detection engineering** is the discipline of designing, building, testing, and maintaining detection logic that identifies threats in security data. It applies software engineering principles to security detection content, treating detection rules as code with version control, testing, and continuous improvement. Detection engineering is the foundation of a mature SOC's ability to find threats.

---

## 1. Detection Engineering Fundamentals

```
WHAT IS DETECTION ENGINEERING?

  The practice of systematically creating, testing,
  deploying, and maintaining security detection content.

  ┌──────────────────────────────────────────────────┐
  │           DETECTION ENGINEERING LIFECYCLE         │
  │                                                    │
  │  ┌─────────┐  ┌──────────┐  ┌─────────────────┐  │
  │  │RESEARCH │─▶│  BUILD   │─▶│  TEST/VALIDATE  │  │
  │  │         │  │          │  │                 │  │
  │  │Threat   │  │Write rule│  │Red team/atomic  │  │
  │  │analysis │  │Map ATT&CK│  │test, validate   │  │
  │  └─────────┘  └──────────┘  └────────┬────────┘  │
  │       ▲                              │            │
  │       │                              ▼            │
  │  ┌────┴──────┐  ┌──────────┐  ┌──────────────┐   │
  │  │  IMPROVE  │◀─│ MONITOR  │◀─│   DEPLOY     │   │
  │  │           │  │          │  │              │   │
  │  │Tune, fix  │  │Track FP, │  │Production    │   │
  │  │update     │  │accuracy  │  │SIEM/EDR      │   │
  │  └───────────┘  └──────────┘  └──────────────┘   │
  └──────────────────────────────────────────────────┘

DETECTION ENGINEERING vs RULE WRITING:
  Rule Writing:
  → Ad-hoc rule creation
  → No testing process
  → No version control
  → No metrics on effectiveness
  
  Detection Engineering:
  → Systematic methodology
  → Version-controlled rules
  → Automated testing
  → Effectiveness metrics
  → Continuous improvement
  → ATT&CK-mapped coverage
```

---

## 2. Detection Development Process

```
DEVELOPMENT WORKFLOW:

STEP 1: THREAT RESEARCH
  → Identify threat to detect
  → Study attack technique (ATT&CK)
  → Understand data requirements
  → Identify data sources needed
  → Review existing detections
  
  Sources:
  → MITRE ATT&CK descriptions
  → Threat intelligence reports
  → Red team findings
  → Incident post-mortems
  → Public threat research
  → Vendor advisories

STEP 2: DATA ANALYSIS
  → Verify required logs are collected
  → Understand log format and fields
  → Identify normal vs malicious patterns
  → Determine baseline behavior
  → Assess data quality

STEP 3: RULE DEVELOPMENT
  → Write detection logic
  → Choose rule format (Sigma, native)
  → Map to ATT&CK technique
  → Set appropriate severity
  → Add context and enrichment
  → Write analyst guidance

STEP 4: TESTING
  → Test with known attack data
  → Use atomic red team tests
  → Validate true positive detection
  → Measure false positive rate
  → Test at production scale
  → Peer review

STEP 5: DEPLOYMENT
  → Deploy to staging/test environment
  → Monitor for initial period
  → Tune based on early results
  → Move to production
  → Document in detection catalog

STEP 6: MAINTENANCE
  → Regular review cycle
  → Tune based on analyst feedback
  → Update for new attack variants
  → Retire stale detections
  → Track effectiveness metrics
```

---

## 3. Detection Quality Framework

```
DETECTION QUALITY METRICS:

DETECTION MATURITY LEVEL:
  Level 1: Basic signatures
    → Known IOCs, simple patterns
    → Easy to evade
    → High false positive rate
  
  Level 2: Enhanced signatures
    → Multiple conditions
    → Better context
    → Moderate evasion resistance
  
  Level 3: Behavioral detection
    → Detects technique, not specific tool
    → Process relationships
    → Harder to evade
  
  Level 4: Advanced behavioral
    → Statistical baselines
    → Multiple data source correlation
    → Tool-agnostic detection
  
  Level 5: ML/Analytics-based
    → Anomaly detection
    → User behavior analytics
    → Predictive detection

PYRAMID OF PAIN (Detection Value):
  
       /\
      /  \  TTPs
     /    \  ← Hardest to change (most valuable)
    /──────\
   / Tools  \
  /──────────\
  Network/Host \
  ──────────────\
  Domain Names   \
  ────────────────\
  IP Addresses     \
  ──────────────────\
  Hash Values       \  ← Easiest to change (least valuable)
  ────────────────────

QUALITY CHECKLIST:
  [ ] Detects the intended technique
  [ ] Tested with attack simulation
  [ ] False positive rate acceptable (<30%)
  [ ] Mapped to MITRE ATT&CK
  [ ] Analyst runbook/playbook exists
  [ ] Severity accurately reflects risk
  [ ] Data source requirements documented
  [ ] Peer reviewed
  [ ] Version controlled
  [ ] Documented in detection catalog
```

---

## 4. Detection Testing

```
TESTING METHODOLOGIES:

ATOMIC RED TEAM:
  → Pre-built test cases for ATT&CK techniques
  → Automated execution
  → Validate detection works
  
  # Run a test
  Invoke-AtomicTest T1003.001  # Credential Dumping
  Invoke-AtomicTest T1059.001  # PowerShell execution
  Invoke-AtomicTest T1547.001  # Registry Run Keys
  
  # Run all tests for a tactic
  Invoke-AtomicTest T1003 -GetPrereqs
  Invoke-AtomicTest T1003 -ShowDetailsBrief

DETECTION TESTING PIPELINE:
  ┌──────────┐    ┌──────────┐    ┌──────────┐
  │  Write   │───▶│ Execute  │───▶│ Verify   │
  │  Rule    │    │ Attack   │    │ Alert    │
  └──────────┘    │ Simulation│   │ Generated│
                  └──────────┘    └──────────┘
                       │               │
                  ┌────▼────┐    ┌─────▼─────┐
                  │ Atomic  │    │ Detection │
                  │ Red Team│    │ Validated │
                  │ Caldera │    │ or Failed │
                  └─────────┘    └───────────┘

PURPLE TEAMING:
  → Red team executes techniques
  → Blue team validates detection
  → Joint effort to improve coverage
  → Iterative improvement cycle
  → Documents gaps and fixes

DETECTION VALIDATION TOOLS:
  → Atomic Red Team (open source)
  → MITRE Caldera (adversary simulation)
  → AttackIQ (commercial)
  → SafeBreach (commercial)
  → Cymulate (commercial)
  → Detection Lab (lab environment)
```

---

## 5. Detection Catalog

```
MANAGING DETECTIONS:

DETECTION CATALOG:
  → Central repository of all detections
  → Metadata for each detection
  → ATT&CK mapping
  → Status tracking
  → Effectiveness metrics

  Catalog Entry:
  ┌──────────────────────────────────────┐
  │ Detection ID: DET-2024-001          │
  │ Title: Brute Force Authentication   │
  │ ATT&CK: T1110.001                  │
  │ Status: Active                       │
  │ Severity: High                       │
  │ SIEM: Splunk ES                     │
  │ Data Sources: Windows Security Log  │
  │ Created: 2024-01-15                 │
  │ Last Updated: 2024-06-01            │
  │ Author: Detection Engineering Team  │
  │ FP Rate: 8%                         │
  │ TP Count (30d): 12                  │
  │ Playbook: PB-AUTH-001              │
  │ Test: Atomic T1110.001             │
  └──────────────────────────────────────┘

COVERAGE TRACKING:
  → Map detections to ATT&CK matrix
  → Identify gaps in coverage
  → Prioritize new detections
  → Track coverage improvement over time
  → Report to leadership

TOOLS:
  → Sigma (detection rule format)
  → Git (version control for rules)
  → ATT&CK Navigator (coverage visualization)
  → DeTT&CT (detection coverage mapping)
  → Detection catalog (custom/spreadsheet)
```

---

## Summary Table

| Aspect | Description | Key Metric |
|--------|-------------|------------|
| Development | Research → Build → Test → Deploy | Time to create |
| Quality | Maturity levels 1-5 | Detection level |
| Testing | Atomic Red Team, purple team | Test pass rate |
| Maintenance | Tune, update, retire | FP rate trend |
| Coverage | ATT&CK mapping | % techniques covered |

---

## Revision Questions

1. How does detection engineering differ from ad-hoc rule writing?
2. What are the steps in the detection development lifecycle?
3. How does the Pyramid of Pain guide detection strategy?
4. What tools can validate that detections are working?
5. What should a detection catalog include for each rule?

---

*Previous: None (First topic in this unit) | Next: [02-indicator-based-detection.md](02-indicator-based-detection.md)*

---

*[Back to README](../README.md)*
