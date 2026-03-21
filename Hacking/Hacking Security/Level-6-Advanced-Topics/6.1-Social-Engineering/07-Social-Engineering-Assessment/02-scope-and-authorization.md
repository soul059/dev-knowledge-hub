# Unit 7: Social Engineering Assessment — Topic 2: Scope and Authorization

## Overview

**Scope and authorization** define the legal and operational boundaries of a social engineering assessment. Proper scoping ensures the assessment tests what matters, while authorization protects the tester from legal consequences. Without clear, written authorization, social engineering testing is indistinguishable from criminal activity.

---

## 1. Defining Scope

```
SCOPE DIMENSIONS:

WHO IS IN SCOPE:
  → All employees vs specific departments
  → Contractors and temporary staff?
  → Executive/VIP exclusions
  → Specific individuals (by name or role)
  → Remote workers included?
  → International locations included?
  → Third-party vendors/partners?

WHAT VECTORS ARE IN SCOPE:
  → Email phishing (mass, spear, whaling)
  → Vishing (phone attacks)
  → Smishing (SMS attacks)
  → Physical access attempts
  → USB/media drops
  → Social media attacks
  → In-person social engineering
  → Combination attacks

WHAT LOCATIONS ARE IN SCOPE:
  → Headquarters
  → Branch offices
  → Data centers
  → Remote locations
  → Public areas (conferences, etc.)
  → Parking lots and perimeter
  → Loading docks and service entrances

WHAT SYSTEMS/DATA:
  → Email systems
  → Phone systems
  → Physical access controls
  → Badge/access card systems
  → Specific applications
  → What data can be accessed if successful
  → What data can be exfiltrated (if any)

WHAT IS OUT OF SCOPE:
  → Specific exclusions documented
  → Protected individuals (CEO's EA, etc.)
  → Certain locations (government facilities)
  → Destructive actions
  → Actual data exfiltration vs proof of access
  → Actions that could harm individuals
  → Production system disruption
```

---

## 2. Authorization Documentation

```
ESSENTIAL DOCUMENTS:

1. STATEMENT OF WORK (SOW):
   → Detailed scope description
   → Deliverables and timeline
   → Pricing and payment terms
   → Roles and responsibilities
   → Acceptance criteria

2. RULES OF ENGAGEMENT (ROE):
   → Approved attack vectors
   → Prohibited actions list
   → Escalation procedures
   → Communication protocols
   → Emergency contacts
   → Hours of testing
   → Data handling requirements

3. AUTHORIZATION LETTER:
   → "Get out of jail free" card
   → Signed by authorized executive
   → States tester is authorized to perform tests
   → Specifies scope and timeframe
   → Tester carries during physical testing
   → Shows to law enforcement if confronted

   SAMPLE LANGUAGE:
   "This letter authorizes [Tester Name] and 
   [Company] to perform social engineering 
   testing activities at [Location] between 
   [Start Date] and [End Date] as part of an 
   authorized security assessment. If you have 
   questions, contact [Authorized Person] at 
   [Phone Number]."

4. NON-DISCLOSURE AGREEMENT (NDA):
   → Protects client's sensitive information
   → Covers test results and findings
   → Covers any data accessed during testing
   → Duration of confidentiality obligation

5. LIABILITY/INSURANCE:
   → Professional liability insurance
   → Errors and omissions (E&O) coverage
   → General liability
   → Cyber liability insurance
   → Indemnification clauses

6. EMERGENCY PROCEDURES:
   → Abort protocol (when to stop)
   → Emergency contacts (24/7)
   → Incident response coordination
   → Law enforcement encounter procedure
   → Medical emergency procedures
```

---

## 3. Legal Considerations

```
LEGAL FRAMEWORK:

FEDERAL LAWS (US):
  → Computer Fraud and Abuse Act (CFAA)
    - Unauthorized access to computers
    - Written authorization is defense
  → Wire fraud statutes
    - Email/phone deception considerations
  → Privacy laws
    - Recording calls (one-party/two-party consent)
    - Email monitoring regulations
  → Identity theft statutes
    - Creating fake identities

STATE LAWS:
  → Vary significantly by state
  → Trespassing laws
  → Wiretapping/recording laws
  → Impersonation laws
  → Data breach notification
  → Privacy regulations

INTERNATIONAL:
  → GDPR (EU) — employee data handling
  → Country-specific privacy laws
  → Cross-border data transfer
  → Local impersonation laws
  → Recording and surveillance laws

KEY LEGAL PRINCIPLES:
  → Authorization must come from someone 
    with AUTHORITY to grant it
  → Authorization must be SPECIFIC to activities
  → Authorization must be WRITTEN
  → Authorization must be CURRENT (dated)
  → Tester must STAY WITHIN scope
  → Document everything for legal protection
```

---

## 4. Scope Creep Prevention

```
MANAGING SCOPE:

CLEAR BOUNDARIES:
  → Explicitly document what's included
  → Explicitly document what's excluded
  → If it's not in scope, DON'T do it
  → Any changes require written approval

CHANGE MANAGEMENT:
  → Scope changes must be authorized
  → Document reason for change
  → Get written approval before proceeding
  → Update SOW and ROE accordingly
  → Communicate changes to all stakeholders

COMMON SCOPE CREEP SCENARIOS:
  → "While you're here, test the other floor too"
    → Requires new authorization
  → "Can you also try to get into the CEO's email?"
    → Requires explicit scope addition
  → Discovering open door → entering unauthorized area
    → Document and report, don't enter
  → Finding credentials → accessing production systems
    → Depends on scope — verify before acting

RED LINES (Never Cross):
  → Never access systems/areas not in scope
  → Never test excluded individuals
  → Never cause actual harm
  → Never continue after abort signal
  → Never retain client data beyond agreement
  → Never share findings with unauthorized parties
```

---

## 5. Authorization Verification

```
VERIFICATION CHECKLIST:

BEFORE STARTING:
  □ SOW signed by all parties
  □ ROE documented and acknowledged
  □ Authorization letter in hand
  □ NDA executed
  □ Insurance current and adequate
  □ Emergency contacts verified
  □ Legal review completed
  □ Scope clearly understood by all team members

VERIFICATION OF AUTHORIZER:
  → Does this person have authority to authorize?
  → CEO/CISO: Yes
  → IT Manager: Maybe (depends on scope)
  → Department head: For their department only
  → Third party: Only with owner's written consent
  → Landlord vs tenant distinction

MULTI-TENANT CONSIDERATIONS:
  → Building shared with other companies
  → Only authorized for specific tenant space
  → Common areas may need building management approval
  → Don't access other tenant's space
  → Parking lot may be shared property
```

---

## Summary Table

| Document | Purpose | Who Signs |
|----------|---------|-----------|
| SOW | Defines scope and deliverables | Both parties |
| ROE | Sets rules and boundaries | Both parties |
| Authorization letter | Legal protection | Client executive |
| NDA | Confidentiality | Both parties |
| Insurance | Liability coverage | Tester |
| Emergency procedures | Safety protocols | Both parties |

---

## Revision Questions

1. What are the key dimensions of social engineering assessment scope?
2. Why is written authorization essential and what should it contain?
3. What legal frameworks govern social engineering testing?
4. How do you prevent and manage scope creep during assessments?
5. What must be verified about the person providing authorization?

---

*Previous: [01-engagement-planning.md](01-engagement-planning.md) | Next: [03-campaign-execution.md](03-campaign-execution.md)*

---

*[Back to README](../README.md)*
