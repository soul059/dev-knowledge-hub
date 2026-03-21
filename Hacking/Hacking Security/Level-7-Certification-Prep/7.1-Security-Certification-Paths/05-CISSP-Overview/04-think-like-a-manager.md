# Unit 5: CISSP Overview — Topic 4: Think Like a Manager

## Overview

The single most important concept for passing CISSP is learning to **"think like a manager."** Unlike technical certifications, CISSP questions require you to choose answers from a managerial, risk-based perspective rather than a purely technical one. This topic explores this critical mindset shift.

---

## 1. The Manager's Mindset

```
TECHNICAL vs MANAGERIAL THINKING:

TECHNICAL MINDSET (WRONG for CISSP):
  → "What tool should I use?"
  → "How do I configure this?"
  → "What protocol is best?"
  → "What's the technical fix?"
  → Focus on implementation details

MANAGERIAL MINDSET (RIGHT for CISSP):
  → "What policy addresses this?"
  → "What process should be followed?"
  → "What risk does this represent?"
  → "What's the business impact?"
  → Focus on governance and oversight

EXAMPLE QUESTION:
  "A new vulnerability is discovered in your
   web application. What should you do FIRST?"

  A. Patch the vulnerability immediately
  B. Scan all systems for the vulnerability
  C. Assess the risk to the organization
  D. Notify management of the vulnerability

  Technical answer: A (patch it!)
  CISSP answer: C (assess risk first)

  WHY? A manager assesses risk before acting.
  Not all vulnerabilities require immediate patching.
  The impact and likelihood must be evaluated.

KEY PRINCIPLE:
  → CISSP is about managing security, not doing it
  → You advise, recommend, and oversee
  → Think: "What would a CISO do?"
```

---

## 2. Decision-Making Framework

```
CISSP ANSWER SELECTION HIERARCHY:

PRIORITY ORDER FOR ANSWERS:

  1. SAFETY OF HUMAN LIFE
     → Always the top priority
     → Overrides all other considerations
     → If lives are at risk, protect people first

  2. PREVENT FURTHER DAMAGE
     → Contain the incident
     → Stop the bleeding
     → Isolate affected systems

  3. PRESERVE EVIDENCE
     → Chain of custody
     → Forensic procedures
     → Don't destroy evidence

  4. RESTORE OPERATIONS
     → Business continuity
     → Disaster recovery
     → Return to normal

  5. REMEDIATE
     → Fix the root cause
     → Implement controls
     → Prevent recurrence

WHEN TWO ANSWERS SEEM RIGHT:
  → Which is more ADMINISTRATIVE?
  → Which addresses POLICY?
  → Which involves RISK ASSESSMENT?
  → Which protects the BUSINESS?
  → Which is done FIRST in the process?
  → Administrative > Technical > Physical

PROCESS-BASED THINKING:
  Always follow the process:
  → Plan → Do → Check → Act
  → Identify → Assess → Mitigate → Monitor
  → Detect → Respond → Recover → Learn
```

---

## 3. Common Question Patterns

```
CISSP QUESTION PATTERNS:

"WHAT IS THE FIRST THING...":
  → Almost always: Assess, identify, or plan
  → NOT: Implement, configure, or execute
  → Process step 1 is always assessment
  → Example: "First" = scope, identify, assess

"BEST" ANSWER QUESTIONS:
  → Multiple answers may be correct
  → Choose the MOST appropriate
  → Consider context and constraints
  → Business-oriented > technical
  → Preventive > detective > corrective

"MOST IMPORTANT" QUESTIONS:
  → Safety of life
  → Protecting the organization
  → Compliance with regulations
  → Following established policy
  → Risk-based decision making

SCENARIO-BASED PATTERNS:
  → New employee: Background check, training
  → Data breach: Contain, assess, notify
  → Disaster: BCP activation, safety first
  → Vendor: Due diligence, contract review
  → Compliance: Audit, assess, remediate
  → Risk: Assess, treat, accept residual

ROLE-BASED ANSWERS:
  → Data Owner: Senior management, classifies data
  → Data Custodian: IT, implements controls
  → Data Steward: Quality and compliance
  → Security Officer: Policies and oversight
  → Know WHO is responsible for WHAT
```

