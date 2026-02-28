# Chapter 1.5: Shared Responsibility Model

## Overview

In DevSecOps, security is a **shared responsibility** across all teams and roles. No single team owns security — developers, operations, security, QA, and management all contribute. This chapter explores how responsibilities are distributed and how organizations implement effective shared security ownership.

---

## The Shared Responsibility Concept

### Traditional vs. Shared Model

```
TRADITIONAL MODEL (Siloed):

  +----------+     +------------+     +----------+
  |   Dev    |     |  Security  |     |   Ops    |
  | "I code" |     | "I test"   |     | "I run"  |
  +----+-----+     +------+-----+     +----+-----+
       |                  |                 |
       |          "Throw over the wall"     |
       |                  |                 |
       v                  v                 v
  +---------+      +-----------+      +---------+
  | Feature |----->| Security  |----->| Deploy  |
  | Done    |      | Review    |      | & Pray  |
  +---------+      +-----------+      +---------+
                        |
                   "47 findings!"
                   "Release blocked"
                        |
                   BOTTLENECK!


SHARED RESPONSIBILITY MODEL:

  +----------------------------------------------------------+
  |                SHARED SECURITY OWNERSHIP                   |
  +----------------------------------------------------------+
  |                                                            |
  |  +-----------+  +-----------+  +-----------+              |
  |  |    Dev    |  | Security  |  |    Ops    |              |
  |  |----------|  |-----------|  |-----------|              |
  |  | - Secure |  | - Tools   |  | - Runtime |              |
  |  |   code   |  | - Policy  |  |   monitor |              |
  |  | - Fix    |  | - Train   |  | - Patch   |              |
  |  |   vulns  |  | - Advise  |  |   mgmt    |              |
  |  | - Tests  |  | - Review  |  | - Incident|              |
  |  +-----------+  +-----------+  +-----------+              |
  |       \              |              /                      |
  |        \             |             /                       |
  |         +------------+------------+                        |
  |                      |                                     |
  |              CONTINUOUS SECURITY                           |
  |              (Everyone owns it)                            |
  +----------------------------------------------------------+
```

---

## RACI Matrix for DevSecOps

```
R = Responsible (does the work)
A = Accountable (owns the outcome)
C = Consulted (provides input)
I = Informed (kept up to date)
```

| Security Activity | Dev | Security | Ops | QA | Management |
|-------------------|-----|----------|-----|-----|------------|
| **Threat Modeling** | C | R/A | C | I | I |
| **Secure Coding** | R/A | C | I | I | I |
| **Code Review (Security)** | R | A | I | C | I |
| **SAST Configuration** | C | R/A | C | I | I |
| **SAST Remediation** | R/A | C | I | I | I |
| **SCA Scanning** | I | R/A | C | I | I |
| **Dependency Updates** | R/A | C | C | I | I |
| **DAST Scanning** | I | R/A | C | C | I |
| **DAST Remediation** | R/A | C | I | C | I |
| **Container Security** | C | C | R/A | I | I |
| **IaC Security** | C | C | R/A | I | I |
| **Secrets Management** | R | C | R/A | I | I |
| **Monitoring & Alerting** | I | C | R/A | I | I |
| **Incident Response** | C | R/A | R | I | A |
| **Compliance** | C | R | C | I | A |
| **Security Training** | I | R/A | I | I | A |
| **Risk Acceptance** | I | C | I | I | R/A |
| **Security Budget** | I | C | I | I | R/A |

---

## Role-Specific Responsibilities

### Developers

```
+================================================================+
|  DEVELOPER SECURITY RESPONSIBILITIES                            |
+================================================================+
|                                                                  |
|  AT CODE TIME:                                                   |
|  [x] Use IDE security plugins (SonarLint, Snyk)                |
|  [x] Follow secure coding standards                             |
|  [x] Input validation and output encoding                       |
|  [x] Use parameterized queries (prevent SQL injection)          |
|  [x] Handle errors securely (no sensitive data in errors)       |
|  [x] Use security libraries (don't roll your own crypto)        |
|                                                                  |
|  AT COMMIT TIME:                                                 |
|  [x] No secrets in code (use environment variables/vaults)      |
|  [x] Run pre-commit security hooks                              |
|  [x] Write security unit tests                                  |
|                                                                  |
|  AT REVIEW TIME:                                                 |
|  [x] Review PRs for security issues                             |
|  [x] Check SAST/SCA results before merging                      |
|  [x] Remediate findings within SLA                               |
|                                                                  |
|  ONGOING:                                                        |
|  [x] Complete security training                                  |
|  [x] Stay updated on new vulnerabilities                        |
|  [x] Participate in threat modeling                              |
+================================================================+
```

