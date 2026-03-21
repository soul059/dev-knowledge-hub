# Least Privilege

## Unit 3 - Topic 1 | Security Principles

---

## Overview

The **Principle of Least Privilege (PoLP)** states that every user, program, and process should operate with the minimum level of access necessary to perform their function — and nothing more. It is one of the most fundamental and effective security principles.

---

## 1. The Core Concept

```
┌──────────────────────────────────────────────────────────────────┐
│                 PRINCIPLE OF LEAST PRIVILEGE                     │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  "Give every user and process ONLY the access they NEED         │
│   to do their job — NO MORE."                                    │
│                                                                  │
│  ┌──────────────────────┐    ┌──────────────────────┐           │
│  │  WITHOUT PoLP ❌      │    │  WITH PoLP ✅         │           │
│  │                      │    │                      │           │
│  │  User: "I need to    │    │  User: "I need to    │           │
│  │  edit reports"       │    │  edit reports"       │           │
│  │                      │    │                      │           │
│  │  Access Given:       │    │  Access Given:       │           │
│  │  • ALL files ❌       │    │  • Report folder ✅   │           │
│  │  • Admin panel ❌     │    │  • Edit permission ✅ │           │
│  │  • Database ❌        │    │  • Nothing else      │           │
│  │  • Server config ❌   │    │                      │           │
│  └──────────────────────┘    └──────────────────────┘           │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Why Least Privilege Matters

| Benefit | Description |
|---------|-------------|
| **Limits blast radius** | If account is compromised, damage is contained |
| **Reduces attack surface** | Fewer permissions = fewer things an attacker can exploit |
| **Prevents insider threats** | Users can't access/modify what they don't need |
| **Supports compliance** | Required by HIPAA, PCI-DSS, SOX, GDPR |
| **Minimizes human error** | Users can't accidentally delete/modify critical data |
| **Simplifies auditing** | Easier to track who has access to what |

### Blast Radius Comparison

```
WITHOUT Least Privilege:                WITH Least Privilege:
┌─────────────────────────┐            ┌─────────────────────────┐
│  Compromised Admin      │            │  Compromised User       │
│  Account                │            │  Account                │
│                         │            │                         │
│  Access to:             │            │  Access to:             │
│  ✗ All servers          │            │  ✗ Own files only       │
│  ✗ All databases        │            │  ✗ One application      │
│  ✗ All user data        │            │                         │
│  ✗ Network configs      │            │  Impact: MINIMAL        │
│  ✗ Backup systems       │            │  Contained to one area  │
│                         │            │                         │
│  Impact: CATASTROPHIC   │            └─────────────────────────┘
│  Entire org compromised │
└─────────────────────────┘
```

---

## 3. Implementing Least Privilege

### For Users

| Implementation | Description |
|---------------|-------------|
| **Standard user accounts** | No admin rights for daily use |
| **RBAC** | Assign permissions based on job role |
| **Time-limited access** | Grant elevated access only when needed (JIT) |
| **Regular access reviews** | Quarterly review of who has access to what |
| **Separate admin accounts** | Use different accounts for admin tasks |

### For Systems and Services

| Implementation | Description |
|---------------|-------------|
| **Service accounts** | Run services with minimal permissions |
| **Application permissions** | Apps should only access required resources |
| **Database access** | Read-only where possible; no admin to app users |
| **API permissions** | Scope API keys to specific operations |
| **Container security** | Run containers as non-root |

### For Networks

| Implementation | Description |
|---------------|-------------|
| **Firewall rules** | Default deny, explicitly allow only needed traffic |
| **Network segmentation** | Isolate sensitive systems |
| **VPN access** | Grant access only to required network segments |
| **Zero trust** | Verify every access request regardless of location |

---

## 4. Just-In-Time (JIT) Access

```
┌──────────────────────────────────────────────────────────────┐
│                  JUST-IN-TIME ACCESS                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Normal State:        Elevated State:       After Timeout:   │
│  ┌──────────────┐    ┌──────────────┐      ┌──────────────┐ │
│  │ Standard     │    │ Admin Access │      │ Standard     │ │
│  │ User Access  │───►│ (Approved)   │─────►│ User Access  │ │
│  │              │    │              │      │              │ │
│  │ Request      │    │ Time-limited │      │ Auto-revoke  │ │
│  │ elevation    │    │ (e.g., 4 hrs)│      │ after expiry │ │
│  └──────────────┘    └──────────────┘      └──────────────┘ │
│                                                              │
│  Tools: Azure PIM, CyberArk, BeyondTrust, HashiCorp Vault   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 5. Common Violations

| Violation | Risk | Fix |
|-----------|------|-----|
| Everyone is local admin | Malware runs as admin | Remove admin rights, use PAM |
| Shared admin accounts | No accountability | Individual accounts + MFA |
| Service accounts with domain admin | Compromise = domain takeover | Minimal required permissions |
| No access reviews | Permission creep over time | Quarterly access audits |
| Root access on Linux servers | Total system compromise | sudo with specific commands |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Definition** | Minimum access necessary for the job |
| **Key Benefit** | Limits blast radius of compromise |
| **JIT Access** | Temporary elevation, auto-revoked |
| **Common Violation** | Everyone as admin, no access reviews |
| **Compliance** | Required by HIPAA, PCI-DSS, SOX, GDPR |

---

## Quick Revision Questions

1. **Define the Principle of Least Privilege in one sentence.**
2. **How does PoLP reduce the blast radius of a security incident?**
3. **What is Just-In-Time (JIT) access and why is it important?**
4. **Name 3 common violations of least privilege in organizations.**
5. **How would you implement PoLP for a database administrator?**
6. **Why should service accounts never have domain admin rights?**

---

[Next: Need to Know →](02-need-to-know.md)

---

*Unit 3 - Topic 1 of 6 | [Back to README](../README.md)*
