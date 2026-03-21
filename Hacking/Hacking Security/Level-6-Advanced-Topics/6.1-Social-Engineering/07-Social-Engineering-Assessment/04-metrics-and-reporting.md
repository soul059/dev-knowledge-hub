# Unit 7: Social Engineering Assessment — Topic 4: Metrics and Reporting

## Overview

**Metrics and reporting** transform raw social engineering assessment data into actionable intelligence that helps organizations understand their human security posture. Effective reporting quantifies risk, identifies patterns, provides context for findings, and delivers clear recommendations. The report is the primary deliverable — it must be clear, accurate, and compelling.

---

## 1. Key Metrics

```
PHISHING METRICS:

PRIMARY METRICS:
  → Emails Sent:          500
  → Emails Delivered:      487 (97.4%)
  → Emails Opened:         312 (64.1%)
  → Links Clicked:         147 (30.2%)
  → Credentials Submitted:  89 (18.3%)
  → Attachments Opened:     73 (15.0%)
  → Reported to Security:   43 (8.8%)
  → No Action:             175 (35.9%)

DERIVED METRICS:
  → Click-to-Submit Ratio: 60.5%
    (of those who clicked, how many submitted)
  → Report Rate: 8.8%
    (indicates security culture)
  → Time to First Click: 2 min 14 sec
  → Time to First Report: 47 minutes
  → Mean Time to Click: 24 minutes

DEPARTMENT BREAKDOWN:
  Department    | Sent | Clicked | Submitted | Reported
  Finance       |  45  |  18(40%)| 14(31%)   |  2(4%)
  Marketing     |  62  |  28(45%)| 20(32%)   |  3(5%)
  Engineering   |  89  |  15(17%)|  8(9%)    | 12(13%)
  HR            |  35  |  14(40%)|  9(26%)   |  1(3%)
  Executive     |  12  |   6(50%)|  5(42%)   |  0(0%)
  IT            |  55  |   8(15%)|  4(7%)    | 15(27%)

VISHING METRICS:
  → Calls Made:           100
  → Calls Answered:        67 (67%)
  → Info Disclosed:        31 (46% of answered)
  → Credentials Given:     12 (18% of answered)
  → Challenged Caller:     14 (21% of answered)
  → Reported Call:          4 (6% of answered)

PHYSICAL METRICS:
  → Entry Attempts:          5
  → Successful Entries:      4 (80%)
  → Challenged:             1 (20%)
  → Reached Target Area:    3 (60%)
  → Time to Access:         Average 12 min
  → Reported to Security:   0 (0%)

USB DROP METRICS:
  → Devices Dropped:       10
  → Devices Plugged In:     4 (40%)
  → Time to First Plug-in: 23 minutes
  → Reported to Security:   1 (10%)
```

---

## 2. Report Structure

```
SOCIAL ENGINEERING ASSESSMENT REPORT:

1. EXECUTIVE SUMMARY (1-2 pages)
   → High-level findings
   → Overall risk rating
   → Key statistics
   → Critical recommendations
   → Written for executives (non-technical)

2. SCOPE AND METHODOLOGY
   → Assessment scope and objectives
   → Attack vectors used
   → Timeline of activities
   → Tools and techniques employed
   → Limitations and constraints

3. FINDINGS
   → Detailed results per attack vector
   → Supporting evidence
   → Risk ratings per finding
   → Statistical analysis
   → Trend comparison (if repeat assessment)

4. RISK ANALYSIS
   → Risk ratings and justifications
   → Business impact analysis
   → Likelihood and severity matrix
   → Comparison to industry benchmarks

5. RECOMMENDATIONS
   → Prioritized by risk and effort
   → Technical controls
   → Process improvements
   → Training and awareness
   → Quick wins vs long-term improvements

6. APPENDICES
   → Detailed metrics and data
   → Screenshots and evidence
   → Email templates used
   → Timeline of events
   → Methodology details
```

---

## 3. Benchmarking