### Security Team

```
+================================================================+
|  SECURITY TEAM RESPONSIBILITIES                                 |
+================================================================+
|                                                                  |
|  ENABLE (Not Gate):                                              |
|  [x] Select and configure security tools                         |
|  [x] Define security policies and standards                      |
|  [x] Create secure coding guidelines                             |
|  [x] Build reusable security libraries                           |
|  [x] Provide secure defaults ("paved roads")                    |
|                                                                  |
|  TRAIN:                                                          |
|  [x] Conduct security training programs                          |
|  [x] Run CTF exercises and war games                             |
|  [x] Mentor security champions                                   |
|  [x] Create security documentation and playbooks                |
|                                                                  |
|  ASSESS:                                                         |
|  [x] Conduct penetration testing                                 |
|  [x] Perform architecture security reviews                      |
|  [x] Manage vulnerability disclosure programs                    |
|  [x] Oversee compliance requirements                             |
|                                                                  |
|  RESPOND:                                                        |
|  [x] Lead incident response                                     |
|  [x] Conduct forensic analysis                                   |
|  [x] Manage threat intelligence                                  |
|  [x] Coordinate with external security researchers              |
+================================================================+
```

### Operations Team

```
+================================================================+
|  OPERATIONS TEAM RESPONSIBILITIES                                |
+================================================================+
|                                                                  |
|  INFRASTRUCTURE:                                                 |
|  [x] Secure infrastructure configuration                         |
|  [x] Implement network segmentation                              |
|  [x] Manage firewalls and WAFs                                   |
|  [x] Apply CIS benchmarks                                       |
|                                                                  |
|  DEPLOYMENT:                                                     |
|  [x] Validate IaC security before apply                          |
|  [x] Manage secrets and certificates                             |
|  [x] Implement least-privilege access                            |
|  [x] Secure CI/CD pipeline infrastructure                       |
|                                                                  |
|  MONITORING:                                                     |
|  [x] Deploy SIEM and monitoring tools                            |
|  [x] Configure security alerts and dashboards                    |
|  [x] Monitor for anomalous behavior                              |
|  [x] Manage log aggregation and retention                       |
|                                                                  |
|  MAINTENANCE:                                                    |
|  [x] Patch management and updates                                |
|  [x] Rotate secrets and certificates                             |
|  [x] Disaster recovery and backup security                      |
|  [x] Capacity planning for security tools                       |
+================================================================+
```

---

## Cloud Shared Responsibility Model

In cloud environments, responsibility is split between the cloud provider and the customer.

```
+=======================================================================+
|              CLOUD SHARED RESPONSIBILITY MODEL                         |
+=======================================================================+
|                                                                         |
|  IaaS              PaaS              SaaS                              |
|  (e.g., EC2)       (e.g., RDS)       (e.g., Office365)                |
|                                                                         |
|  +-------------+   +-------------+   +-------------+                   |
|  |             |   |             |   |             |                   |
|  | Data        | C | Data        | C | Data        | C                 |
|  | App         | U | App         | U | Config      | U                 |
|  | Middleware  | S | Identity    | S | Identity    | S                 |
|  | OS          | T |             | T |             | T                 |
|  | Network*    | O |             | O |             | O                 |
|  | Identity    | M |             | M |             | M                 |
|  |             | E |             | E |             | E                 |
|  |             | R |             | R |             | R                 |
|  |-------------|   |-------------|   |-------------|                   |
|  | Virtualizat.| P | Middleware  | P | App         | P                 |
|  | Storage     | R | OS          | R | Middleware  | R                 |
|  | Networking  | O | Runtime     | O | OS          | O                 |
|  | Physical    | V | Virtualizat.| V | Runtime     | V                 |
|  |             | I | Storage     | I | Virtualizat.| I                 |
|  |             | D | Networking  | D | Storage     | D                 |
|  |             | E | Physical    | E | Networking  | E                 |
|  |             | R |             | R | Physical    | R                 |
|  +-------------+   +-------------+   +-------------+                   |
|                                                                         |
|  KEY:  CUSTOMER = You secure this                                       |
|        PROVIDER = Cloud provider secures this                           |
+=======================================================================+
```

### AWS Shared Responsibility

