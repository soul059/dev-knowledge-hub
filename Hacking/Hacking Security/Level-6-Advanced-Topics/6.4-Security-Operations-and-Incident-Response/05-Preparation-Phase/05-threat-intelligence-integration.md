# Unit 5: Preparation Phase — Topic 5: Threat Intelligence Integration

## Overview

**Threat intelligence (TI) integration** into incident response enhances detection capabilities, accelerates investigation, and informs strategic decision-making. By consuming, analyzing, and applying threat intelligence, IR teams can better anticipate, detect, and respond to threats relevant to their organization.

---

## 1. Threat Intelligence Fundamentals

```
THREAT INTELLIGENCE TYPES:

  STRATEGIC:
  → High-level trends and risk
  → Audience: Executives, leadership
  → Format: Reports, briefings
  → Example: "APT groups targeting financial sector"

  TACTICAL:
  → TTPs of threat actors
  → Audience: SOC, IR teams
  → Format: MITRE ATT&CK mappings, playbooks
  → Example: "APT29 uses OAuth token theft"

  OPERATIONAL:
  → Details of specific attacks/campaigns
  → Audience: IR team, threat hunters
  → Format: Campaign reports, IOC packages
  → Example: "Active phishing campaign targeting org"

  TECHNICAL:
  → Specific indicators (IOCs)
  → Audience: Detection systems
  → Format: IP, hash, domain, URL lists
  → Example: "C2 IP: 203.0.113.50"

INTELLIGENCE LIFECYCLE:
  ┌───────────┐
  │ Direction │ ← What do we need to know?
  └─────┬─────┘
        ▼
  ┌───────────┐
  │ Collection│ ← Gather raw data
  └─────┬─────┘
        ▼
  ┌───────────┐
  │ Processing│ ← Normalize, filter, enrich
  └─────┬─────┘
        ▼
  ┌───────────┐
  │ Analysis  │ ← Assess relevance and impact
  └─────┬─────┘
        ▼
  ┌───────────┐
  │Disseminat.│ ← Share with stakeholders
  └─────┬─────┘
        ▼
  ┌───────────┐
  │ Feedback  │ ← Was it useful?
  └───────────┘
```

---

## 2. TI Sources and Platforms

```
INTELLIGENCE SOURCES:

OPEN SOURCE (OSINT):
  → AlienVault OTX
  → Abuse.ch (URLhaus, MalBazaar, ThreatFox)
  → VirusTotal
  → AbuseIPDB
  → CIRCL (MISP community)
  → CISA Alerts and Advisories
  → US-CERT
  → Shodan
  → GreyNoise

COMMERCIAL:
  → Recorded Future
  → CrowdStrike Intelligence
  → Mandiant Advantage
  → Intel471
  → Flashpoint
  → ThreatConnect
  → Anomali

GOVERNMENT:
  → CISA (US)
  → NCSC (UK)
  → BSI (Germany)
  → FBI Flash Alerts
  → NSA Advisories

INDUSTRY (ISACs):
  → FS-ISAC (Financial Services)
  → H-ISAC (Healthcare)
  → MS-ISAC (State/Local Gov)
  → IT-ISAC (IT sector)
  → ONG-ISAC (Oil & Gas)

TI PLATFORMS:
  Platform    | Type       | Key Feature
  MISP        | Open source| Sharing, correlation
  OpenCTI     | Open source| Knowledge management
  ThreatConnect| Commercial| Analysis + automation
  Anomali     | Commercial | ThreatStream platform
  Recorded Future| Commercial| ML-powered intelligence
```

---

## 3. Integration with SOC/IR

