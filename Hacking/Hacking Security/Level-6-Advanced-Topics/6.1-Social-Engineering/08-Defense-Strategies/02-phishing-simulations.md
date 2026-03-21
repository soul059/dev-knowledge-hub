# Unit 8: Defense Strategies — Topic 2: Phishing Simulations

## Overview

**Phishing simulations** are controlled, authorized phishing campaigns conducted by an organization against its own employees to test awareness, measure security culture, and provide just-in-time training. Unlike real attacks, simulations are designed to educate rather than exploit. They are the most effective method to measure and improve an organization's human defense against phishing.

---

## 1. Why Run Phishing Simulations

```
BENEFITS:

MEASUREMENT:
  → Baseline security awareness level
  → Track improvement over time
  → Identify high-risk groups/departments
  → Quantify human risk factor
  → Demonstrate ROI of training program

EDUCATION:
  → Just-in-time learning at point of failure
  → More memorable than classroom training
  → Contextual — connected to real behavior
  → Creates teachable moments
  → Reinforces training content

BEHAVIORAL CHANGE:
  → Increases vigilance with each simulation
  → Builds "think before you click" habit
  → Normalizes reporting suspicious emails
  → Creates peer accountability
  → Develops security reflex

COMPLIANCE:
  → Many frameworks require testing
  → PCI DSS, NIST, ISO 27001
  → Insurance requirements
  → Regulatory expectations
  → Audit evidence
```

---

## 2. Simulation Program Design

```
PROGRAM STRUCTURE:

FREQUENCY:
  → Monthly simulations (recommended)
  → Minimum: quarterly
  → Vary timing within month
  → Don't create predictable pattern
  → Year-round continuous program

DIFFICULTY PROGRESSION:
  
  MONTH 1-3: BASIC
    → Generic mass phishing templates
    → Obvious red flags present
    → External sender pretexts
    → Easy to identify if trained
    → Goal: establish baseline

  MONTH 4-6: INTERMEDIATE
    → Industry-specific pretexts
    → Better crafted emails
    → Fewer obvious red flags
    → Internal-looking pretexts
    → Goal: test basic training

  MONTH 7-9: ADVANCED
    → Spear phishing with personalization
    → Current event themes
    → Trusted brand impersonation
    → Subtle red flags only
    → Goal: test advanced awareness

  MONTH 10-12: EXPERT
    → Highly targeted pretexts
    → Reference real projects/people
    → Multiple attack vectors
    → Near-zero obvious indicators
    → Goal: test resilience

TEMPLATE CATEGORIES:
  → IT/Security notifications
  → HR/Benefits communications
  → Shipping/Delivery notifications
  → Social media alerts
  → Financial/Banking
  → Cloud service notifications
  → Internal communications
  → Current events/news
```

---

## 3. Running Simulations

```
SIMULATION EXECUTION:

SETUP:
  1. Select or create email template
  2. Build or clone landing page
  3. Configure tracking (open, click, submit)
  4. Define target groups
  5. Schedule send time
  6. Configure "teachable moment" page
  7. Test everything internally first

TEACHABLE MOMENT PAGE:
  ┌──────────────────────────────────────┐
  │  ⚠ This was a phishing simulation   │
  │                                      │
  │  You clicked a simulated phishing    │
  │  link. Don't worry — this was a     │
  │  training exercise. No harm done!    │
  │                                      │
  │  Here's what to look for:           │
  │  • Check the sender's email address  │
  │  • Hover over links before clicking  │
  │  • Be wary of urgency and fear      │
  │  • Report suspicious emails         │
  │                                      │
  │  [Watch a 2-minute training video]   │
  │  [Take a quick quiz]                │
  └──────────────────────────────────────┘

TRACKING METRICS:
  → Email opened (tracking pixel)
  → Link clicked
  → Credentials submitted
  → Attachment opened
  → Email reported (via report button)
  → No action taken
  → Time to action

POST-SIMULATION:
  → Generate reports
  → Send results to management (aggregate)
  → Individual notifications (educational)
  → Additional training for repeat clickers
  → Recognition for reporters
  → Update metrics dashboard
```

---

## 4. Best Practices

```
DO:
  ✓ Start with baseline assessment
  ✓ Increase difficulty gradually
  ✓ Use varied templates and pretexts
  ✓ Provide immediate educational feedback
  ✓ Track and report improvement
  ✓ Make reporting easy (one-click button)
  ✓ Recognize and reward reporters
  ✓ Use results to improve training
  ✓ Get executive buy-in
  ✓ Communicate the purpose positively

DON'T:
  ✗ Use simulations as a "gotcha"
  ✗ Punish employees who click
  ✗ Name and shame individuals
  ✗ Send simulations during crises
  ✗ Use insensitive pretexts (health, family)
  ✗ Overwhelm with too many simulations
  ✗ Ignore the results
  ✗ Stop after one campaign
  ✗ Create anxiety about email use
  ✗ Forget to celebrate improvements

HANDLING REPEAT CLICKERS:
  → Identify patterns (same person repeatedly)
  → Provide additional one-on-one training
  → Assign specialized e-learning modules
  → Consider role-specific risk factors
  → Explore if technical controls can help
  → Never: threaten, punish, or terminate
  → Document training provided

REPORTING BUTTON:
  → Install in all email clients
  → One-click to report suspicious email
  → Sends to security team for analysis
  → Auto-removes from inbox
  → Provides immediate feedback
  → "Thank you for reporting!"
  → Critical infrastructure for awareness
```

---

## 5. Tools and Platforms

```
SIMULATION PLATFORMS:

GOPHISH (Open Source):
  → Free and self-hosted
  → Full campaign management
  → Landing page cloning
  → Tracking and reporting
  → API for automation
  → Great for small/medium orgs

KNOWBE4:
  → Market leader
  → Extensive template library (14,000+)
  → Automated campaigns
  → Advanced reporting
  → Integrated training platform
  → AI-driven recommendations

COFENSE (PhishMe):
  → Phishing simulation focus
  → Reporter button integration
  → Threat intelligence
  → Incident response integration
  → Human-verified threat intelligence

PROOFPOINT:
  → Integrated with email security
  → Advanced analytics
  → Adaptive simulations
  → Compliance tracking
  → Enterprise-grade

MICROSOFT ATTACK SIMULATION:
  → Built into Microsoft 365 E5
  → No additional cost for M365 users
  → Integrated with Defender
  → Training assignments
  → Good for Microsoft-heavy orgs
```

---

## Summary Table

| Aspect | Recommendation |
|--------|---------------|
| Frequency | Monthly |
| Difficulty | Progressive (basic → expert) |
| Templates | Varied, realistic, ethical |
| Feedback | Immediate teachable moments |
| Reporting | Aggregate, never individual shaming |
| Repeat clickers | Additional training, not punishment |
| Tool | Based on budget and org size |

---

## Revision Questions

1. What are the primary benefits of phishing simulations?
2. How should simulation difficulty progress over time?
3. What should a "teachable moment" page include?
4. What best practices ensure ethical and effective simulations?
5. How should organizations handle repeat clickers?

---

*Previous: [01-security-awareness-training.md](01-security-awareness-training.md) | Next: [03-reporting-mechanisms.md](03-reporting-mechanisms.md)*

---

*[Back to README](../README.md)*
