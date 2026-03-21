# Unit 2: SIEM Fundamentals — Topic 3: Normalization and Parsing

## Overview

**Log normalization and parsing** transforms raw log data from diverse sources into a consistent, structured format that enables efficient searching, correlation, and analysis. Without proper normalization, analysts struggle to correlate events across different systems, and detection rules become fragile and source-specific.

---

## 1. Parsing Fundamentals

```
WHAT IS LOG PARSING?

  RAW LOG → PARSER → STRUCTURED FIELDS

  Input (raw syslog):
  Jan 15 10:30:45 fw01 %ASA-6-302013: Built outbound TCP
  connection 12345 for outside:203.0.113.50/443
  (203.0.113.50/443) to inside:192.168.1.100/54321
  (10.0.0.100/54321)

  Output (parsed fields):
  {
    "timestamp": "2024-01-15T10:30:45Z",
    "source": "fw01",
    "vendor": "Cisco",
    "product": "ASA",
    "severity": 6,
    "event_id": "302013",
    "action": "Built",
    "direction": "outbound",
    "protocol": "TCP",
    "src_ip": "192.168.1.100",
    "src_port": 54321,
    "dst_ip": "203.0.113.50",
    "dst_port": 443,
    "connection_id": "12345"
  }

PARSING TECHNIQUES:
  → Regular Expressions (regex)
  → Grok patterns (named regex)
  → Key-value parsing
  → JSON/XML parsing
  → CSV parsing
  → Delimiter-based splitting
  → Fixed-width parsing
```

---

## 2. Normalization Schemas

```
COMMON NORMALIZATION SCHEMAS:

ELASTIC COMMON SCHEMA (ECS):
  → Standard field naming convention
  → Nested object structure
  → Widely adopted with Elastic
  → Versioned schema
  
  Key Field Groups:
  → @timestamp: Event timestamp
  → source.ip / destination.ip
  → source.port / destination.port
  → event.action: What happened
  → event.category: High-level category
  → event.outcome: success/failure
  → user.name: Username
  → process.name: Process name
  → host.name: Hostname

OPEN CYBERSECURITY SCHEMA FRAMEWORK (OCSF):
  → AWS-led open standard
  → Vendor-neutral schema
  → Event categories and classes
  → Comprehensive attribute dictionary

COMMON EVENT FORMAT (CEF):
  → ArcSight format
  → Header + Extension
  → Key=Value pairs
  
  Format:
  CEF:Version|Vendor|Product|Version|EventID|Name|Severity|Extension
  
  Example:
  CEF:0|Cisco|ASA|9.12|302013|TCP connection built|3|
  src=192.168.1.100 spt=54321 dst=203.0.113.50 dpt=443

COMMON INFORMATION MODEL (CIM):
  → Splunk's normalization model
  → Data models with standard fields
  → Accelerated searches
  → Add-on system for field mapping

FIELD MAPPING EXAMPLE:
  Raw Field       | ECS            | CIM (Splunk)   | CEF
  Source IP        | source.ip      | src_ip         | src
  Destination IP   | destination.ip | dest_ip        | dst
  Source Port      | source.port    | src_port       | spt
  Username         | user.name      | user           | suser
  Action           | event.action   | action         | act
  Result           | event.outcome  | status         | outcome
```

---

## 3. Parser Development

```
BUILDING PARSERS:

GROK PATTERNS (Logstash/Elastic):
  # Apache access log
  %{IPORHOST:source.ip} - %{DATA:user.name}
  \[%{HTTPDATE:timestamp}\]
  "%{WORD:http.request.method} %{DATA:url.path}
  HTTP/%{NUMBER:http.version}"
  %{NUMBER:http.response.status_code}
  %{NUMBER:http.response.bytes}

  # Common Grok patterns
  %{IP}          → IP address
  %{HOSTNAME}    → Hostname
  %{NUMBER}      → Numeric value
  %{WORD}        → Single word
  %{DATA}        → Any data (non-greedy)
  %{GREEDYDATA}  → Any data (greedy)

REGEX PARSING:
  # Windows Security Event 4624
  Pattern: Account Name:\s+(\S+).*Logon Type:\s+(\d+)
  .*Source Network Address:\s+(\S+)
  
  Captures:
  Group 1: username
  Group 2: logon_type
  Group 3: source_ip

SPLUNK FIELD EXTRACTION:
  # props.conf
  [source::WinEventLog:Security]
  EXTRACT-user = Account Name:\s+(?P<user>\S+)
  EXTRACT-src = Source Network Address:\s+(?P<src_ip>\S+)
  EXTRACT-logon = Logon Type:\s+(?P<logon_type>\d+)

  # transforms.conf
  [extract_firewall]
  REGEX = src=(\d+\.\d+\.\d+\.\d+).*dst=(\d+\.\d+\.\d+\.\d+)
  FORMAT = src_ip::$1 dst_ip::$2

KQL PARSING (Sentinel):
  Syslog
  | parse SyslogMessage with * "src=" SrcIP:string " " *
  | parse SyslogMessage with * "dst=" DstIP:string " " *
  | parse SyslogMessage with * "action=" Action:string " " *

PARSER TESTING:
  → Test with sample logs from production
  → Verify all fields extract correctly
  → Test edge cases (missing fields, unusual formats)
  → Validate timestamp parsing
  → Check performance at scale
```