---

## 4. Practice Scenarios

```
PRACTICE THINKING LIKE A MANAGER:

SCENARIO 1: Critical Patch
  "A critical vulnerability is announced for your
   production servers. What do you do FIRST?"
  
  A. Apply the patch immediately
  B. Test the patch in a staging environment
  C. Assess the risk and impact of the vulnerability
  D. Notify the change management board
  
  Answer: C — Assess risk first, then follow
  change management process for patching.

SCENARIO 2: Security Incident
  "You discover unauthorized access to sensitive
   customer data. What is your FIRST action?"
  
  A. Disconnect the affected server
  B. Contact law enforcement
  C. Follow the incident response plan
  D. Notify affected customers
  
  Answer: C — Always follow established procedures.
  The IR plan will guide subsequent actions.

SCENARIO 3: New Application
  "A business unit wants to deploy a new cloud
   application. What is MOST important?"
  
  A. Conduct a security assessment
  B. Ensure it meets compliance requirements
  C. Perform a risk assessment
  D. Review the vendor's security practices
  
  Answer: C — Risk assessment encompasses all
  other concerns and drives decisions.

SCENARIO 4: Budget Decision
  "You can only fund one security initiative.
   Which provides the MOST value?"
  
  A. Deploy new SIEM solution
  B. Implement security awareness training
  C. Hire additional SOC analysts
  D. Address the highest-risk vulnerabilities
  
  Answer: D — Risk-based approach. Address
  what poses the greatest risk to the business.
```

---

## 5. Ethics and Professional Conduct

```
(ISC)² CODE OF ETHICS:

FOUR CANONS (MUST MEMORIZE ORDER):
  1. Protect society, the common good, necessary 
     public trust and confidence, and the 
     infrastructure
  2. Act honorably, honestly, justly, responsibly,
     and legally
  3. Provide diligent and competent service to 
     principals
  4. Advance and protect the profession

PRIORITY ORDER:
  Society > Yourself > Employer > Profession

ETHICS EXAM QUESTIONS:
  → You discover your employer is breaking the law
     → Report it (Canon 1 > Canon 3)
  → A colleague is cheating on a certification
     → Report to (ISC)² (Canon 4)
  → You find a vulnerability in a client system
     → Report to client, not public (Canon 2)
  → A regulation conflicts with employer policy
     → Follow the regulation (Canon 1)

KEY ETHICS CONCEPTS:
  → Due diligence: Research before acting
  → Due care: Act responsibly with knowledge
  → Negligence: Failure to exercise due care
  → Conflict of interest: Disclose and recuse
  → Whistleblowing: Report illegal activity
  → Professional responsibility: Maintain competence

EXAM TIP: If a question involves ethics,
the answer that protects the PUBLIC and
SOCIETY is almost always correct, even if
it conflicts with employer wishes.
```

---

## Summary Table

| Concept | Key Principle | Application |
|---------|-------------|------------|
| Manager Mindset | Govern, don't implement | Choose policy over tools |
| Priority Order | Life > damage > evidence > ops | Use in every question |
| First Action | Always assess/identify | Never jump to fixing |
| Best Answer | Administrative > technical | Choose governance |
| Ethics | Society > employer | Public good first |

---

## Revision Questions

1. How does "thinking like a manager" change your answer approach on the CISSP?
2. What is the priority order when responding to a security incident?
3. In what order should the (ISC)² Code of Ethics canons be applied?
4. Why is "assess the risk" often the correct first answer?
5. How do you choose between two seemingly correct CISSP answers?

---

*Previous: [03-study-strategies.md](03-study-strategies.md) | Next: [05-common-pitfalls.md](05-common-pitfalls.md)*

---

*[Back to README](../README.md)*
