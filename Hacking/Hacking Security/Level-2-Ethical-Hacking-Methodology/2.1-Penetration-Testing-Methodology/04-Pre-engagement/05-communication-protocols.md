# Communication Protocols

## Unit 4 - Topic 5 | Pre-engagement

---

## Overview

**Communication protocols** define how information flows between the testing team and the client throughout the engagement. Secure, structured communication prevents **data leaks**, ensures **timely updates**, and maintains **professional standards**.

---

## 1. Communication Channels

| Channel | Use Case | Security |
|---------|----------|:--------:|
| **Encrypted email (PGP/S-MIME)** | Reports, findings, documentation | ✅ High |
| **Signal / Wire** | Urgent messages, quick questions | ✅ High |
| **Phone call** | Emergency contacts, critical findings | ⚠️ Medium |
| **Encrypted file sharing** | Report delivery, evidence | ✅ High |
| **Client portal** | Status tracking, deliverables | ✅ High |
| **Regular email** | ❌ NOT for sensitive data | ❌ Low |
| **Slack / Teams** | ❌ Only if client-approved | ⚠️ Variable |

---

## 2. Communication Schedule

```
TYPICAL COMMUNICATION PLAN:

DAILY:
├── Brief status email (encrypted)
├── Hours worked, activities performed
├── Any blockers or questions
└── Planned activities for next day

WEEKLY:
├── Detailed progress report
├── Significant findings summary
├── Scope completion percentage
└── Risk concerns or adjustments

AD-HOC:
├── Critical vulnerability: Within 1 hour (phone)
├── System impact: Immediate (phone)
├── Scope questions: Same business day (email)
└── Access issues: Within 2 hours (email/phone)

END OF ENGAGEMENT:
├── Draft report delivery
├── Report review period (5-10 business days)
├── Final report delivery
├── Debrief presentation
└── Retesting (if included)
```

---

## 3. Status Update Template

```
DAILY STATUS UPDATE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Project: [Name]
Date: [Date]
Tester: [Name]

ACTIVITIES COMPLETED:
• Completed port scanning of external range
• Identified 3 web applications
• Started vulnerability assessment of web apps

KEY FINDINGS (High-level):
• [Critical] RCE vulnerability in web server
• [High] Default credentials on admin portal

BLOCKERS:
• VPN disconnects frequently (need stable connection)

PLANNED FOR TOMORROW:
• Continue web app testing
• Test API endpoints
• Begin internal network scan

HOURS: 8 hours (running total: 24/80)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 4. Secure Communication Best Practices

```
DATA SECURITY IN COMMUNICATION:

DO:
✅ Encrypt all emails containing findings
✅ Use PGP/GPG for email encryption
✅ Use secure file sharing (encrypted, access-controlled)
✅ Use Signal/Wire for urgent messaging
✅ Verify recipient before sending sensitive data
✅ Use code words for phone conversations

DON'T:
❌ Send vulnerability details in plain text email
❌ Discuss findings over unencrypted channels
❌ Share credentials in email/chat
❌ Post findings on social media
❌ Discuss client details with unauthorized parties
❌ Use personal email for business communication
```

---

## 5. Debrief and Report Delivery

```
END-OF-ENGAGEMENT COMMUNICATION:

1. DRAFT REPORT DELIVERY
   ├── Encrypted file transfer
   ├── Password sent via separate channel
   └── Allow 5-10 business days for review

2. REPORT REVIEW MEETING
   ├── Walk through findings with technical team
   ├── Address questions and clarifications
   ├── Correct any factual errors
   └── Finalize risk ratings

3. EXECUTIVE DEBRIEF
   ├── High-level presentation for management
   ├── Focus on business impact
   ├── Strategic recommendations
   └── Answer executive questions

4. FINAL REPORT DELIVERY
   ├── Incorporate feedback
   ├── Deliver via encrypted channel
   └── Confirm receipt
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Encryption** | All findings must be encrypted in transit |
| **Daily updates** | Brief status to keep client informed |
| **Critical findings** | Phone call within 1 hour |
| **Secure channels** | PGP email, Signal, encrypted file sharing |
| **Debrief** | Present findings to both technical and executive teams |

---

## Quick Revision Questions

1. **Why must penetration test communications be encrypted?**
2. **How quickly should a critical finding be communicated?**
3. **What should a daily status update include?**
4. **Name 3 secure communication channels.**
5. **What is an executive debrief?**

---

[← Previous: Emergency Contacts](04-emergency-contacts.md) | [Next: Environment Preparation →](06-environment-preparation.md)

---

*Unit 4 - Topic 5 of 6 | [Back to README](../README.md)*
