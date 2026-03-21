# Unit 1: Security Operations Fundamentals — Topic 2: SOC Models

## Overview

Organizations can implement different **SOC models** based on their size, budget, industry, and security maturity. Each model offers distinct advantages in terms of cost, control, expertise, and coverage. Understanding these models helps organizations select the right approach for their specific needs and constraints.

---

## 1. SOC Model Types

```
SOC DEPLOYMENT MODELS:

  ┌──────────────────────────────────────────────────────┐
  │                    SOC MODELS                         │
  │                                                       │
  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌─────────┐ │
  │  │ IN-HOUSE │ │ OUTSOURCE│ │  HYBRID  │ │ VIRTUAL │ │
  │  │   SOC    │ │  (MSSP)  │ │   SOC    │ │   SOC   │ │
  │  │          │ │          │ │          │ │         │ │
  │  │ Full     │ │ Managed  │ │ Mixed    │ │ Remote  │ │
  │  │ internal │ │ security │ │ in+out   │ │ staff   │ │
  │  │ control  │ │ provider │ │ sourced  │ │ no loc  │ │
  │  └──────────┘ └──────────┘ └──────────┘ └─────────┘ │
  │                                                       │
  │  ┌──────────┐ ┌──────────┐ ┌──────────┐             │
  │  │  CO-MAN  │ │   MDR    │ │ COMMAND  │             │
  │  │   SOC    │ │          │ │  CENTER  │             │
  │  │          │ │ Managed  │ │          │             │
  │  │ Shared   │ │ detect & │ │ Multi-   │             │
  │  │ manage   │ │ respond  │ │ site     │             │
  │  └──────────┘ └──────────┘ └──────────┘             │
  └──────────────────────────────────────────────────────┘
```

---

## 2. In-House SOC

```
IN-HOUSE (INTERNAL) SOC:

  Description:
  → Fully staffed and managed by the organization
  → Dedicated facility and infrastructure
  → Complete control over operations
  → Organization owns all tools and data

  ADVANTAGES:
  ✓ Full control over security operations
  ✓ Deep knowledge of the environment
  ✓ Customized processes and tools
  ✓ Direct access to systems
  ✓ Data stays within organization
  ✓ Aligned with business objectives
  ✓ Builds internal security culture

  DISADVANTAGES:
  ✗ High cost (staff, tools, facility)
  ✗ Difficult to staff 24/7
  ✗ Talent acquisition and retention challenges
  ✗ Requires significant investment
  ✗ Slow to build expertise
  ✗ Tool procurement complexity

  BEST FOR:
  → Large enterprises
  → Highly regulated industries
  → Government/military
  → Organizations with sensitive data
  → Companies with mature security programs

  STAFFING (24/7):
  → Minimum 8-12 analysts (3 shifts)
  → 2-3 senior analysts
  → 1-2 detection engineers
  → 1 SOC manager
  → 1 threat intel analyst
  → Total: 13-19 FTEs minimum
  → Approximate cost: $1.5M-$3M/year (staff only)
```

---

## 3. Outsourced SOC (MSSP/MDR)

```
MANAGED SECURITY SERVICE PROVIDER (MSSP):

  Description:
  → Third-party manages security monitoring
  → Provider owns infrastructure and staff
  → SLA-based service delivery
  → Shared resources across clients

  ADVANTAGES:
  ✓ Lower cost than in-house
  ✓ Immediate 24/7 coverage
  ✓ Access to experienced analysts
  ✓ Scalable services
  ✓ No staffing burden
  ✓ Faster deployment

  DISADVANTAGES:
  ✗ Less control over operations
  ✗ Generic processes (not customized)
  ✗ Shared analyst attention
  ✗ Data sent to third party
  ✗ Vendor lock-in risk
  ✗ Limited environment knowledge
  ✗ Communication overhead

  BEST FOR:
  → Small to medium businesses
  → Organizations lacking security staff
  → Companies needing quick security capability
  → Budget-constrained organizations

MANAGED DETECTION AND RESPONSE (MDR):

  Description:
  → Focused on threat detection and response
  → More specialized than traditional MSSP
  → Active threat hunting
  → Response capabilities (not just monitoring)
  → Uses advanced tools (EDR, XDR)

  MDR vs MSSP:
  Feature          | MSSP           | MDR
  Focus            | Monitoring     | Detection + Response
  Technology       | SIEM/Firewall  | EDR/XDR
  Depth            | Alert-based    | Investigation-based
  Hunting          | Rare           | Standard
  Response         | Alert/escalate | Contain/remediate
  Customization    | Low            | Medium-High
  Cost             | Lower          | Higher

  KEY MDR PROVIDERS:
  → CrowdStrike Falcon Complete
  → SentinelOne Vigilance
  → Arctic Wolf
  → Expel
  → Red Canary
  → Secureworks
  → Binary Defense
```

