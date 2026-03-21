# Baseline Establishment

## Unit 7 - Topic 5 | Network Traffic Analysis

---

## Overview

A **network baseline** is a documented record of normal network behavior — traffic volumes, protocols, connections, and timing patterns. Without a baseline, you can't detect anomalies. Baselines are the foundation of **anomaly-based detection**, allowing security teams to identify deviations that may indicate attacks.

---

## 1. What to Baseline

| Metric | What to Measure |
|--------|----------------|
| **Bandwidth** | Average and peak throughput per link/segment |
| **Protocol Distribution** | % of HTTP, DNS, SMB, etc. traffic |
| **Top Talkers** | Hosts generating most traffic |
| **Connection Patterns** | Who connects to whom, how often |
| **DNS Query Volume** | Normal query rate per host |
| **Authentication Events** | Login frequency, success/failure ratios |
| **Time Patterns** | Business hours vs off-hours traffic |
| **Port Usage** | Which ports are normally in use |
| **External Connections** | Normal outbound destinations |

---

## 2. Baseline Process

```
┌──────────────────────────────────────────────────────────────┐
│              BASELINE ESTABLISHMENT                           │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. COLLECT                                                  │
│     Capture traffic data over 2-4 weeks minimum              │
│     Include weekdays, weekends, month-end, peaks             │
│                                                              │
│  2. ANALYZE                                                  │
│     Calculate averages, peaks, and standard deviations       │
│     Identify normal patterns and expected ranges             │
│                                                              │
│  3. DOCUMENT                                                 │
│     Record baseline metrics and acceptable thresholds        │
│     Note seasonal variations (holidays, events)              │
│                                                              │
│  4. MONITOR                                                  │
│     Compare live traffic against baseline continuously       │
│     Alert when metrics exceed thresholds                     │
│                                                              │
│  5. UPDATE                                                   │
│     Re-baseline periodically (quarterly or after changes)    │
│     Account for business growth and new applications         │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3. Baseline Tools

| Tool | Use |
|------|-----|
| **NetFlow/sFlow/IPFIX** | Flow-based traffic statistics |
| **PRTG / Nagios / Zabbix** | Network monitoring and graphing |
| **MRTG / Cacti** | SNMP-based traffic graphing |
| **Zeek** | Connection logs for behavior analysis |
| **Wireshark Statistics** | Protocol hierarchy, conversations |
| **SIEM** | Correlate and baseline security events |
| **Elastic Stack (ELK)** | Log aggregation and visualization |

---

## 4. Deviation Alerts

| Normal Baseline | Deviation | Possible Cause |
|----------------|-----------|----------------|
| 100 Mbps average | Spike to 500 Mbps | DDoS, data exfil, backup |
| 95% HTTP/HTTPS | Sudden 30% DNS | DNS tunneling |
| 200 DNS queries/min | 5000 queries/min | DNS amplification/tunnel |
| No traffic after 8 PM | Large transfers at 2 AM | Data exfiltration |
| User connects to 10 hosts | User connects to 500 hosts | Lateral movement/scan |
| 0 failed logins/hour | 1000 failed logins/hour | Brute force attack |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Baseline** | Documented normal network behavior |
| **Duration** | 2-4 weeks minimum to establish |
| **Purpose** | Detect anomalies by comparing to normal |
| **Key Metrics** | Bandwidth, protocols, connections, timing |
| **Update** | Re-baseline quarterly or after changes |
| **Tools** | NetFlow, PRTG, Zeek, SIEM |

---

## Quick Revision Questions

1. **What is a network baseline and why is it important?**
2. **How long should you collect data to establish a baseline?**
3. **Name 5 metrics that should be included in a baseline.**
4. **What tools can be used for baseline establishment?**
5. **When should a baseline be updated?**

---

[← Previous: Traffic Patterns](04-traffic-patterns.md) | [Next: Anomaly Detection →](06-anomaly-detection.md)

---

*Unit 7 - Topic 5 of 6 | [Back to README](../README.md)*
