# Unit 4: A04 - Insecure Design — Topic 2: Threat Modeling

## Overview

Threat modeling is a **structured process** for identifying security threats, vulnerabilities, and countermeasures during the design phase — before writing code. It answers four fundamental questions: *What are we building? What can go wrong? What are we going to do about it? Did we do a good job?* Effective threat modeling is the single most impactful activity for preventing insecure design.

---

## 1. Why Threat Model?

```
┌─────────────────────────────────────────────────────────────────┐
│               COST OF FIXING SECURITY ISSUES                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Cost                                                          │
│  ▲                                                             │
│  │                                               ████ $30,000  │
│  │                                    ████                      │
│  │                         ████       $7,600                    │
│  │              ████       $2,500                               │
│  │   ████       $750                                           │
│  │   $150                                                      │
│  └──────────────────────────────────────────────────── Phase    │
│     Design   Coding   Testing   Release   Post-Release         │
│                                                                 │
│  Finding issues in DESIGN is 30-200x cheaper than in production│
│  Threat modeling catches design issues before code exists       │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Threat Modeling Methodologies

### STRIDE

```
┌────────────┬──────────────────────────────────────────────────────┐
│ Threat     │ Description                                         │
├────────────┼──────────────────────────────────────────────────────┤
│ Spoofing   │ Pretending to be someone/something else             │
│            │ Example: Forging authentication tokens               │
│            │ Control: Strong authentication, MFA                  │
├────────────┼──────────────────────────────────────────────────────┤
│ Tampering  │ Modifying data or code                              │
│            │ Example: Changing price in API request               │
│            │ Control: Integrity checks, digital signatures        │
├────────────┼──────────────────────────────────────────────────────┤
│ Repudiation│ Denying an action was performed                     │
│            │ Example: User denies making a transaction            │
│            │ Control: Audit logging, digital signatures            │
├────────────┼──────────────────────────────────────────────────────┤
│ Info       │ Unauthorized access to information                  │
│ Disclosure │ Example: Error messages revealing DB structure       │
│            │ Control: Encryption, access controls, data masking   │
├────────────┼──────────────────────────────────────────────────────┤
│ Denial of  │ Making service unavailable                          │
│ Service    │ Example: Resource exhaustion attack                  │
│            │ Control: Rate limiting, auto-scaling, CDN            │
├────────────┼──────────────────────────────────────────────────────┤
│ Elevation  │ Gaining unauthorized privileges                     │
│ of Priv    │ Example: Regular user accessing admin functions     │
│            │ Control: RBAC, least privilege, input validation     │
└────────────┴──────────────────────────────────────────────────────┘
```

### PASTA (Process for Attack Simulation and Threat Analysis)

```
7-Stage Process:

Stage 1: Define Business Objectives
  → What is the application's purpose?
  → What data does it handle?
  → Compliance requirements?

Stage 2: Define Technical Scope
  → Architecture diagrams
  → Technology stack
  → Data flows

Stage 3: Application Decomposition
  → Identify components, entry points, trust boundaries
  → Data flow diagrams

Stage 4: Threat Analysis
  → Research threat landscape
  → Identify threat actors and their capabilities
  → Review CVEs and known attack patterns

Stage 5: Vulnerability Analysis
  → Map weaknesses to components
  → Review existing security controls
  → Identify gaps

Stage 6: Attack Modeling
  → Build attack trees
  → Simulate attack scenarios
  → Estimate likelihood and impact

Stage 7: Risk and Impact Analysis
  → Prioritize risks
  → Define countermeasures
  → Map to business impact
```

### DREAD (Risk Rating)

| Factor | Question | Scale |
|--------|----------|-------|
| **D**amage | How bad is an exploit? | 1-10 |
| **R**eproducibility | How easy to reproduce? | 1-10 |
| **E**xploitability | How easy to attack? | 1-10 |
| **A**ffected Users | How many users impacted? | 1-10 |
| **D**iscoverability | How easy to find? | 1-10 |

```
Risk Score = (D + R + E + A + D) / 5

Example: SQL Injection in login form
  Damage:          9 (full DB access)
  Reproducibility: 8 (easy to reproduce)
  Exploitability:  9 (tools like SQLMap)
  Affected Users: 10 (all users)
  Discoverability: 7 (scanner can find)
  Score: (9+8+9+10+7)/5 = 8.6 → Critical
```

---

## 3. Data Flow Diagrams (DFDs)

```
DFD Elements:
┌──────────────┐
│  ══════════  │  External Entity (user, external system)
│  [Process]   │  Process (transforms data)  
│  (Data Store)│  Data Store (database, file)
│  ──────────→ │  Data Flow (arrow)
│  ┈┈┈┈┈┈┈┈┈┈ │  Trust Boundary (dashed line)
└──────────────┘

Example — E-Commerce Application DFD:

                        Trust Boundary
┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┈ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐
│                         ┈                                       │
│  ══════════             ┈     [Web Server]                      │
│   Browser  ──HTTPS──→   ┈   ──→ [API Server] ──→ (Database)   │
│  ══════════             ┈        │                              │
│                         ┈        ├──→ (File Storage)           │
│  ══════════             ┈        │                              │
│  Mobile App ──HTTPS──→  ┈        └──→ ══════════               │
│  ══════════             ┈             Payment                  │
│                         ┈             Gateway                  │
│                         ┈             ══════════               │
│  UNTRUSTED              ┈     TRUSTED                          │
└ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┈ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘

Every data flow crossing a trust boundary is an attack surface.
Apply STRIDE to each crossing point.
```

---

## 4. Practical Threat Modeling Exercise

```
Feature: "Password Reset via Email"

Step 1: Draw the flow
  User → [Submit Email] → [Generate Token] → [Send Email] → [Click Link] → [Reset Password]

Step 2: Apply STRIDE

  SPOOFING:
  - Can attacker trigger reset for another user? → YES
  - Mitigation: Don't confirm if email exists; rate limit

  TAMPERING:
  - Can attacker modify the reset token? → Possible
  - Mitigation: Cryptographically signed tokens, server-side validation

  REPUDIATION:
  - Can user deny resetting their password? → Yes
  - Mitigation: Log all reset events with IP, timestamp

  INFORMATION DISCLOSURE:
  - Does "email not found" reveal user existence? → YES
  - Mitigation: Generic "If account exists, email sent" message

  DENIAL OF SERVICE:
  - Can attacker flood reset requests? → YES
  - Mitigation: Rate limit per IP and per email, CAPTCHA

  ELEVATION OF PRIVILEGE:
  - Can attacker reset admin password? → YES if no extra verification
  - Mitigation: Admin accounts require additional verification (MFA)

Step 3: Document mitigations as security requirements
Step 4: Verify mitigations are implemented in code
```

---

## 5. Threat Modeling Tools

| Tool | Type | Features |
|------|------|----------|
| **Microsoft Threat Modeling Tool** | Desktop app | STRIDE auto-generation, DFD editor, report generation |
| **OWASP Threat Dragon** | Web/Desktop | Open source, DFD editor, STRIDE/LINDDUN |
| **IriusRisk** | Commercial | Automated threat modeling, integrates with Jira |
| **Threagile** | CLI/Code | YAML-based, infrastructure-as-code |
| **pytm** | Python library | Threat model as code, generates DFDs |

### Threat Model as Code (pytm)

```python
from pytm import TM, Server, Dataflow, Boundary, Actor, Datastore

tm = TM("E-Commerce App")
tm.description = "Online shopping platform"

# Define boundaries
internet = Boundary("Internet")
dmz = Boundary("DMZ")  
internal = Boundary("Internal Network")

# Define components
user = Actor("Customer")
user.inBoundary = internet

web = Server("Web Server")
web.inBoundary = dmz

api = Server("API Server")
api.inBoundary = internal

db = Datastore("Database")
db.inBoundary = internal

# Define data flows
Dataflow(user, web, "HTTPS Request")
Dataflow(web, api, "API Call")
Dataflow(api, db, "SQL Query")

# Generate threats automatically
tm.process()
```

---

## 6. When to Threat Model

```
Threat Model at Every Stage:

1. NEW FEATURE DESIGN
   → Before writing any code
   → Part of design review

2. ARCHITECTURE CHANGES
   → New services, new data stores
   → Changed trust boundaries

3. INTEGRATION POINTS
   → New third-party APIs
   → New authentication methods

4. INCIDENT RESPONSE
   → After a security breach
   → Update threat model with real attack data

5. PERIODIC REVIEW
   → At least annually
   → When threat landscape changes
```

---

## Summary Table

| Methodology | Focus | Best For |
|-------------|-------|----------|
| **STRIDE** | Threat categories | Per-component analysis |
| **PASTA** | Business risk | Enterprise applications |
| **DREAD** | Risk scoring | Prioritizing threats |
| **Attack Trees** | Attack paths | Complex scenarios |
| **LINDDUN** | Privacy threats | GDPR/privacy compliance |

---

## Revision Questions

1. What are the four fundamental questions of threat modeling?
2. Explain each letter in STRIDE with real-world examples.
3. Draw a Data Flow Diagram for a simple web application with authentication. Identify trust boundaries.
4. Perform a STRIDE analysis on a "file upload" feature.
5. Compare STRIDE and PASTA methodologies — when would you use each?
6. Why should threat modeling happen before code is written, not after?

---

*Previous: [01-secure-design-principles.md](01-secure-design-principles.md) | Next: [03-secure-design-patterns.md](03-secure-design-patterns.md)*

---

*[Back to README](../README.md)*
