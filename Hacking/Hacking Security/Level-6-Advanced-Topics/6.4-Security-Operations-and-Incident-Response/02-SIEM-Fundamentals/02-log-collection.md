# Unit 2: SIEM Fundamentals — Topic 2: Log Collection

## Overview

**Log collection** is the foundation of SIEM operations — without reliable, comprehensive log ingestion, detection and investigation capabilities are severely limited. This topic covers log collection methods, protocols, agent-based and agentless collection, log forwarding architectures, and best practices for ensuring complete and reliable log ingestion.

---

## 1. Log Collection Methods

```
COLLECTION METHODS:

AGENT-BASED COLLECTION:
  → Software installed on source system
  → Reads local logs and forwards to SIEM
  → Rich data collection (process, file, network)
  → Examples: Beats, Splunk UF, Wazuh Agent, NXLog

  Advantages:
  ✓ Detailed endpoint telemetry
  ✓ Reliable delivery (buffering)
  ✓ Can pre-process/filter at source
  ✓ Works across network boundaries
  ✓ Can collect non-syslog data

  Disadvantages:
  ✗ Requires installation on every system
  ✗ Agent management overhead
  ✗ Resource consumption on endpoints
  ✗ Compatibility issues

AGENTLESS COLLECTION:
  → No software on source system
  → Pull via API, WMI, or receive via syslog
  → Network devices, cloud services, legacy systems

  Advantages:
  ✓ No installation required
  ✓ No endpoint resource impact
  ✓ Works with any syslog sender
  ✓ Good for network devices

  Disadvantages:
  ✗ Less detailed data
  ✗ Network dependency
  ✗ May miss events during outages
  ✗ Limited pre-processing

API-BASED COLLECTION:
  → Pull logs via REST API
  → Used for cloud services, SaaS apps
  → Examples: AWS CloudTrail, O365, Azure AD
  → Scheduled polling or webhook-based

  Advantages:
  ✓ Structured data
  ✓ Works with cloud/SaaS
  ✓ No network port requirements
  ✓ Rich metadata

  Disadvantages:
  ✗ API rate limits
  ✗ Polling delays
  ✗ API changes break collection
  ✗ Authentication management
```

---

## 2. Log Collection Protocols

```
SYSLOG:
  → Standard logging protocol (RFC 5424)
  → UDP port 514 (unreliable but fast)
  → TCP port 514 (reliable delivery)
  → TLS port 6514 (encrypted)
  
  Syslog Message Format:
  <PRIORITY>VERSION TIMESTAMP HOSTNAME APP-NAME PROCID MSGID MSG
  
  Example:
  <134>1 2024-01-15T10:30:00Z firewall01 fw - - Connection denied
  src=192.168.1.100 dst=10.0.0.5 port=443 proto=TCP

  Priority = Facility * 8 + Severity
  Severity: 0=Emergency, 1=Alert, 2=Critical ... 7=Debug
  Facility: 0=kern, 1=user, 4=auth, 10=security

WINDOWS EVENT FORWARDING (WEF):
  → Native Windows log collection
  → Source-initiated or collector-initiated
  → Uses WinRM protocol
  → XML-based subscriptions
  → No agent required (built into Windows)
  
  # Configure WEF collector
  wecutil cs subscription.xml
  
  # Subscription XML
  <Subscription>
    <SubscriptionType>SourceInitiated</SubscriptionType>
    <Query>
      <QueryList>
        <Query Path="Security">
          <Select>*[System[(EventID=4624 or EventID=4625)]]</Select>
        </Query>
      </QueryList>
    </Query>
  </Subscription>

BEATS (ELASTIC):
  → Lightweight data shippers
  → Filebeat: Log files
  → Winlogbeat: Windows Event Logs
  → Packetbeat: Network traffic
  → Auditbeat: Audit data
  → Metricbeat: System metrics
  
  # Filebeat configuration
  filebeat.inputs:
    - type: log
      paths:
        - /var/log/syslog
        - /var/log/auth.log
  output.elasticsearch:
    hosts: ["siem:9200"]

CRIBL:
  → Data pipeline / log router
  → Collect, process, route logs
  → Reduce data volume
  → Format conversion
  → Route to multiple destinations
```

---

## 3. Critical Log Sources Configuration

```
WINDOWS LOGGING:

  ESSENTIAL WINDOWS EVENTS:
  Event ID | Description              | Category
  4624     | Successful logon          | Authentication
  4625     | Failed logon              | Authentication
  4648     | Logon with explicit creds | Credential Use
  4672     | Special privileges        | Privilege
  4688     | Process creation          | Execution
  4689     | Process termination       | Execution
  4697     | Service installed         | Persistence
  4698     | Scheduled task created    | Persistence
  4720     | User account created      | Account Mgmt
  4732     | Member added to group     | Account Mgmt
  7045     | New service installed     | Persistence
  1102     | Audit log cleared         | Defense Evasion
  
  SYSMON EVENTS:
  Event ID | Description
  1        | Process creation (with hash, parent)
  3        | Network connection
  7        | Image loaded (DLL)
  8        | CreateRemoteThread
  10       | Process access (credential dumping)
  11       | File created
  12-14    | Registry events
  15       | FileCreateStreamHash (ADS)
  17-18    | Named pipe events
  22       | DNS query
  23       | File delete (archived)
  25       | Process tampering

  ENABLE POWERSHELL LOGGING:
  # Script Block Logging
  # Group Policy: Administrative Templates > 
  # Windows Components > PowerShell > 
  # Turn on Script Block Logging
  
  # Module Logging
  # Logs all PowerShell module executions

LINUX LOGGING:
  Log File           | Content
  /var/log/auth.log  | Authentication events
  /var/log/syslog    | System messages
  /var/log/secure    | Security events (RHEL)
  /var/log/audit/    | Audit framework logs
  /var/log/kern.log  | Kernel messages
  /var/log/cron      | Scheduled tasks
  
  # auditd rules for key events
  -a always,exit -F arch=b64 -S execve -k process_exec
  -w /etc/passwd -p wa -k identity
  -w /etc/shadow -p wa -k identity
  -w /etc/sudoers -p wa -k priv_esc
```