---

## 4. Hybrid and Virtual SOC

```
HYBRID SOC:

  Description:
  → Combination of internal staff and outsourced services
  → In-house team handles Tier 2/3 analysis
  → MSSP provides 24/7 Tier 1 monitoring
  → Organization retains control of critical decisions
  → Most common model for mid-size enterprises

  TYPICAL STRUCTURE:
  ┌─────────────────────────────────────────┐
  │              HYBRID SOC                  │
  │                                          │
  │  OUTSOURCED (MSSP):     INTERNAL:       │
  │  ┌─────────────────┐  ┌──────────────┐  │
  │  │ 24/7 Monitoring │  │ Tier 2/3     │  │
  │  │ Tier 1 Triage   │  │ Investigation│  │
  │  │ Alert Filtering │  │ IR Lead      │  │
  │  │ Log Management  │  │ Threat Intel │  │
  │  │                 │  │ Threat Hunt  │  │
  │  │                 │  │ Engineering  │  │
  │  └─────────────────┘  └──────────────┘  │
  └─────────────────────────────────────────┘

  ADVANTAGES:
  ✓ Cost-effective 24/7 coverage
  ✓ Retain critical expertise internally
  ✓ Scalable and flexible
  ✓ Best of both models
  ✓ Internal team focuses on high-value work

  DISADVANTAGES:
  ✗ Complex coordination required
  ✗ Communication challenges
  ✗ Split responsibility
  ✗ Integration between teams
  ✗ Requires clear escalation paths

VIRTUAL SOC:

  Description:
  → No physical SOC facility
  → Remote/distributed team members
  → Cloud-based tools and infrastructure
  → Team members may be part-time security
  → On-demand activation for incidents

  ADVANTAGES:
  ✓ Lowest cost
  ✓ Geographic diversity
  ✓ No facility costs
  ✓ Flexible staffing

  DISADVANTAGES:
  ✗ Communication challenges
  ✗ Limited monitoring capability
  ✗ Not suitable for 24/7
  ✗ Coordination overhead
  ✗ Security of remote access

  BEST FOR:
  → Startups
  → Small businesses
  → Organizations with limited budget
  → Initial SOC establishment
```

---

## 5. Selecting the Right Model

```
SOC MODEL COMPARISON:

  Model      | Cost    | Control | Coverage | Expertise | Speed
  In-House   | High    | Full    | Custom   | Internal  | Slow
  MSSP       | Low-Med | Low     | 24/7     | External  | Fast
  MDR        | Medium  | Medium  | 24/7     | External  | Fast
  Hybrid     | Medium  | Medium  | 24/7     | Mixed     | Medium
  Virtual    | Low     | Full    | Limited  | Internal  | Slow
  Co-Managed | Med-High| Shared  | 24/7     | Mixed     | Medium

DECISION FACTORS:
  Factor                  | Weight
  Budget                  | How much can you spend?
  Regulatory requirements | Must data stay in-house?
  Existing staff          | Do you have security talent?
  Environment complexity  | How complex is your IT?
  Risk tolerance          | How much risk is acceptable?
  Time to operationalize  | How quickly do you need it?
  Industry                | What do peers do?

EVOLUTION PATH:
  Virtual SOC
      ↓
  MSSP/MDR (outsourced)
      ↓
  Hybrid SOC (in-house + MSSP)
      ↓
  Full In-House SOC
      ↓
  Advanced SOC (hunting, automation)

CHECKLIST:
  [ ] Assess current security maturity
  [ ] Define budget and timeline
  [ ] Identify regulatory requirements
  [ ] Evaluate available talent
  [ ] Compare vendor options
  [ ] Define SLAs and metrics
  [ ] Plan transition/evolution path
  [ ] Get executive sponsorship
```

---

## Summary Table

| Model | Cost | Control | Best For |
|-------|------|---------|----------|
| In-House | $$$$$ | Full | Large enterprises, regulated industries |
| MSSP | $$ | Low | SMBs needing basic monitoring |
| MDR | $$$ | Medium | Active detection and response |
| Hybrid | $$$$ | Medium | Mid-large orgs, best of both |
| Virtual | $ | Full | Startups, budget-constrained |

---

## Revision Questions

1. What are the main differences between MSSP and MDR services?
2. Why is the hybrid SOC model the most common for mid-size enterprises?
3. What factors should organizations consider when selecting a SOC model?
4. What is the typical evolution path for SOC maturity?
5. What are the minimum staffing requirements for a 24/7 in-house SOC?

---

*Previous: [01-soc-overview.md](01-soc-overview.md) | Next: [03-soc-roles-and-responsibilities.md](03-soc-roles-and-responsibilities.md)*

---

*[Back to README](../README.md)*
