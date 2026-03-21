# Separation of Duties

## Unit 3 - Topic 3 | Security Principles

---

## Overview

**Separation of Duties (SoD)** is a security principle that ensures no single individual has enough authority or access to commit fraud, make critical errors, or cause significant damage undetected. Critical tasks are divided among multiple people so that collusion would be required to abuse the system.

---

## 1. Core Concept

```
┌──────────────────────────────────────────────────────────────────┐
│                   SEPARATION OF DUTIES                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  WITHOUT SoD ❌                     WITH SoD ✅                    │
│  ──────────────                     ──────────                   │
│                                                                  │
│  One person can:                    Different people:             │
│  ├── Create a vendor  ┐            ├── Person A: Creates vendor  │
│  ├── Approve payment  │ = FRAUD!   ├── Person B: Approves payment│
│  └── Issue the check  ┘            └── Person C: Issues check    │
│                                                                  │
│  "No single person should control ALL steps of a                │
│   critical process."                                             │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Real-World Examples

### Financial Process

```
┌──────────────────────────────────────────────────────────────┐
│              PAYMENT PROCESSING - SoD APPLIED                 │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────┐   ┌──────────────┐   ┌─────────────┐      │
│  │  Employee A  │──►│  Manager B   │──►│  Finance C  │      │
│  │  Requests    │   │  Approves    │   │  Processes  │      │
│  │  purchase    │   │  request     │   │  payment    │      │
│  └─────────────┘   └──────────────┘   └─────────────┘      │
│                                                              │
│  No single person can request AND approve AND pay.           │
│  Fraud requires collusion between multiple people.           │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### IT Security Examples

| Process | Separation |
|---------|-----------|
| **Code Development** | Developer writes code → Different person reviews → Another deploys |
| **User Account Management** | Requester → Approver → Implementer |
| **Firewall Changes** | Request change → Approve change → Implement change → Verify change |
| **Backup & Restore** | Admin creates backups → Different admin tests restoration |
| **Database Administration** | DBA manages schema → Security team manages access → Auditor reviews |

---

## 3. SoD in Cybersecurity

| Role | Should NOT Also | Why |
|------|----------------|-----|
| **Security auditor** | Be a system administrator | Can't audit your own work objectively |
| **Developer** | Deploy to production | Prevents inserting unauthorized code |
| **Network admin** | Review firewall logs | Shouldn't audit own changes |
| **Key custodian** | Be the only key holder | Prevents single point of compromise |
| **Incident responder** | Be the only forensic analyst | Prevents evidence tampering |

---

## 4. Dual Control vs Separation of Duties

```
┌──────────────────────────────────────────────────────────────┐
│        DUAL CONTROL vs SEPARATION OF DUTIES                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  SEPARATION OF DUTIES:                                       │
│  Different people handle different STEPS of a process        │
│  Person A ──► Step 1 (Request)                               │
│  Person B ──► Step 2 (Approve)                               │
│  Person C ──► Step 3 (Execute)                               │
│                                                              │
│  DUAL CONTROL:                                               │
│  Multiple people must act TOGETHER on the SAME step          │
│  Person A + Person B ──► BOTH needed to open vault           │
│  Person A + Person B ──► BOTH needed to launch missile       │
│                                                              │
│  Split Knowledge (related concept):                          │
│  Person A knows half the combination                         │
│  Person B knows the other half                               │
│  Neither can open alone                                      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 5. Challenges and Solutions

| Challenge | Solution |
|-----------|---------|
| Small teams (not enough people) | Compensating controls (extra logging, auditing) |
| Slows down processes | Automate approval workflows |
| Role confusion | Clear RACI matrix (Responsible, Accountable, Consulted, Informed) |
| Cost of additional staff | Risk-based approach — SoD for critical processes only |
| Emergency situations | Break-glass procedures with full audit trail |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Definition** | No single person controls all steps of a critical process |
| **Purpose** | Prevent fraud, errors, and abuse of authority |
| **Key Idea** | Requires collusion to bypass |
| **Related** | Dual control (both needed), split knowledge (each knows part) |
| **Challenge** | Small orgs — use compensating controls |
| **Compliance** | Required by SOX, PCI-DSS, banking regulations |

---

## Quick Revision Questions

1. **What is Separation of Duties and why is it important?**
2. **Give 3 IT security examples where SoD should be applied.**
3. **What is the difference between SoD and Dual Control?**
4. **Why should a developer not deploy their own code to production?**
5. **How do small organizations implement SoD with limited staff?**

---

[← Previous: Need to Know](02-need-to-know.md) | [Next: Defense in Depth →](04-defense-in-depth.md)

---

*Unit 3 - Topic 3 of 6 | [Back to README](../README.md)*
