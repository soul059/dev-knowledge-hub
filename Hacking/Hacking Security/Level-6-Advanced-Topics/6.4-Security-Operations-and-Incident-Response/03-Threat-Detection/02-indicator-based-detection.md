# Unit 3: Threat Detection — Topic 2: Indicator-Based Detection

## Overview

**Indicator-based detection** (also called signature-based or IOC-based detection) identifies threats by matching observed artifacts against known indicators of compromise. This is the most fundamental detection method, using known-bad IP addresses, file hashes, domain names, and other observable artifacts to identify threats.

---

## 1. Indicators of Compromise (IOCs)

```
IOC TYPES:

  ┌──────────────────────────────────────────────────┐
  │            INDICATORS OF COMPROMISE               │
  │                                                    │
  │  NETWORK IOCs:                                    │
  │  → IP addresses (C2 servers)                      │
  │  → Domain names (malicious domains)               │
  │  → URLs (phishing, malware delivery)              │
  │  → JA3/JA3S hashes (TLS fingerprints)            │
  │  → User-Agent strings                             │
  │  → DNS patterns                                   │
  │                                                    │
  │  HOST IOCs:                                       │
  │  → File hashes (MD5, SHA256)                      │
  │  → File names and paths                           │
  │  → Registry keys/values                           │
  │  → Mutex names                                    │
  │  → Service names                                  │
  │  → Scheduled task names                           │
  │                                                    │
  │  EMAIL IOCs:                                      │
  │  → Sender addresses                               │
  │  → Subject line patterns                          │
  │  → Attachment hashes                               │
  │  → Header anomalies                               │
  │                                                    │
  │  BEHAVIORAL IOCs:                                 │
  │  → Process execution patterns                     │
  │  → Command line arguments                         │
  │  → Parent-child process relationships             │
  │  → Network connection patterns                    │
  └──────────────────────────────────────────────────┘

IOC LIFECYCLE:
  Discovery → Validation → Distribution → Detection →
  → Monitoring → Expiration/Retirement
```

---

## 2. IOC Sources and Feeds

```
THREAT INTELLIGENCE SOURCES:

OPEN SOURCE FEEDS:
  Feed                  | IOC Types        | Format
  AlienVault OTX       | IP, Domain, Hash | STIX/TAXII
  Abuse.ch (URLhaus)   | URLs, Malware    | CSV, JSON
  Abuse.ch (MalBazaar) | Malware hashes   | CSV, JSON
  Abuse.ch (ThreatFox) | IOCs             | CSV, JSON
  VirusTotal           | Hashes, IPs, URLs| API
  AbuseIPDB            | Malicious IPs    | API
  PhishTank            | Phishing URLs    | CSV
  EmergingThreats      | Snort/Suricata   | Rules
  CIRCL MISP           | Various          | MISP
  CISA AIS             | Various          | STIX/TAXII

COMMERCIAL FEEDS:
  → Recorded Future
  → CrowdStrike Intelligence
  → Mandiant Advantage
  → ThreatConnect
  → Anomali ThreatStream
  → Intel471
  → Flashpoint

IOC MANAGEMENT PLATFORMS:
  MISP (Open Source):
  → Threat intelligence sharing platform
  → IOC storage and correlation
  → Community sharing
  → STIX/TAXII support
  → Automation and enrichment
  
  OpenCTI (Open Source):
  → Knowledge management
  → STIX 2.1 native
  → Graph visualization
  → Connector ecosystem

IOC QUALITY ASSESSMENT:
  Factor           | Consideration
  Age              | How recent? (older = less relevant)
  Confidence       | How sure is the source?
  Context          | What attack/actor is it linked to?
  Specificity      | How unique is this indicator?
  Source reputation | How reliable is the source?
```

---

## 3. Implementing IOC Detection

```
SIEM IOC MATCHING:

SPLUNK:
  # IP matching against threat list
  index=firewall
  | lookup threat_intel_ip ip AS dest_ip
    OUTPUT threat_description, threat_score
  | where isnotnull(threat_description)
  
  # Hash matching
  index=sysmon EventCode=1
  | lookup threat_intel_hash hash AS Hashes
    OUTPUT malware_family, confidence
  | where isnotnull(malware_family)

SENTINEL (KQL):
  // IP matching against TI
  let malicious_ips = ThreatIntelligenceIndicator
    | where isnotempty(NetworkIP)
    | distinct NetworkIP;
  CommonSecurityLog
  | where DestinationIP in (malicious_ips)
  | project TimeGenerated, SourceIP, DestinationIP

ELASTIC:
  # Threat intel matching
  GET security-*/_search
  {
    "query": {
      "bool": {
        "must": [
          {"match": {"threat.indicator.type": "ipv4-addr"}},
          {"exists": {"field": "threat.indicator.matched"}}
        ]
      }
    }
  }

SNORT/SURICATA RULES:
  # Block known C2 IP
  alert tcp $HOME_NET any -> 203.0.113.50 any
  (msg:"Known C2 Server"; sid:1000001; rev:1;)
  
  # Detect malicious domain
  alert dns $HOME_NET any -> any any
  (msg:"Malicious Domain Query";
   dns.query; content:"malware.evil.com";
   sid:1000002; rev:1;)

YARA RULES:
  rule Suspicious_PowerShell {
    meta:
      description = "Detects encoded PowerShell"
      author = "SOC Team"
    strings:
      $enc = "-EncodedCommand" nocase
      $bypass = "-ExecutionPolicy Bypass" nocase
      $hidden = "-WindowStyle Hidden" nocase
    condition:
      2 of them
  }
```