```
TI INTEGRATION POINTS:

  ┌─────────────────────────────────────────────┐
  │     THREAT INTELLIGENCE INTEGRATION         │
  │                                              │
  │  ┌──────────┐    ┌───────────────────────┐  │
  │  │   TI     │───▶│ SIEM                  │  │
  │  │ Platform │    │ IOC matching           │  │
  │  │ (MISP)   │    │ Correlation enrichment │  │
  │  │          │───▶│ Alert context          │  │
  │  └──────────┘    └───────────────────────┘  │
  │       │                                      │
  │       │          ┌───────────────────────┐   │
  │       ├─────────▶│ EDR                   │   │
  │       │          │ IOC watchlists         │   │
  │       │          │ Behavioral detection   │   │
  │       │          └───────────────────────┘   │
  │       │                                      │
  │       │          ┌───────────────────────┐   │
  │       ├─────────▶│ Firewall / Proxy      │   │
  │       │          │ Block known-bad IPs    │   │
  │       │          │ Block malicious domains│   │
  │       │          └───────────────────────┘   │
  │       │                                      │
  │       │          ┌───────────────────────┐   │
  │       └─────────▶│ SOAR                  │   │
  │                  │ Enrichment playbooks   │   │
  │                  │ Automated lookups      │   │
  │                  └───────────────────────┘   │
  └─────────────────────────────────────────────┘

SHARING STANDARDS:
  → STIX (Structured Threat Information Expression)
    - Standard format for TI data
    - JSON-based, rich schema
    - Supports relationships between objects
  
  → TAXII (Trusted Automated Exchange)
    - Transport mechanism for STIX
    - Server/client model
    - Collections and channels
  
  → TLP (Traffic Light Protocol)
    - RED: Named recipients only
    - AMBER: Organization + clients
    - GREEN: Community
    - CLEAR: Public
```

---

## 4. Operationalizing Intelligence

```
OPERATIONAL INTELLIGENCE WORKFLOW:

  FOR INCIDENTS:
  1. Receive intelligence (feed, alert, report)
  2. Assess relevance to organization
  3. Extract actionable indicators
  4. Search for indicators in environment
  5. If found: Initiate investigation
  6. If not found: Monitor prospectively
  7. Share findings with community

  FOR DETECTION:
  1. Review new TI reports
  2. Identify TTPs used by relevant actors
  3. Map to MITRE ATT&CK
  4. Create or update detection rules
  5. Test with attack simulation
  6. Deploy to production

  FOR HUNTING:
  1. Develop hypothesis from intelligence
  2. Identify relevant data sources
  3. Build hunting queries
  4. Execute hunt
  5. Document findings
  6. Create detections for confirmed threats

MISP INTEGRATION EXAMPLE:
  # Export IOCs from MISP to SIEM
  # MISP → STIX/TAXII → SIEM
  
  # Python MISP API
  from pymisp import PyMISP
  misp = PyMISP('https://misp.org', 'API_KEY')
  
  # Get recent events
  events = misp.search(published=True, 
                        last='1d',
                        type_attribute='ip-dst')
  
  # Extract IOCs
  for event in events:
      for attr in event.attributes:
          print(f"{attr.type}: {attr.value}")
```

---

## 5. Measuring TI Effectiveness

```
TI METRICS:

CONSUMPTION METRICS:
  → Number of feeds consumed
  → IOCs ingested per day
  → Feed coverage (threat types)
  → Feed quality (FP rate)
  → Time from publication to ingestion

DETECTION METRICS:
  → IOC matches per day/week
  → True positive rate of TI matches
  → Threats detected via TI vs other methods
  → Time to detect TI-matched threats
  → Detection rule created from TI

OPERATIONAL METRICS:
  → TI-informed investigations
  → TI-driven hunting campaigns
  → TI-based detection improvements
  → Reports produced/consumed
  → Intelligence shared with community

MATURITY MODEL:
  Level 1: Consume IOC feeds only
  Level 2: Integrate with SIEM/EDR
  Level 3: Tactical TI (TTP-based)
  Level 4: Produce and share intelligence
  Level 5: Intelligence-driven operations

TI PROGRAM CHECKLIST:
  [ ] TI platform deployed (MISP/OpenCTI)
  [ ] Multiple feed sources integrated
  [ ] SIEM/EDR IOC matching configured
  [ ] TI analyst assigned (dedicated or shared)
  [ ] Regular TI briefings for IR team
  [ ] Intelligence-driven detection creation
  [ ] Hunting informed by intelligence
  [ ] Community sharing participation
  [ ] TI effectiveness metrics tracked
  [ ] Annual TI program review
```

---

## Summary Table

| TI Type | Audience | Format | Use Case |
|---------|----------|--------|----------|
| Strategic | Executive | Reports | Risk decisions |
| Tactical | SOC/IR | ATT&CK TTPs | Detection design |
| Operational | IR/Hunting | Campaign details | Investigation |
| Technical | Systems | IOCs | Automated detection |

---

## Revision Questions

1. What are the four types of threat intelligence?
2. What open-source TI platforms are available?
3. How is threat intelligence integrated with SIEM and EDR?
4. What are STIX and TAXII and how do they support TI sharing?
5. How is threat intelligence program effectiveness measured?

---

*Previous: [04-training-and-exercises.md](04-training-and-exercises.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*
