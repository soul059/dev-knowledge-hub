# Unit 5: Cloud IAM Security — Topic 6: Privileged Access Management

## Overview

**Privileged Access Management (PAM)** in cloud environments controls and monitors access to critical resources by privileged identities. Cloud PAM encompasses just-in-time access, privilege elevation, session recording, and continuous monitoring of administrative activities. As cloud environments grow, PAM becomes essential for preventing privilege abuse and meeting compliance requirements.

---

## 1. PAM Fundamentals

```
PRIVILEGED ACCESS IN CLOUD:

WHAT IS PRIVILEGED ACCESS?
  → Administrative access to cloud resources
  → Root/admin account usage
  → Infrastructure modification capabilities
  → Security configuration changes
  → Data access and export abilities
  → Identity and access management
  → Billing and financial operations

RISK OF UNMANAGED PRIVILEGE:
  → Standing admin access = always at risk
  → Compromised admin = full breach
  → Insider threat amplified
  → No accountability for actions
  → Compliance violations

PAM PRINCIPLES:
  ┌────────────────────────────────────────┐
  │        PRIVILEGED ACCESS MANAGEMENT    │
  │                                        │
  │  1. DISCOVER → Find all privileged     │
  │     accounts and access paths          │
  │                                        │
  │  2. MANAGE → Vault credentials,        │
  │     eliminate standing access           │
  │                                        │
  │  3. CONTROL → JIT access, approval     │
  │     workflows, session management      │
  │                                        │
  │  4. MONITOR → Log all privileged       │
  │     activity, alert on anomalies       │
  │                                        │
  │  5. REVIEW → Regular access reviews,   │
  │     certification, cleanup             │
  │                                        │
  │  6. RESPOND → Automated response       │
  │     to privilege abuse                 │
  └────────────────────────────────────────┘
```

---

## 2. Cloud Provider PAM Solutions

```
AZURE PIM:
  → Privileged Identity Management
  → Just-in-time role activation
  → Time-bound assignments
  → Approval workflows
  → MFA on activation
  → Access reviews

  How It Works:
  1. Admin is "eligible" (not active)
  2. Requests activation when needed
  3. Provides justification
  4. Approval (if configured)
  5. MFA verification
  6. Role active for X hours
  7. Auto-deactivates

  Configuration:
  → Max activation duration: 1-24 hours
  → Require MFA: Yes
  → Require justification: Yes
  → Require approval: Yes (for critical roles)
  → Notification to approvers
  → Activation alerts

AWS PRIVILEGED ACCESS:
  → AWS IAM Identity Center (SSO)
  → Temporary role assumption via STS
  → Permission boundaries
  → Service Control Policies (SCPs)
  → No built-in PIM equivalent
  
  Alternatives:
  → CyberArk for AWS
  → HashiCorp Vault AWS secrets engine
  → Custom solutions with Lambda + Step Functions
  → AWS SSO with JIT provisioning

  AWS STS (Security Token Service):
  # Assume role with limited duration
  aws sts assume-role \
    --role-arn arn:aws:iam::123456:role/AdminRole \
    --role-session-name "admin-session" \
    --duration-seconds 3600

GCP PRIVILEGED ACCESS:
  → IAM Conditions (time-based)
  → Organization policies
  → No built-in PIM equivalent
  → BeyondCorp Enterprise
  
  Temporary IAM Binding:
  # Grant role with expiration
  gcloud projects add-iam-policy-binding PROJECT_ID \
    --member="user:admin@example.com" \
    --role="roles/owner" \
    --condition="expression=request.time < timestamp('2024-01-16T00:00:00Z'),title=temp-access"
```

---

## 3. PAM Architecture

```
ENTERPRISE PAM ARCHITECTURE:

  ┌─────────────────────────────────────────┐
  │           PAM SOLUTION                   │
  │                                         │
  │  ┌──────────┐  ┌──────────────────────┐ │
  │  │ Password  │  │ Session Management  │ │
  │  │ Vault     │  │ Recording/Proxy     │ │
  │  └──────────┘  └──────────────────────┘ │
  │                                         │
  │  ┌──────────┐  ┌──────────────────────┐ │
  │  │ JIT       │  │ Threat Analytics    │ │
  │  │ Access    │  │ Behavior Monitoring │ │
  │  └──────────┘  └──────────────────────┘ │
  │                                         │
  │  ┌──────────┐  ┌──────────────────────┐ │
  │  │ MFA/Auth  │  │ Compliance          │ │
  │  │ Gateway   │  │ Reporting           │ │
  │  └──────────┘  └──────────────────────┘ │
  └─────────────────────────────────────────┘
       │           │            │
       ▼           ▼            ▼
    ┌──────┐  ┌────────┐  ┌─────────┐
    │ AWS  │  │ Azure  │  │  GCP    │
    └──────┘  └────────┘  └─────────┘

PAM SOLUTIONS:
  → CyberArk Privileged Access Security
  → BeyondTrust
  → Thycotic/Delinea Secret Server
  → HashiCorp Vault
  → Teleport (open source)
  → StrongDM
  → Azure PIM (native)

KEY CAPABILITIES:
  Password Vaulting:
  → Centralized credential storage
  → Automatic password rotation
  → Check-out/check-in workflow
  → No standing passwords

  Session Management:
  → Proxy all privileged sessions
  → Record SSH, RDP, database sessions
  → Real-time monitoring
  → Session replay for audit

  Just-In-Time Access:
  → Request-based privilege elevation
  → Time-limited access
  → Approval workflows
  → Automatic de-provisioning
```

