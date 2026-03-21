# Unit 2: CompTIA Security+ — Topic 6: Domain 5 — Risk Management

## Overview

**Domain 5: Security Program Management and Oversight (20%)** covers governance, risk management, compliance, and security policies. This domain requires you to "think like a manager" — understanding organizational risk, frameworks, and regulatory requirements.

---

## 1. Risk Management Concepts

```
RISK TERMINOLOGY:

  Term          | Definition
  Risk          | Likelihood × Impact of a threat
  Threat        | Potential cause of harm
  Vulnerability | Weakness that can be exploited
  Asset         | Anything of value to org
  Exploit       | Method to use a vulnerability
  Impact        | Damage caused by a threat
  Likelihood    | Probability of occurrence

RISK FORMULA:
  Risk = Threat × Vulnerability × Impact
  
  Or simplified:
  Risk = Likelihood × Impact

RISK ASSESSMENT TYPES:
  Quantitative:
  → Uses numbers and dollar values
  → SLE = Asset Value × Exposure Factor
  → ALE = SLE × ARO
  → More objective but harder

  Qualitative:
  → Uses categories (High/Med/Low)
  → Risk matrix approach
  → Easier but subjective

  Semi-quantitative:
  → Combination of both
  → Numerical scales for categories

QUANTITATIVE FORMULAS (MUST MEMORIZE):
  → AV (Asset Value): Worth of asset
  → EF (Exposure Factor): % of loss
  → SLE (Single Loss Expectancy): AV × EF
  → ARO (Annual Rate of Occurrence): Frequency
  → ALE (Annual Loss Expectancy): SLE × ARO

EXAMPLE:
  Server worth $50,000 (AV)
  Fire destroys 60% (EF = 0.6)
  SLE = $50,000 × 0.6 = $30,000
  Fire happens once every 5 years (ARO = 0.2)
  ALE = $30,000 × 0.2 = $6,000/year
  → Spend up to $6,000/year on fire mitigation
```

---

## 2. Risk Response Strategies

```
RISK RESPONSE OPTIONS:

  Strategy      | Action              | Example
  Accept        | Acknowledge risk    | Low-impact risk
  Mitigate      | Reduce likelihood   | Install firewall
  Transfer      | Shift to third      | Buy insurance
  Avoid         | Eliminate activity   | Stop using service
  
  NEW TERMS (SY0-701):
  → Risk appetite: How much risk org tolerates
  → Risk tolerance: Range of acceptable risk
  → Risk threshold: Point where action required
  → Residual risk: Risk remaining after controls
  → Inherent risk: Risk before controls

RISK REGISTER:
  → Document listing all identified risks
  → Risk owner assignment
  → Likelihood and impact ratings
  → Current controls
  → Planned treatments
  → Status tracking

RISK MATRIX:
                    Impact
              Low   Med   High
  Likely    | Med | High | Crit |
  Possible  | Low | Med  | High |
  Unlikely  | Low | Low  | Med  |

BUSINESS IMPACT ANALYSIS (BIA):
  → Identify critical business functions
  → Determine maximum tolerable downtime
  → RTO: Recovery Time Objective
  → RPO: Recovery Point Objective
  → MTBF: Mean Time Between Failures
  → MTTR: Mean Time To Repair
```

---

## 3. Security Frameworks and Standards

```
FRAMEWORKS:

NIST FRAMEWORKS:
  → NIST CSF (Cybersecurity Framework):
    Identify → Protect → Detect → Respond → Recover
  → NIST SP 800-53: Security controls
  → NIST SP 800-171: CUI protection
  → NIST RMF: Risk Management Framework

ISO STANDARDS:
  → ISO 27001: ISMS requirements
  → ISO 27002: Security controls
  → ISO 27701: Privacy management
  → ISO 31000: Risk management

INDUSTRY FRAMEWORKS:
  → CIS Controls: Top 18 critical controls
  → COBIT: IT governance
  → ITIL: IT service management
  → MITRE ATT&CK: Adversary tactics

REGULATORY COMPLIANCE:
  Regulation | Applies To           | Focus
  GDPR       | EU data processing   | Privacy
  HIPAA      | Healthcare (US)      | Health data
  PCI-DSS    | Payment cards        | Card data
  SOX        | Public companies     | Financial
  FERPA      | Education (US)       | Student data
  GLBA       | Financial (US)       | Consumer data
  CCPA       | California residents | Privacy

EXAM TIP: Know which framework/regulation 
applies to which scenario. Questions will describe
a situation and ask which applies.
```

