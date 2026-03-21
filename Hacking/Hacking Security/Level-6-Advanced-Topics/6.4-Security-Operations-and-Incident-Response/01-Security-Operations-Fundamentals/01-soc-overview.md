# Unit 1: Security Operations Fundamentals — Topic 1: SOC Overview

## Overview

A **Security Operations Center (SOC)** is the centralized function within an organization responsible for continuously monitoring and improving the organization's security posture while preventing, detecting, analyzing, and responding to cybersecurity incidents. The SOC serves as the nerve center for security operations, integrating people, processes, and technology to defend against threats.

---

## 1. What is a SOC?

```
SOC DEFINITION:

  A centralized unit that deals with security issues
  on an organizational and technical level.

  ┌──────────────────────────────────────────────────┐
  │              SECURITY OPERATIONS CENTER           │
  │                                                    │
  │  ┌────────────┐  ┌────────────┐  ┌────────────┐  │
  │  │   PEOPLE   │  │  PROCESS   │  │ TECHNOLOGY │  │
  │  │            │  │            │  │            │  │
  │  │ Analysts   │  │ Playbooks  │  │ SIEM       │  │
  │  │ Engineers  │  │ Procedures │  │ EDR        │  │
  │  │ Managers   │  │ Workflows  │  │ SOAR       │  │
  │  │ Hunters    │  │ Policies   │  │ Firewall   │  │
  │  │ IR Team    │  │ Runbooks   │  │ IDS/IPS    │  │
  │  └────────────┘  └────────────┘  └────────────┘  │
  │                                                    │
  │  MISSION: Detect, Analyze, Respond to Threats     │
  └──────────────────────────────────────────────────┘

SOC FUNCTIONS:
  1. MONITORING
     → 24/7/365 security monitoring
     → Real-time alert analysis
     → Dashboard monitoring
     → Network traffic analysis
  
  2. DETECTION
     → Threat detection (known & unknown)
     → Anomaly identification
     → Correlation of events
     → Indicator matching
  
  3. INVESTIGATION
     → Alert triage and validation
     → Deep-dive analysis
     → Root cause determination
     → Scope assessment
  
  4. RESPONSE
     → Incident containment
     → Threat mitigation
     → Recovery coordination
     → Communication management
  
  5. PREVENTION
     → Vulnerability management support
     → Security recommendations
     → Threat intelligence consumption
     → Configuration review
```

---

## 2. SOC Mission and Objectives

```
PRIMARY OBJECTIVES:

  PROTECT:
  → Continuously monitor the environment
  → Reduce attack surface
  → Implement security controls
  → Maintain security posture

  DETECT:
  → Identify threats quickly (reduce MTTD)
  → Detect known and unknown threats
  → Correlate events for patterns
  → Leverage threat intelligence

  RESPOND:
  → Contain threats rapidly (reduce MTTR)
  → Minimize business impact
  → Coordinate remediation
  → Document and report incidents

  IMPROVE:
  → Learn from incidents
  → Enhance detection capabilities
  → Reduce false positives
  → Automate repetitive tasks

KEY PERFORMANCE INDICATORS:
  Metric                     | Target
  Mean Time to Detect (MTTD) | < 24 hours
  Mean Time to Respond (MTTR)| < 4 hours
  Alert-to-Triage Time       | < 15 minutes
  False Positive Rate        | < 30%
  Incidents Resolved         | > 95% within SLA
  Escalation Rate            | < 20%
  Coverage (24/7)            | 100%
```

---

## 3. SOC Architecture

```
SOC ARCHITECTURE:

  ┌─────────────────────────────────────────────────┐
  │                   DATA SOURCES                   │
  │  Endpoints │ Network │ Cloud │ Apps │ Identity   │
  └──────────────────────┬──────────────────────────┘
                         │
  ┌──────────────────────▼──────────────────────────┐
  │              DATA COLLECTION LAYER               │
  │  Log Collectors │ Agents │ API │ Syslog │ Beats  │
  └──────────────────────┬──────────────────────────┘
                         │
  ┌──────────────────────▼──────────────────────────┐
  │           DATA PROCESSING & STORAGE              │
  │  SIEM │ Data Lake │ Indexing │ Normalization     │
  └──────────────────────┬──────────────────────────┘
                         │
  ┌──────────────────────▼──────────────────────────┐
  │              DETECTION & ANALYTICS               │
  │  Rules │ ML │ Correlation │ Threat Intel │ UEBA  │
  └──────────────────────┬──────────────────────────┘
                         │
  ┌──────────────────────▼──────────────────────────┐
  │            RESPONSE & ORCHESTRATION              │
  │  SOAR │ Ticketing │ Playbooks │ Automation       │
  └──────────────────────┬──────────────────────────┘
                         │
  ┌──────────────────────▼──────────────────────────┐
  │              REPORTING & VISIBILITY              │
  │  Dashboards │ Reports │ Metrics │ Executive View │
  └─────────────────────────────────────────────────┘

TECHNOLOGY STACK:
  Layer              | Components
  Collection         | Syslog, Beats, Agents, API connectors
  SIEM               | Splunk, Elastic, Sentinel, QRadar
  EDR                | CrowdStrike, SentinelOne, Defender
  NDR                | Zeek, Darktrace, ExtraHop
  SOAR               | Splunk SOAR, XSOAR, Shuffle
  Threat Intel       | MISP, OTX, ThreatConnect
  Ticketing          | TheHive, ServiceNow, Jira
  Vulnerability      | Qualys, Nessus, Rapid7
  Identity           | Azure AD, CyberArk, Okta
```