---

## 4. Log Collection Architecture

```
COLLECTION ARCHITECTURE:

SIMPLE (SMALL ENVIRONMENT):
  ┌──────────┐    Syslog     ┌──────┐
  │ Sources  │──────────────▶│ SIEM │
  └──────────┘               └──────┘

INTERMEDIATE (MEDIUM ENVIRONMENT):
  ┌──────────┐    ┌────────────┐    ┌──────┐
  │ Sources  │───▶│ Collector/ │───▶│ SIEM │
  └──────────┘    │ Forwarder  │    └──────┘
                  └────────────┘
  Benefits: Buffering, filtering, load balancing

ENTERPRISE (LARGE ENVIRONMENT):
  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────┐
  │ Sources  │───▶│ Edge     │───▶│ Central  │───▶│ SIEM │
  │ (Site A) │    │ Collector│    │ Log      │    │      │
  └──────────┘    └──────────┘    │ Pipeline │    │      │
  ┌──────────┐    ┌──────────┐    │ (Cribl/  │    │      │
  │ Sources  │───▶│ Edge     │───▶│ Kafka/   │───▶│      │
  │ (Site B) │    │ Collector│    │ Logstash)│    │      │
  └──────────┘    └──────────┘    └──────────┘    └──────┘
  
  Benefits:
  → Redundancy and buffering
  → Data enrichment at pipeline
  → Volume reduction (filtering)
  → Multi-destination routing
  → Format normalization

HIGH AVAILABILITY:
  → Redundant collectors
  → Clustered SIEM
  → Persistent queues (no data loss)
  → Geographic distribution
  → Automatic failover

LOG PIPELINE TOOLS:
  Tool         | Type       | Key Feature
  Logstash     | Open source| Elastic ecosystem
  Fluentd      | Open source| Cloud-native logging
  Cribl Stream | Commercial | Data routing/reduction
  Kafka        | Open source| Message queue (buffering)
  Vector       | Open source| High performance
```

---

## 5. Best Practices

```
LOG COLLECTION BEST PRACTICES:

COVERAGE:
  [ ] All critical systems sending logs
  [ ] Network devices (firewalls, switches)
  [ ] Authentication systems (AD, SSO, MFA)
  [ ] Endpoint (Sysmon, EDR)
  [ ] Cloud environments
  [ ] Email security
  [ ] DNS servers
  [ ] Web proxies
  [ ] Database servers
  [ ] Custom applications

RELIABILITY:
  → Use TCP/TLS for syslog (not UDP)
  → Enable persistent queues
  → Monitor log source health
  → Alert on missing log sources
  → Test failover procedures
  → Document all log sources

OPTIMIZATION:
  → Filter noise at source when possible
  → Compress data in transit
  → Use efficient formats (JSON over text)
  → Implement data tiering (hot/warm/cold)
  → Archive to cost-effective storage
  → Set retention per compliance requirements

MONITORING LOG HEALTH:
  Metric                     | Alert Condition
  Log source count           | Drops below expected
  EPS from source            | Drops > 50% from baseline
  Last event received        | > 1 hour gap
  Parse failures             | > 5% of events
  Queue depth                | > threshold
  Collector CPU/memory       | > 80%

RETENTION GUIDELINES:
  Data Type         | Minimum Retention
  Security events   | 1 year
  Authentication    | 1 year
  Network logs      | 90 days
  Application logs  | 90 days
  Full PCAP         | 30 days
  Compliance data   | Per regulation (1-7 years)
```

---

## Summary Table

| Collection Method | Protocol | Best For | Reliability |
|------------------|----------|----------|-------------|
| Agent-based | Various | Endpoints, detailed data | High |
| Syslog | UDP/TCP/TLS | Network devices | Medium-High |
| WEF | WinRM | Windows environments | High |
| Beats | HTTP/HTTPS | Elastic ecosystem | High |
| API | REST/Webhook | Cloud, SaaS | Medium |
| Pipeline | Kafka/Cribl | Enterprise routing | Very High |

---

## Revision Questions

1. What are the differences between agent-based and agentless log collection?
2. Why should TCP/TLS be preferred over UDP for syslog?
3. What Windows Event IDs are most critical for security monitoring?
4. How does a log pipeline improve enterprise log collection?
5. What metrics should be monitored for log source health?

---

*Previous: [01-siem-concepts.md](01-siem-concepts.md) | Next: [03-normalization-and-parsing.md](03-normalization-and-parsing.md)*

---

*[Back to README](../README.md)*
