# Unit 3: Azure Security — Topic 5: Azure Security Center

## Overview

**Microsoft Defender for Cloud** (formerly Azure Security Center and Azure Defender) is Azure's unified security management system. It provides security posture management (CSPM), threat protection across Azure, hybrid, and multi-cloud workloads, and compliance monitoring. It is the central hub for assessing, improving, and monitoring the security of Azure environments.

---

## 1. Defender for Cloud Overview

```
DEFENDER FOR CLOUD CAPABILITIES:

  ┌──────────────────────────────────────┐
  │     MICROSOFT DEFENDER FOR CLOUD     │
  ├──────────────────┬───────────────────┤
  │  CSPM (Free)     │ CWP (Paid Plans)  │
  │                  │                   │
  │ • Secure Score   │ • Defender for    │
  │ • Recommendations│   Servers         │
  │ • Asset inventory│ • Defender for    │
  │ • Compliance     │   Storage         │
  │ • Policy mgmt   │ • Defender for    │
  │                  │   SQL             │
  │                  │ • Defender for    │
  │                  │   Containers      │
  │                  │ • Defender for    │
  │                  │   Key Vault       │
  │                  │ • Defender for    │
  │                  │   App Service     │
  │                  │ • Defender for    │
  │                  │   DNS             │
  └──────────────────┴───────────────────┘

TIERS:
  Free (CSPM):
  → Security recommendations
  → Secure Score
  → Asset inventory
  → Azure Policy integration
  → Basic compliance assessment
  
  Enhanced (Defender Plans):
  → Per-resource pricing
  → Advanced threat protection
  → Vulnerability scanning
  → Just-in-time VM access
  → Adaptive application controls
  → File integrity monitoring
  → Network hardening recommendations
```

---

## 2. Secure Score

```
SECURE SCORE:

WHAT IS IT?
  → Numerical score (0-100%) representing security posture
  → Based on assessment of active recommendations
  → Higher score = better security posture
  → Tracks improvement over time

CALCULATION:
  → Each control has maximum score points
  → Completing recommendations earns points
  → Score = earned points / total possible points
  
  Example Controls:
  Control                          | Max Points
  Enable MFA                       | 10
  Secure management ports           | 8
  Apply system updates              | 6
  Remediate vulnerabilities         | 6
  Enable encryption at rest         | 4
  Enable auditing and logging       | 4
  Manage access and permissions     | 4

RECOMMENDATIONS EXAMPLE:
  Severity | Recommendation                              | Status
  High     | MFA should be enabled for accounts with     | Unhealthy
           | owner permissions                            |
  High     | Management ports should be closed            | Unhealthy
  Medium   | Diagnostic logs should be enabled            | Unhealthy
  Low      | Email notification for high severity alerts  | Unhealthy

IMPROVING SCORE:
  1. View recommendations in Defender for Cloud
  2. Sort by impact (highest score increase)
  3. Click recommendation for remediation steps
  4. Some have "Quick Fix" buttons
  5. Apply fix → Score improves
  6. Track progress over time

CLI:
  # Get secure score
  az security secure-score list --output table
  
  # List recommendations
  az security assessment list --output table
```

---

## 3. Threat Protection

```
DEFENDER THREAT PROTECTION:

DEFENDER FOR SERVERS:
  → Endpoint Detection and Response (EDR)
  → Powered by Microsoft Defender for Endpoint
  → Vulnerability assessment
  → File integrity monitoring
  → Adaptive application controls
  → Just-in-time VM access
  → Network hardening
  
  Just-In-Time (JIT) VM Access:
  → Block management ports by default
  → Request access when needed
  → Time-limited (max 24 hours)
  → Source IP restricted
  → Audit trail of access

DEFENDER FOR STORAGE:
  → Detect unusual access patterns
  → Malware uploaded to storage
  → Suspicious IP access
  → Data exfiltration patterns
  → Anonymous access anomalies

DEFENDER FOR SQL:
  → SQL injection detection
  → Brute force detection
  → Anomalous queries
  → Data exfiltration alerts
  → Vulnerability assessment

DEFENDER FOR CONTAINERS:
  → Container image scanning (ACR)
  → Kubernetes runtime protection
  → Node-level threat detection
  → Admission control
  → Network-level protection

ALERT TYPES:
  Severity | Example Alert
  High     | Suspicious process execution
  High     | Communication with suspicious IP
  Medium   | Unusual login time/location
  Medium   | Brute force attack detected
  Low      | Unusual outbound traffic
  Low      | Deprecated API usage
```

---

## 4. Compliance

```
REGULATORY COMPLIANCE:

BUILT-IN STANDARDS:
  → Azure Security Benchmark (ASB)
  → CIS Microsoft Azure Foundations
  → NIST SP 800-53
  → PCI DSS
  → ISO 27001
  → SOC 2
  → HIPAA
  → GDPR
  → Custom standards

COMPLIANCE DASHBOARD:
  Standard             | Compliance % | Passed | Failed
  Azure Security       | 72%          | 180    | 70
  CIS Azure            | 65%          | 95     | 52
  PCI DSS              | 68%          | 120    | 57
  NIST 800-53          | 61%          | 200    | 128

HOW IT WORKS:
  1. Select compliance standard
  2. Standard maps to Azure Policy definitions
  3. Policies evaluate resources automatically
  4. Dashboard shows compliance status
  5. Drill down to individual controls
  6. Remediate non-compliant resources
  7. Export compliance reports

CUSTOM INITIATIVES:
  → Create custom compliance standards
  → Map organizational policies
  → Combine existing Azure Policy definitions
  → Track compliance for internal standards
```

---

## 5. Integration and Automation

```
INTEGRATION:

WITH SENTINEL:
  → Forward alerts to Sentinel
  → Correlate with other data sources
  → Automated investigation
  → Playbook-based response
  → Unified incident view

WITH LOGIC APPS:
  → Trigger automated workflows
  → Alert → Logic App → Response
  → Example: Block IP, create ticket, notify team
  → No-code automation

WORKFLOW AUTOMATION:
  → Trigger on recommendations
  → Trigger on alerts
  → Trigger on compliance changes
  → Execute Logic Apps
  → Send to Event Hub

CONTINUOUS EXPORT:
  → Stream data to:
    - Log Analytics workspace
    - Event Hub
    - External SIEM
  → Real-time export
  → Historical data retention

GOVERNANCE:
  → Auto-provision agents
  → Enforce Defender plans across subscriptions
  → Management group-level policies
  → Regular compliance reporting
  → Security governance workflows
```

---

## Summary Table

| Feature | Tier | Key Capability |
|---------|------|---------------|
| Secure Score | Free | Posture assessment |
| Recommendations | Free | Security guidance |
| Compliance | Free | Standard tracking |
| Defender for Servers | Paid | EDR, JIT, FIM |
| Defender for Storage | Paid | Anomaly detection |
| Defender for SQL | Paid | SQL threat protection |
| Defender for Containers | Paid | Container security |

---

## Revision Questions

1. What is the difference between free CSPM and paid Defender plans?
2. How is Secure Score calculated and what does it represent?
3. What is Just-In-Time VM access and why is it important?
4. How does Defender for Cloud handle regulatory compliance?
5. How can Defender for Cloud be integrated with Sentinel for automated response?

---

*Previous: [04-network-security-groups.md](04-network-security-groups.md) | Next: [06-azure-sentinel.md](06-azure-sentinel.md)*

---

*[Back to README](../README.md)*