```
+----------------------------------------------------------------+
|                  AWS RESPONSIBILITY (OF the Cloud)               |
+----------------------------------------------------------------+
| Hardware / AWS Global Infrastructure                             |
| Regions, Availability Zones, Edge Locations                      |
| Compute, Storage, Database, Networking (physical)                |
| Managed Services infrastructure                                  |
+----------------------------------------------------------------+
                              |
                    Shared Controls
                    (Patch mgmt, config mgmt, awareness)
                              |
+----------------------------------------------------------------+
|                  CUSTOMER RESPONSIBILITY (IN the Cloud)          |
+----------------------------------------------------------------+
| Customer Data                                                    |
| Platform, Applications, Identity & Access Management             |
| Operating System, Network & Firewall Configuration              |
| Client-side encryption, Server-side encryption                  |
| Network traffic protection, Security group config               |
+----------------------------------------------------------------+
```

---

## Implementing Shared Responsibility

### Step-by-Step Approach

```
PHASE 1: ASSESS (Weeks 1-2)
+----------------------------------------------------------+
| - Map current security responsibilities                   |
| - Identify gaps and overlaps                              |
| - Survey teams on security awareness                      |
| - Assess current tools and processes                      |
+----------------------------------------------------------+
                        |
                        v
PHASE 2: DEFINE (Weeks 3-4)
+----------------------------------------------------------+
| - Create RACI matrix for all security activities          |
| - Define security policies and standards                  |
| - Establish SLAs for vulnerability remediation            |
| - Get management buy-in and budget                        |
+----------------------------------------------------------+
                        |
                        v
PHASE 3: IMPLEMENT (Weeks 5-12)
+----------------------------------------------------------+
| - Deploy security tools in CI/CD pipeline                 |
| - Train developers (Level 1 security training)            |
| - Launch Security Champions program                       |
| - Establish security gates in pipeline                    |
+----------------------------------------------------------+
                        |
                        v
PHASE 4: OPERATE (Ongoing)
+----------------------------------------------------------+
| - Monitor security metrics                                |
| - Conduct regular security reviews                        |
| - Iterate on processes based on feedback                  |
| - Continuous training and improvement                     |
+----------------------------------------------------------+
```

### Vulnerability Remediation SLAs

```
+------------------+------------------+------------------------+
|  Severity        |  Remediation SLA |  Escalation            |
+------------------+------------------+------------------------+
|  CRITICAL        |  24 hours        |  Immediate to CISO     |
|  (CVSS 9.0-10.0) |                  |  War room if needed    |
+------------------+------------------+------------------------+
|  HIGH            |  7 days          |  After 3 days to       |
|  (CVSS 7.0-8.9)  |                  |  engineering manager   |
+------------------+------------------+------------------------+
|  MEDIUM          |  30 days         |  After 14 days to      |
|  (CVSS 4.0-6.9)  |                  |  team lead             |
+------------------+------------------+------------------------+
|  LOW             |  90 days         |  Tracked in backlog    |
|  (CVSS 0.1-3.9)  |                  |  Monthly review        |
+------------------+------------------+------------------------+
|  INFO            |  Best effort     |  No SLA, but tracked   |
|  (CVSS 0.0)      |                  |                        |
+------------------+------------------+------------------------+
```

---

## Real-World Scenario: Capital One Breach (2019)

```
WHAT HAPPENED:
+----------------------------------------------------------------+
| A former AWS employee exploited a misconfigured WAF to access  |
| Capital One's AWS S3 buckets, exposing 100 million records.     |
+----------------------------------------------------------------+

SHARED RESPONSIBILITY FAILURE:

  AWS Responsibility (MET):
  +------------------------------------------------------------+
  | - Physical security of data centers          [PASS]        |
  | - Hypervisor and hardware security           [PASS]        |
  | - S3 service availability                    [PASS]        |
  +------------------------------------------------------------+

  Capital One Responsibility (GAPS):
  +------------------------------------------------------------+
  | - WAF configuration (too permissive)         [FAIL]        |
  | - IAM role permissions (overly broad)        [FAIL]        |
  | - Server-Side Request Forgery protection     [FAIL]        |
  | - S3 bucket access policies                  [FAIL]        |
  | - Monitoring for unusual data access         [DELAYED]     |
  +------------------------------------------------------------+

LESSON: Even in the cloud, YOU are responsible for:
  - Proper configuration
  - Access control
  - Network security
  - Data protection
  - Monitoring
```

### Prevention with DevSecOps