```
INDUSTRY BENCHMARKS:

PHISHING CLICK RATES (by industry):
  Industry           | Average Click Rate
  Healthcare         | 25-35%
  Education          | 20-30%
  Government         | 15-25%
  Financial Services | 10-20%
  Technology         | 8-15%
  Manufacturing      | 20-30%

MATURITY LEVELS:

LEVEL 1 — INITIAL:
  → Click rate > 30%
  → Report rate < 5%
  → No security awareness program
  → No technical controls

LEVEL 2 — DEVELOPING:
  → Click rate 20-30%
  → Report rate 5-10%
  → Basic awareness training
  → Some email controls

LEVEL 3 — DEFINED:
  → Click rate 10-20%
  → Report rate 10-20%
  → Regular training program
  → Email gateway + MFA

LEVEL 4 — MANAGED:
  → Click rate 5-10%
  → Report rate 20-40%
  → Continuous training + simulations
  → Advanced email security + EDR

LEVEL 5 — OPTIMIZED:
  → Click rate < 5%
  → Report rate > 40%
  → Security culture embedded
  → Full defense-in-depth
  → Continuous improvement cycle
```

---

## 4. Presenting Results

```
PRESENTATION TIPS:

EXECUTIVE AUDIENCE:
  → Lead with business impact
  → Use clear visuals (charts, graphs)
  → Avoid technical jargon
  → Compare to industry benchmarks
  → Focus on risk and recommendations
  → Time: 15-20 minutes

TECHNICAL AUDIENCE:
  → Detailed methodology
  → Tool configurations
  → Technical findings
  → Specific remediation steps
  → Integration with existing tools
  → Time: 30-60 minutes

VISUALIZATION:

  Phishing Results:
  
  Sent    ████████████████████████████████ 500
  Opened  ████████████████████░░░░░░░░░░░░ 312
  Clicked ██████████░░░░░░░░░░░░░░░░░░░░░░ 147
  Submit  ██████░░░░░░░░░░░░░░░░░░░░░░░░░░  89
  Report  ███░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  43

DO:
  → Tell a story with the data
  → Highlight positive findings too
  → Provide actionable recommendations
  → Offer to help with remediation
  → Show improvement over time

DON'T:
  → Shame individual employees
  → Use real names in broad reporting
  → Present data without context
  → Make the organization feel attacked
  → Leave without clear next steps
```

---

## 5. Trend Analysis

```
TRACKING OVER TIME:

ASSESSMENT COMPARISON:
  
  Metric      | Q1 2023 | Q3 2023 | Q1 2024 | Trend
  Click Rate  | 32%     | 24%     | 18%     | ↓ Improving
  Submit Rate | 22%     | 16%     | 11%     | ↓ Improving
  Report Rate |  5%     | 12%     | 22%     | ↑ Improving
  Avg Time    | 3 min   | 8 min   | 15 min  | ↑ Improving

VALUE DEMONSTRATION:
  → Shows ROI of security awareness program
  → Justifies continued investment
  → Identifies persistent problem areas
  → Reveals training effectiveness
  → Supports budget requests

REGRESSION INDICATORS:
  → Click rates increasing
  → Report rates declining
  → Training fatigue
  → New employee onboarding gaps
  → Seasonal variations
  → New attack vector exposure
```

---

## Summary Table

| Metric Category | Key Metrics | Good Target |
|----------------|-------------|-------------|
| Phishing | Click rate, submit rate | <10%, <5% |
| Reporting | Report rate, time to report | >30%, <15 min |
| Vishing | Disclosure rate | <15% |
| Physical | Entry success rate | <20% |
| USB Drop | Plug-in rate | <10% |
| Overall | Security maturity level | Level 4+ |

---

## Revision Questions

1. What are the key metrics for each type of social engineering assessment?
2. How should assessment reports be structured for different audiences?
3. What industry benchmarks exist for phishing assessment results?
4. How do you track and demonstrate security improvement over time?
5. What mistakes should be avoided when presenting assessment results?

---

*Previous: [03-campaign-execution.md](03-campaign-execution.md) | Next: [05-ethical-considerations.md](05-ethical-considerations.md)*

---

*[Back to README](../README.md)*