---

## 4. SOC Maturity Model

```
SOC MATURITY LEVELS:

LEVEL 0: NO SOC
  → No formal security monitoring
  → Reactive only (after breach)
  → No dedicated security staff
  → Ad-hoc incident handling

LEVEL 1: INITIAL / AD-HOC
  → Basic monitoring (firewall logs, antivirus)
  → Part-time security staff
  → Manual processes
  → Limited documentation
  → Reactive posture
  → No formal IR process

LEVEL 2: MANAGED / DEFINED
  → SIEM deployed and configured
  → Dedicated SOC team (business hours)
  → Basic playbooks and procedures
  → Regular vulnerability scanning
  → Log collection from key sources
  → Basic threat intelligence

LEVEL 3: ESTABLISHED / MEASURED
  → 24/7 monitoring capability
  → Tiered analyst structure
  → Comprehensive detection rules
  → Formal IR process
  → Threat intelligence program
  → Metrics-driven operations
  → Regular tabletop exercises

LEVEL 4: ADVANCED / OPTIMIZED
  → Threat hunting program
  → Advanced analytics (ML/UEBA)
  → SOAR automation
  → Purple team exercises
  → Detection engineering program
  → Continuous improvement cycle
  → Industry-leading practices

LEVEL 5: INNOVATIVE
  → Cutting-edge detection
  → Full automation where possible
  → Predictive capabilities
  → Threat intelligence production
  → Open-source contribution
  → Research and development
  → Community leadership

MATURITY ASSESSMENT:
  Area            | Questions
  People          | Skills, staffing, training?
  Process         | Documented, repeatable, measured?
  Technology      | Integrated, automated, effective?
  Intelligence    | Consumed, produced, shared?
  Detection       | Coverage, accuracy, timeliness?
  Response        | Speed, effectiveness, coordination?
```

---

## 5. Building a SOC

```
SOC IMPLEMENTATION ROADMAP:

PHASE 1: FOUNDATION (Months 1-3)
  → Define mission and objectives
  → Identify stakeholders
  → Budget approval
  → Hire initial team (SOC Manager, 2-3 analysts)
  → Select and deploy SIEM
  → Connect critical log sources
  → Establish basic procedures

PHASE 2: CORE CAPABILITIES (Months 3-6)
  → Expand log collection
  → Build detection rules
  → Create initial playbooks
  → Deploy EDR solution
  → Establish monitoring schedules
  → Integrate threat intelligence feeds
  → Implement ticketing system

PHASE 3: MATURATION (Months 6-12)
  → Move toward 24/7 coverage
  → Hire additional analysts
  → Expand detection coverage
  → Automate repetitive tasks
  → Establish metrics program
  → Conduct tabletop exercises
  → Refine processes

PHASE 4: OPTIMIZATION (Year 2+)
  → Deploy SOAR platform
  → Start threat hunting
  → Advanced analytics
  → Purple team exercises
  → Detection engineering program
  → Continuous improvement
  → Measure and benchmark

BUDGET CONSIDERATIONS:
  Category         | % of Budget
  People           | 50-60%
  Technology       | 25-35%
  Process/Training | 10-15%
  Facilities       | 5-10%

COMMON CHALLENGES:
  → Alert fatigue (too many false positives)
  → Staff burnout and retention
  → Tool sprawl (too many disconnected tools)
  → Lack of executive support
  → Insufficient logging coverage
  → Inadequate processes
  → Skills gap
  → Budget constraints
```

---

## Summary Table

| SOC Function | Description | Key Outcome |
|-------------|-------------|-------------|
| Monitoring | 24/7 security observation | Continuous visibility |
| Detection | Threat identification | Early warning |
| Investigation | Alert analysis and triage | Accurate assessment |
| Response | Incident handling | Threat mitigation |
| Prevention | Proactive security | Reduced risk |
| Improvement | Post-incident learning | Enhanced capability |

---

## Revision Questions

1. What are the primary functions of a Security Operations Center?
2. How does the SOC maturity model help organizations assess their capabilities?
3. What are the key components of SOC architecture?
4. What metrics are used to measure SOC effectiveness?
5. What are the main challenges in building and operating a SOC?

---

*Previous: None (First topic in this unit) | Next: [02-soc-models.md](02-soc-models.md)*

---

*[Back to README](../README.md)*