---

## 4. Security Policies and Procedures

```
POLICY HIERARCHY:

  ┌─────────────────────────┐
  │   Laws & Regulations    │ ← External
  │   (GDPR, HIPAA, PCI)    │
  ├─────────────────────────┤
  │   Policies              │ ← High-level
  │   (AUP, Password)       │   management
  ├─────────────────────────┤
  │   Standards             │ ← Specific
  │   (Encryption standard) │   requirements
  ├─────────────────────────┤
  │   Procedures            │ ← Step-by-step
  │   (Incident response)   │   instructions
  ├─────────────────────────┤
  │   Guidelines            │ ← Recommended
  │   (Best practices)      │   practices
  └─────────────────────────┘

KEY POLICIES:
  → Acceptable Use Policy (AUP)
  → Password policy
  → Data classification policy
  → Incident response policy
  → Remote access policy
  → BYOD policy
  → Change management policy
  → Data retention policy
  → Clean desk policy

SECURITY AWARENESS:
  → Phishing simulations
  → Annual security training
  → New hire orientation
  → Role-based training
  → Gamification
  → Metrics and reporting
  → Executive buy-in required
```

---

## 5. Third-Party Risk and Auditing

```
THIRD-PARTY RISK:

VENDOR ASSESSMENT:
  → Security questionnaires
  → Right to audit clauses
  → Penetration test results
  → SOC reports (SOC 1, 2, 3)
  → Supply chain risk assessment
  → Due diligence/due care

SOC REPORTS:
  SOC 1: Financial controls
  SOC 2: Security, availability, privacy
    → Type I: Point-in-time
    → Type II: Over a period
  SOC 3: Public summary

AGREEMENTS:
  → SLA (Service Level Agreement)
  → BPA (Business Partners Agreement)
  → NDA (Non-Disclosure Agreement)
  → MOU (Memorandum of Understanding)
  → MOA (Memorandum of Agreement)
  → ISA (Interconnection Security Agreement)

AUDIT TYPES:
  → Internal audit: By organization
  → External audit: By third party
  → Regulatory audit: By authority
  → Compliance scan: Automated
  → Attestation: Formal declaration

DATA GOVERNANCE:
  → Data classification
    Public → Internal → Confidential → 
    Restricted/Top Secret
  → Data roles:
    Owner → Custodian → Steward → User
  → Data retention and disposal
  → Data sovereignty
  → Privacy impact assessment
```

---

## Summary Table

| Topic | Key Concepts | Exam Focus |
|-------|-------------|-----------|
| Risk Concepts | SLE, ALE, ARO formulas | Must memorize |
| Risk Response | Accept, mitigate, transfer, avoid | Know when to use |
| Frameworks | NIST, ISO, CIS, MITRE | Know purposes |
| Policies | Hierarchy, key policies | Know types |
| Third-Party | SOC reports, agreements | Know differences |

---

## Revision Questions

1. How do you calculate ALE using quantitative risk analysis?
2. What is the difference between risk acceptance and risk avoidance?
3. Which framework uses Identify-Protect-Detect-Respond-Recover?
4. What is the difference between SOC 2 Type I and Type II?
5. What agreements establish security expectations with vendors?

---

*Previous: [05-domain-4-identity-access-management.md](05-domain-4-identity-access-management.md) | Next: [07-domain-6-cryptography-and-pki.md](07-domain-6-cryptography-and-pki.md)*

---

*[Back to README](../README.md)*