---

## 4. Break-Glass Procedures

```
BREAK-GLASS (EMERGENCY ACCESS):

PURPOSE:
  → Access when normal processes fail
  → IdP outage, PAM system down
  → Critical incident requiring immediate action
  → Disaster recovery scenarios

BREAK-GLASS ACCOUNTS:
  → Non-federated (local cloud accounts)
  → Strong, unique passwords
  → Hardware MFA (FIDO2)
  → Stored in physical safe or secure vault
  → Multiple custodians required
  → Never used for routine access

IMPLEMENTATION:
  AWS:
  → Root account with MFA (rarely used)
  → IAM user "break-glass" with MFA
  → Stored in corporate vault
  → CloudTrail alerts on usage
  
  Azure:
  → Non-federated Global Admin account
  → Not synced via AD Connect
  → Strong password + FIDO2
  → Conditional Access exclusion
  → Usage alerts configured
  
  GCP:
  → Super Admin in Cloud Identity
  → Not federated
  → Hardware security key
  → Monitored by Admin audit logs

PROCEDURES:
  1. Document break-glass scenarios
  2. Create break-glass accounts
  3. Store credentials securely
  4. Configure monitoring alerts
  5. Test quarterly (tabletop or actual)
  6. Post-usage review and password rotation
  7. Document every break-glass usage

MONITORING:
  → Immediate alert on break-glass login
  → SMS/call to security team
  → Automatic ticket creation
  → Session recording
  → Post-usage audit required
```

---

## 5. Assessment and Compliance

```
PAM SECURITY ASSESSMENT:

AUDIT CHECKLIST:
  [ ] All privileged accounts inventoried
  [ ] No shared admin credentials
  [ ] MFA enforced for all privileged access
  [ ] Just-in-time access implemented
  [ ] Standing privileges minimized
  [ ] Privileged sessions recorded
  [ ] Break-glass procedures documented
  [ ] Break-glass tested regularly
  [ ] Regular access reviews conducted
  [ ] Privileged activity monitored

COMPLIANCE REQUIREMENTS:
  Standard    | PAM Requirement
  PCI DSS     | Unique IDs, MFA for admin, audit logs
  SOX         | Access controls, segregation of duties
  HIPAA       | Minimum necessary, audit controls
  SOC 2       | Logical access controls, monitoring
  ISO 27001   | Access control policy, privileged access
  NIST 800-53 | AC-6 (Least Privilege), AU-2 (Audit)

METRICS:
  → % of privileged access that is JIT (target: >90%)
  → Average privilege duration (target: <4 hours)
  → Number of standing privileged accounts (target: 0)
  → Time to revoke compromised privilege (target: <1 hour)
  → Access review completion rate (target: 100%)
  → Break-glass usage frequency (target: near 0)

COMMON FINDINGS:
  Finding                        | Risk     | Remediation
  Standing Owner/Admin access    | Critical | Implement PIM/JIT
  Shared admin credentials       | Critical | Individual accounts + MFA
  No session recording           | High     | Deploy session proxy
  No break-glass procedures      | High     | Create and test
  Infrequent access reviews      | Medium   | Quarterly automated reviews
  Root account without MFA       | Critical | Enable hardware MFA
  No alerts on privileged access | High     | Configure monitoring
```

---

## Summary Table

| Cloud | PAM Solution | JIT Access | Key Feature |
|-------|-------------|------------|-------------|
| Azure | PIM (native) | Yes | Approval workflows |
| AWS | STS + custom | Partial | Temporary credentials |
| GCP | IAM Conditions | Partial | Time-based bindings |
| Multi-cloud | CyberArk/Vault | Yes | Password vaulting |

---

## Revision Questions

1. What is the difference between just-in-time and standing privileged access?
2. How does Azure PIM implement privilege elevation?
3. Why are break-glass accounts necessary and how should they be managed?
4. What should a privileged access management audit cover?
5. What metrics indicate the maturity of a PAM program?

---

*Previous: [05-secrets-management.md](05-secrets-management.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*
