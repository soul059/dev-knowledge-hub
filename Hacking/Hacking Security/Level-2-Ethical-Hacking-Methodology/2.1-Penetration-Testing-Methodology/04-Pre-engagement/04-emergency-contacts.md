# Emergency Contacts

## Unit 4 - Topic 4 | Pre-engagement

---

## Overview

**Emergency contacts** are critical for maintaining safety during penetration testing. When testing goes wrong — systems crash, services go down, or real attackers are discovered — knowing **who to call immediately** can prevent significant damage. Every engagement must have a documented emergency contact list.

---

## 1. Emergency Contact Matrix

| Role | Who | When to Contact |
|------|-----|----------------|
| **Primary POC** | Client IT security lead | Daily updates, questions |
| **Secondary POC** | Backup contact | When primary unavailable |
| **Emergency Technical** | Senior sysadmin/engineer | System crash, outage |
| **Emergency Management** | CISO / IT Director | Critical incident |
| **Legal** | Client legal counsel | Legal issues arise |
| **Testing Lead** | Pen test team lead | Internal team issues |
| **SOC/NOC** | Security/Network Ops Center | If testing triggers alerts |

---

## 2. Emergency Contact Card

```
EMERGENCY CONTACT CARD
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PROJECT: [Project Name]
CLIENT: [Client Name]
DATES: [Start] - [End]

PRIMARY CONTACT:
Name: ____________________
Role: ____________________
Phone: ___________________
Email: ___________________
Available: _______________

SECONDARY CONTACT:
Name: ____________________
Phone: ___________________
Email: ___________________

EMERGENCY TECHNICAL:
Name: ____________________
Phone: ___________________
Available: 24/7

MANAGEMENT ESCALATION:
Name: ____________________
Phone: ___________________

TESTING TEAM LEAD:
Name: ____________________
Phone: ___________________

PROJECT CODE WORD: ____________
(Used to identify authorized testers)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 3. When to Use Emergency Contacts

```
CONTACT IMMEDIATELY IF:

🔴 CRITICAL (Call within minutes):
├── System crash or service outage
├── Data corruption detected
├── Real security breach discovered during testing
├── Testing causes business impact
├── Law enforcement contact
└── Injury during physical testing

🟡 URGENT (Call within 1 hour):
├── Critical vulnerability found
├── Suspected active attacker detected
├── Scope boundary accidentally crossed
├── Credentials for sensitive systems found
└── Unexpected system behavior

🟢 ROUTINE (Email/scheduled update):
├── Daily progress update
├── Questions about scope
├── Request for additional access
├── Minor findings for awareness
└── Schedule adjustments
```

---

## 4. Code Words and Identification

```
CODE WORD SYSTEM:
Used to quickly identify authorized testers

Example:
├── Code word: "GAMMA-SEVEN"
├── Tester calls SOC: "This is [Name] from [Firm], 
│   code word GAMMA-SEVEN, conducting authorized 
│   penetration testing per project [ID]"
├── SOC verifies code word against master list
└── SOC confirms: "Verified. How can I assist?"

BENEFITS:
├── Quick verification without lengthy explanations
├── Prevents social engineering of the SOC
├── Works for both phone and in-person scenarios
└── Can be changed if compromised
```

---

## 5. Escalation Procedures

```
ESCALATION PATH:

Level 1: Primary POC
├── Minor issues, questions, daily updates
├── Response time: Same business day
│
Level 2: Secondary POC
├── When primary is unavailable
├── Urgent technical questions
├── Response time: Within 2 hours
│
Level 3: Emergency Technical
├── System outage, data loss
├── Response time: Within 30 minutes
│
Level 4: Management (CISO/Director)
├── Critical business impact
├── Legal concerns
├── Engagement-stopping issues
├── Response time: Within 15 minutes
│
Level 5: Executive / Legal
├── Legal issues, law enforcement
├── Breach of third-party systems
├── Response time: Immediate
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Emergency contacts** | Documented before testing begins |
| **Code word** | Quick identification of authorized testers |
| **Escalation path** | Clear levels from POC to executive |
| **24/7 availability** | Emergency technical contact always reachable |
| **Critical contact** | Within minutes for outages/breaches |

---

## Quick Revision Questions

1. **Why are emergency contacts needed for pen testing?**
2. **What warrants an immediate emergency call?**
3. **What is the purpose of a code word system?**
4. **How many levels of escalation should be defined?**
5. **When should management be contacted?**

---

[← Previous: Authorization Forms](03-authorization-forms.md) | [Next: Communication Protocols →](05-communication-protocols.md)

---

*Unit 4 - Topic 4 of 6 | [Back to README](../README.md)*