---

## 4. Enrichment

```
LOG ENRICHMENT:

  WHAT IS ENRICHMENT?
  → Adding context to parsed events
  → Threat intelligence lookups
  → Asset information
  → Geographic data
  → User identity correlation
  → Business context

  ENRICHMENT TYPES:
  
  GEO-IP ENRICHMENT:
  → source.ip → source.geo.country
  → MaxMind GeoIP database
  → City, country, coordinates, ASN
  
  THREAT INTELLIGENCE:
  → IP → known malicious? TI score?
  → Hash → known malware family?
  → Domain → known C2?
  → URL → phishing? malware delivery?
  
  ASSET ENRICHMENT:
  → IP → hostname, owner, department
  → hostname → criticality, OS, role
  → MAC → vendor, device type
  → Software inventory correlation
  
  USER ENRICHMENT:
  → username → full name, department
  → username → role, manager
  → username → risk score
  → username → access level

  ENRICHMENT PIPELINE:
  ┌─────────┐    ┌────────┐    ┌──────────┐    ┌──────┐
  │ Raw Log │───▶│ Parse  │───▶│ Enrich   │───▶│ SIEM │
  └─────────┘    └────────┘    │          │    └──────┘
                               │ +GeoIP   │
                               │ +TI      │
                               │ +Asset   │
                               │ +User    │
                               └──────────┘
```

---

## 5. Common Challenges

```
PARSING AND NORMALIZATION CHALLENGES:

FORMAT INCONSISTENCY:
  → Same vendor, different versions = different formats
  → Multiline log events
  → Character encoding issues
  → Timestamp format variations
  → Unstructured vs structured logs

TIMESTAMP CHALLENGES:
  → Multiple timestamp formats
  → Timezone inconsistencies
  → Clock skew between systems
  → Missing timestamps
  
  Solutions:
  → Standardize to UTC
  → Use NTP across all systems
  → Include timezone in log config
  → Validate timestamp parsing

SCALABILITY:
  → High EPS requiring fast parsing
  → Complex regex = slow parsing
  → Enrichment lookup latency
  → Storage for enriched data
  
  Solutions:
  → Pre-process at collector level
  → Use efficient parsing (Grok > regex)
  → Cache enrichment lookups
  → Distributed processing

QUALITY ASSURANCE:
  [ ] Field extraction validated
  [ ] Timestamp correctly parsed
  [ ] All critical fields mapped to schema
  [ ] Enrichment working correctly
  [ ] Edge cases tested
  [ ] Performance under load tested
  [ ] Documentation updated
  [ ] Parser version controlled
```

---

## Summary Table

| Schema | Creator | Format | Adoption |
|--------|---------|--------|----------|
| ECS | Elastic | JSON nested | Elastic ecosystem |
| CIM | Splunk | Data models | Splunk ecosystem |
| CEF | ArcSight | Key=Value | Legacy, universal |
| OCSF | AWS/community | JSON | Growing |

---

## Revision Questions

1. What is the difference between parsing and normalization?
2. What are the major normalization schemas and their differences?
3. How do Grok patterns simplify log parsing?
4. What types of enrichment add the most value to security events?
5. What are the common challenges in log normalization?

---

*Previous: [02-log-collection.md](02-log-collection.md) | Next: [04-correlation-rules.md](04-correlation-rules.md)*

---

*[Back to README](../README.md)*