---

## 4. IOC Enrichment and Correlation

```
IOC ENRICHMENT:

ENRICHMENT WORKFLOW:
  IOC Detected → Lookup Context → Score Risk → Alert

  ┌───────────┐    ┌──────────────┐    ┌──────────┐
  │ Matched   │───▶│ Enrichment   │───▶│ Scored   │
  │ IOC       │    │              │    │ Alert    │
  └───────────┘    │ VirusTotal   │    └──────────┘
                   │ AbuseIPDB    │
                   │ WHOIS        │
                   │ GeoIP        │
                   │ Passive DNS  │
                   │ Shodan       │
                   └──────────────┘

ENRICHMENT SOURCES:
  Source       | Data Provided
  VirusTotal   | Hash/IP/URL reputation, AV detections
  AbuseIPDB    | IP abuse reports, confidence score
  WHOIS        | Domain registration, registrant
  GeoIP        | Geographic location, ASN
  Passive DNS  | Historical DNS resolutions
  Shodan       | Open ports, services, banners
  URLhaus      | URL malware distribution history
  ThreatFox    | IOC context, malware family
  GreyNoise    | Internet scanner identification

CORRELATION WITH CONTEXT:
  → IOC + Asset criticality = Priority
  → IOC + Vulnerability data = Exploitability
  → IOC + User context = Impact assessment
  → IOC + Historical hits = Pattern identification
  → IOC + TI report = Attribution/campaign
```

---

## 5. Limitations and Best Practices

```
LIMITATIONS OF IOC-BASED DETECTION:

  ✗ Reactive (requires known indicators)
  ✗ Easy to evade (change hash, IP, domain)
  ✗ High volume of indicators to manage
  ✗ Short shelf life (indicators age quickly)
  ✗ Cannot detect novel/zero-day attacks
  ✗ False positives from stale indicators
  ✗ CDN/shared hosting false positives
  ✗ Encrypted traffic limits visibility

BEST PRACTICES:

  IOC MANAGEMENT:
  [ ] Automate IOC ingestion from feeds
  [ ] Set expiration dates on indicators
  [ ] Score IOCs by confidence level
  [ ] Validate IOCs before deployment
  [ ] Track IOC hit rates
  [ ] Remove stale indicators regularly
  [ ] Correlate IOCs with context
  [ ] Use multiple feed sources
  [ ] Share IOCs with community (MISP)
  [ ] Track false positive rate per feed

  DETECTION STRATEGY:
  → Use IOCs as one layer of detection
  → Combine with behavioral detection
  → Prioritize high-confidence IOCs
  → Focus on indicators higher on Pyramid of Pain
  → Automate enrichment for all matches
  → Don't rely solely on IOC-based detection

  IOC FEED EVALUATION:
  Feed Quality Metrics:
  → Hit rate (% of IOCs that trigger)
  → False positive rate
  → Freshness (time from discovery to feed)
  → Coverage (types of threats)
  → Uniqueness (IOCs not in other feeds)
  → Cost vs value
```

---

## Summary Table

| IOC Type | Example | Evasion Difficulty | Detection Value |
|----------|---------|-------------------|-----------------|
| Hash | SHA256 of malware | Very Easy | Low |
| IP Address | C2 server IP | Easy | Low-Medium |
| Domain | Malicious domain | Easy | Medium |
| URL | Phishing URL | Easy | Medium |
| JA3 Hash | TLS fingerprint | Medium | Medium-High |
| Behavior Pattern | Process chain | Hard | High |

---

## Revision Questions

1. What are the main types of indicators of compromise?
2. What open-source threat intelligence feeds are available?
3. How are IOCs matched in SIEM platforms?
4. What are the limitations of indicator-based detection?
5. How should IOC feeds be evaluated for quality?

---

*Previous: [01-detection-engineering.md](01-detection-engineering.md) | Next: [03-behavior-based-detection.md](03-behavior-based-detection.md)*

---

*[Back to README](../README.md)*
