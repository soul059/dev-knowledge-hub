# SIEM Integration

## Unit 3 - Topic 6 | Linux Logging and Auditing

---

## Overview

A **SIEM (Security Information and Event Management)** system collects, normalizes, correlates, and analyzes security events from across the infrastructure. Integrating Linux systems with a SIEM enables **real-time threat detection**, **compliance reporting**, and **incident investigation** at scale.

---

## 1. SIEM Architecture

```
┌──────────────────────────────────────────────────────────────┐
│              SIEM INTEGRATION                                 │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  DATA SOURCES:                                               │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────┐         │
│  │ Linux    │ │ Windows  │ │ Firewall │ │ Cloud  │         │
│  │ Servers  │ │ Servers  │ │ IDS/IPS  │ │ (AWS)  │         │
│  └────┬─────┘ └────┬─────┘ └────┬─────┘ └───┬────┘         │
│       │            │            │            │               │
│       └────────────┴────────────┴────────────┘               │
│                         │                                    │
│                    ┌────▼────┐                               │
│                    │COLLECTOR│ (Beats, Fluentd, agents)      │
│                    └────┬────┘                               │
│                    ┌────▼────┐                               │
│                    │ PARSER  │ (Normalize, enrich)           │
│                    └────┬────┘                               │
│                    ┌────▼────┐                               │
│                    │ INDEX   │ (Elasticsearch, Splunk)       │
│                    └────┬────┘                               │
│                    ┌────▼────┐                               │
│                    │DASHBOARD│ (Kibana, Grafana)             │
│                    │ ALERTS  │ (Rules, ML, correlation)     │
│                    └─────────┘                               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Popular SIEM Solutions

| SIEM | Type | Strengths |
|------|------|----------|
| **Elastic SIEM (ELK)** | Open source | Free, flexible, Elastic Agent |
| **Splunk** | Commercial | Powerful search (SPL), mature |
| **Wazuh** | Open source | HIDS + SIEM, free |
| **Graylog** | Open source | Log management focused |
| **Microsoft Sentinel** | Cloud | Azure integration |
| **IBM QRadar** | Commercial | Enterprise, compliance |
| **Sumo Logic** | Cloud | Cloud-native analytics |

---

## 3. Elastic Stack (ELK) Integration

```bash
# INSTALL FILEBEAT (Log Shipper) on Linux server
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.x-amd64.deb
dpkg -i filebeat-8.x-amd64.deb

# Configure: /etc/filebeat/filebeat.yml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/auth.log
    - /var/log/syslog
    - /var/log/audit/audit.log
  tags: ["linux", "security"]

output.elasticsearch:
  hosts: ["https://elastic-server:9200"]
  username: "elastic"
  password: "changeme"

# Enable built-in modules
filebeat modules enable system        # auth.log, syslog
filebeat modules enable auditd        # audit.log

# Start
systemctl enable --now filebeat

# INSTALL AUDITBEAT (Audit-specific shipper)
# Replaces auditd with direct Elasticsearch shipping
dpkg -i auditbeat-8.x-amd64.deb
systemctl enable --now auditbeat
```

---

## 4. Wazuh Integration

```bash
# Wazuh = OSSEC fork + SIEM + HIDS (all-in-one, free)

# Install Wazuh Agent on Linux
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | apt-key add -
echo "deb https://packages.wazuh.com/4.x/apt/ stable main" >> /etc/apt/sources.list.d/wazuh.list
apt update && apt install wazuh-agent

# Configure: /var/ossec/etc/ossec.conf
<ossec_config>
  <client>
    <server>
      <address>wazuh-server.example.com</address>
    </server>
  </client>
  <syscheck>
    <directories check_all="yes">/etc,/usr/bin,/usr/sbin</directories>
  </syscheck>
  <localfile>
    <log_format>syslog</log_format>
    <location>/var/log/auth.log</location>
  </localfile>
</ossec_config>

systemctl enable --now wazuh-agent
```

---

## 5. What to Send to SIEM

| Data Source | Why |
|-------------|-----|
| **auth.log / secure** | Authentication events, brute force |
| **syslog** | System events, service status |
| **audit.log** | File access, syscalls, privilege escalation |
| **Web server logs** | Web attacks, SQLi, XSS |
| **Application logs** | Business logic attacks |
| **Firewall logs** | Connection attempts, blocks |
| **DNS logs** | DNS tunneling, C2 detection |
| **Process creation** | Malware execution, living-off-the-land |

---

## 6. SIEM Detection Rules

```yaml
# Example: Detect SSH brute force
# Elastic SIEM detection rule
name: "SSH Brute Force Detected"
description: "More than 10 failed SSH logins from same IP in 5 minutes"
type: threshold
query: 'event.action:"ssh_login" AND event.outcome:"failure"'
threshold:
  field: source.ip
  value: 10
  time_window: 5m
severity: high
actions:
  - alert
  - create_incident

# Example: Detect new user creation
name: "New User Account Created"
description: "New user account created on Linux system"
query: 'process.name:"useradd" OR process.name:"adduser"'
severity: medium
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **SIEM** | Centralized security event management |
| **ELK Stack** | Elasticsearch + Logstash + Kibana (free) |
| **Wazuh** | Open-source HIDS + SIEM |
| **Filebeat** | Log shipper for Elasticsearch |
| **Detection rules** | Automated alerting on suspicious patterns |
| **Key logs** | auth, syslog, audit, web, firewall, DNS |

---

## Quick Revision Questions

1. **What does SIEM stand for and what does it do?**
2. **Name 3 open-source SIEM solutions.**
3. **What is Filebeat and how does it integrate with ELK?**
4. **What Linux logs should be forwarded to a SIEM?**
5. **How would you write a SIEM rule to detect brute force attacks?**

---

[← Previous: Log Forwarding](05-log-forwarding.md)

---

*Unit 3 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Linux File System Security →](../04-Linux-File-System-Security/01-file-system-hierarchy.md)*