```yaml
# Example: Checkov check for S3 bucket security
# This would have caught the misconfiguration

# terraform/s3.tf
resource "aws_s3_bucket" "sensitive_data" {
  bucket = "capital-one-sensitive-data"
  
  # Checkov would flag if these are missing:
  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "aws:kms"
      }
    }
  }
}

resource "aws_s3_bucket_public_access_block" "sensitive_data" {
  bucket = aws_s3_bucket.sensitive_data.id
  
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

# IAM policy - least privilege
resource "aws_iam_role_policy" "app_role" {
  name = "app-role-policy"
  role = aws_iam_role.app_role.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "s3:GetObject"  # Only what's needed, not s3:*
        ]
        Resource = "arn:aws:s3:::specific-bucket/specific-prefix/*"
      }
    ]
  })
}
```

---

## Communication & Escalation

```
SECURITY ESCALATION FLOW:

  +----------------+
  | Developer      |     Finds vulnerability
  | detects issue  |     in code or scan
  +-------+--------+
          |
          v
  +-------+--------+     Can fix in < 4 hours?
  | Self-fix or    |----YES----> Fix, commit, verify
  | escalate?      |
  +-------+--------+
          |
         NO
          |
          v
  +-------+--------+
  | Security       |     Champion triages
  | Champion       |     severity
  +-------+--------+
          |
     +----+----+
     |         |
  MEDIUM    CRITICAL
   /LOW      /HIGH
     |         |
     v         v
  Backlog   +--------+--------+
  ticket    | Security Team   |
            | engaged         |
            +--------+--------+
                     |
                   WAR ROOM
                   (if needed)
                     |
                     v
            +--------+--------+
            | Fix deployed    |
            | Post-mortem     |
            +-----------------+
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Unclear ownership | No RACI defined | Create and publish RACI matrix |
| "Not my job" attitude | Poor communication | Regular cross-team meetings, shared OKRs |
| Cloud misconfigurations | Misunderstood cloud responsibilities | Cloud security training, automated IaC scanning |
| SLA violations | No tracking or accountability | Automate SLA tracking, dashboard visibility |
| Shared responsibility fatigue | Too many responsibilities spread thin | Prioritize by risk, automate what you can |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Shared Responsibility** | Security ownership distributed across all team roles |
| **RACI Matrix** | Framework defining who is Responsible, Accountable, Consulted, Informed |
| **Developer Responsibilities** | Secure coding, fixing SAST/SCA findings, security tests |
| **Security Team Role** | Enable, train, assess, respond — NOT gate |
| **Ops Responsibilities** | Infrastructure security, monitoring, patching, secrets |
| **Cloud Shared Model** | Split between cloud provider (of) and customer (in the cloud) |
| **Remediation SLAs** | Time-based targets for fixing vulnerabilities by severity |
| **Escalation Flow** | Clear path from detection to resolution |

---

## Quick Revision Questions

1. **What does "shared responsibility" mean in DevSecOps?**
   > Every team member — developers, operations, security, QA, and management — has specific security responsibilities. Security is not delegated solely to the security team but is integrated into everyone's workflow.

2. **What is a RACI matrix and how does it help DevSecOps?**
   > RACI (Responsible, Accountable, Consulted, Informed) is a framework that clearly defines who does what for each security activity, preventing gaps and overlaps in security ownership.

3. **In the AWS shared responsibility model, who is responsible for S3 bucket permissions?**
   > The customer. AWS secures the S3 service infrastructure ("security OF the cloud"), but the customer is responsible for configuring bucket policies, access controls, and encryption ("security IN the cloud").

4. **What is the recommended remediation SLA for a CRITICAL vulnerability (CVSS 9.0+)?**
   > 24 hours, with immediate escalation to the CISO and a war room if needed.

5. **How does the security team's role change in a DevSecOps shared responsibility model?**
   > The security team shifts from being a gatekeeper to an enabler — they select tools, define policies, provide training, and support teams, rather than being the sole reviewer/approver of all security decisions.

6. **What lesson does the Capital One breach teach about shared responsibility in the cloud?**
   > Even when using a major cloud provider like AWS, the customer is responsible for proper configuration (WAF, IAM, S3 policies) and monitoring. The breach was caused by customer misconfigurations, not AWS infrastructure failures.

---

[← Previous: 1.4 Security Culture](04-security-culture.md) | [Next: Unit 2 - 2.1 OWASP Top 10 →](../02-Secure-Coding-Practices/01-owasp-top-10.md)

[Back to Table of Contents](../README.md)
