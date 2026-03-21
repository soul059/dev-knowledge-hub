# Anomaly Detection

## Unit 7 - Topic 6 | Network Traffic Analysis

---

## Overview

**Anomaly detection** identifies network behavior that deviates from established baselines. Unlike signature-based detection (which looks for known attacks), anomaly detection can discover **zero-day attacks**, **insider threats**, and **novel attack techniques** — but it also generates more false positives.

---

## 1. Signature-Based vs Anomaly-Based Detection

| Feature | Signature-Based | Anomaly-Based |
|---------|:-:|:-:|
| **Detects known attacks** | ✅ Excellent | ✅ Can detect |
| **Detects unknown attacks** | ❌ No | ✅ Yes (zero-days) |
| **False positives** | Low | Higher |
| **Requires baseline** | No | Yes |
| **Updates needed** | Signature database | Baseline retraining |
| **Examples** | Snort, Suricata rules | ML-based IDS, Darktrace |

---

## 2. Types of Anomalies

| Type | Description | Example |
|------|-------------|---------|
| **Volume** | Unusual traffic volume | Sudden bandwidth spike (DDoS) |
| **Protocol** | Unexpected protocol usage | DNS on non-standard port |
| **Behavioral** | Unusual user/host behavior | User accessing 100 servers at 2 AM |
| **Geographic** | Connections from unusual locations | Login from new country |
| **Temporal** | Activity at unusual times | Database queries at 3 AM |
| **Statistical** | Values outside normal distribution | 10x average DNS queries |

---

## 3. Anomaly Detection Methods

```
┌──────────────────────────────────────────────────────────────┐
│              DETECTION METHODS                                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  STATISTICAL:                                                │
│  Set thresholds (mean ± 2 standard deviations)               │
│  Alert when traffic exceeds thresholds                       │
│                                                              │
│  MACHINE LEARNING:                                           │
│  Train model on normal behavior → detect deviations          │
│  Clustering, neural networks, decision trees                 │
│                                                              │
│  RULE-BASED:                                                 │
│  "If DNS queries > 500/min AND destination is external       │
│   AND query length > 50 chars → ALERT: DNS Tunneling"       │
│                                                              │
│  BEHAVIORAL ANALYSIS (UEBA):                                 │
│  Build profiles for each user/entity                         │
│  Alert when behavior deviates from their profile             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. Detection Tools

| Tool | Type | Capability |
|------|------|-----------|
| **Zeek + RITA** | Open source | Beaconing, DNS tunneling, long connections |
| **Suricata** | Open source | Signature + anomaly detection |
| **Darktrace** | Commercial | AI-based network anomaly detection |
| **Vectra AI** | Commercial | AI-based threat detection |
| **Elastic SIEM** | Open source | Log-based anomaly detection |
| **Splunk UBA** | Commercial | User and Entity Behavior Analytics |

---

## 5. Reducing False Positives

| Strategy | Description |
|----------|-------------|
| **Tune thresholds** | Adjust sensitivity based on environment |
| **Whitelist known behavior** | Exclude legitimate patterns |
| **Correlate events** | Combine multiple indicators before alerting |
| **Context enrichment** | Add user, asset, and threat intel context |
| **Continuous learning** | Update baselines as environment changes |
| **Tiered alerting** | Low/Medium/High severity based on confidence |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Anomaly Detection** | Identifies deviations from normal behavior |
| **Advantage** | Can detect zero-day and unknown attacks |
| **Challenge** | Higher false positive rate than signatures |
| **Methods** | Statistical, ML, rule-based, UEBA |
| **Key** | Requires solid baseline to be effective |
| **Tools** | Zeek/RITA, Darktrace, Vectra, Suricata |

---

## Quick Revision Questions

1. **What is the key difference between signature and anomaly detection?**
2. **Why can anomaly detection find zero-day attacks?**
3. **What types of anomalies can be detected?**
4. **Why does anomaly detection have more false positives?**
5. **Name 3 strategies for reducing false positives.**

---

[← Previous: Baseline Establishment](05-baseline-establishment.md)

---

*Unit 7 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Network Security Devices →](../08-Network-Security-Devices/01-firewalls.md)*
