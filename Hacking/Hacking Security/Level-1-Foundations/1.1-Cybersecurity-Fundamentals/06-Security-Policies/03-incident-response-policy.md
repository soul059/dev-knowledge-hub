# Incident Response Policy

## Unit 6 - Topic 3 | Security Policies

---

## Overview

An **Incident Response (IR) Policy** defines how an organization prepares for, detects, responds to, and recovers from cybersecurity incidents. It ensures a structured, repeatable approach to handling incidents — minimizing damage, recovery time, and costs.

---

## 1. Incident Response Lifecycle (NIST SP 800-61)

```
┌──────────────────────────────────────────────────────────────────┐
│              INCIDENT RESPONSE LIFECYCLE                         │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────────────┐ │
│  │ 1.PREPARATION │──►│ 2.DETECTION  │──►│ 3.CONTAINMENT,       │ │
│  │               │   │ & ANALYSIS   │   │   ERADICATION &      │ │
│  │ Plan, train,  │   │              │   │   RECOVERY           │ │
│  │ equip         │   │ Identify     │   │                      │ │
│  └──────────────┘   │ & assess     │   │ Stop, remove,        │ │
│         ▲            └──────────────┘   │ restore              │ │
│         │                               └──────────┬───────────┘ │
│         │                                          │             │
│         │            ┌──────────────────────────────┘             │
│         │            ▼                                           │
│         │     ┌──────────────┐                                   │
│         └─────│ 4.POST-      │                                   │
│               │   INCIDENT   │   Lessons learned,                │
│               │   ACTIVITY   │   improve process                 │
│               └──────────────┘                                   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Incident Classification

| Severity | Description | Response Time | Example |
|----------|-------------|--------------|---------|
| **Critical (P1)** | Active data breach, ransomware spreading | Immediate (15 min) | Ransomware encrypting production servers |
| **High (P2)** | Confirmed compromise, significant impact | 1 hour | Compromised admin account |
| **Medium (P3)** | Potential compromise, limited impact | 4 hours | Phishing email with malware clicked |
| **Low (P4)** | Minor incident, no data loss | 24 hours | Failed brute force attempt |
| **Informational** | No impact, awareness only | Next business day | Vulnerability scan finding |

---

## 3. Key IR Policy Components

| Component | Description |
|-----------|-------------|
| **Scope** | What constitutes a "security incident" |
| **Roles & Responsibilities** | IR team, management, legal, communications |
| **Classification Scheme** | How to categorize incident severity |
| **Escalation Procedures** | When and how to escalate to higher levels |
| **Communication Plan** | Internal notifications, external reporting, media |
| **Evidence Handling** | Preserving evidence for forensics/legal proceedings |
| **Reporting Requirements** | Regulatory notifications (72hr GDPR, HIPAA, etc.) |
| **Recovery Procedures** | Steps to restore normal operations |
| **Post-Incident Review** | Lessons learned and process improvement |

---

## 4. IR Team Roles

| Role | Responsibility |
|------|---------------|
| **IR Manager** | Leads the response effort, makes key decisions |
| **Security Analysts** | Investigate, contain, and eradicate threats |
| **IT Operations** | System restoration and recovery |
| **Legal Counsel** | Advise on legal obligations and liability |
| **Communications/PR** | Manage external communications and media |
| **HR** | Handle insider threat situations, employee issues |
| **Executive Sponsor** | Authorize resources and high-level decisions |

---

## 5. Regulatory Notification Requirements

| Regulation | Notification Requirement |
|-----------|------------------------|
| **GDPR** | 72 hours to supervisory authority |
| **HIPAA** | 60 days to HHS; notify affected individuals |
| **PCI-DSS** | Immediately to card brands and acquirer |
| **State Breach Laws** | Varies by state (US) — typically 30-60 days |
| **NIS2 (EU)** | 24 hours initial, 72 hours full notification |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **IR Lifecycle** | Preparation → Detection → Containment/Eradication/Recovery → Post-Incident |
| **Classification** | Critical (P1) through Informational |
| **Key Teams** | Security, IT Ops, Legal, Communications, HR, Executives |
| **Notifications** | GDPR=72hrs, HIPAA=60 days, PCI=Immediate |
| **Post-Incident** | Lessons learned are ESSENTIAL for improvement |
| **Documentation** | Document EVERYTHING during an incident |

---

## Quick Revision Questions

1. **What are the four phases of the NIST IR lifecycle?**
2. **How would you classify a ransomware attack on production systems?**
3. **What roles make up an incident response team?**
4. **What is the GDPR notification requirement for a data breach?**
5. **Why is the post-incident review phase important?**

---

[← Previous: Acceptable Use Policy](02-acceptable-use-policy.md) | [Next: Password Policy →](04-password-policy.md)

---

*Unit 6 - Topic 3 of 6 | [Back to README](../README.md)*
