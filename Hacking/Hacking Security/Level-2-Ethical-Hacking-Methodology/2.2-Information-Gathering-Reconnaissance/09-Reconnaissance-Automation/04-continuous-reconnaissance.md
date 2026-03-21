# Continuous Reconnaissance

## Unit 9 - Topic 4 | Reconnaissance Automation

---

## Overview

**Continuous reconnaissance** monitors the target's attack surface over time, detecting changes like new subdomains, open ports, deployed services, and expired certificates. Unlike point-in-time scanning, continuous recon provides ongoing visibility into infrastructure changes.

---

## 1. Why Continuous Recon?

```
POINT-IN-TIME vs CONTINUOUS:

POINT-IN-TIME RECON (Traditional):
├── Scan once at start of engagement
├── Miss anything added after scan
├── Snapshot becomes stale quickly
└── New servers, subdomains, services missed

CONTINUOUS RECON:
├── Monitor changes over time
├── Detect new attack surface immediately
├── Track infrastructure evolution
├── Alert on security-relevant changes
└── Provides ongoing intelligence

WHAT CHANGES OVER TIME:
├── New subdomains created
├── New servers deployed
├── Ports opened/closed
├── SSL certificates issued/expired
├── DNS records modified
├── Cloud resources provisioned
├── New web applications launched
├── Services updated or deprecated
└── Security configurations changed
```

---

## 2. Continuous Monitoring Setup

```bash
#!/bin/bash
# === CONTINUOUS SUBDOMAIN MONITORING ===
# Run daily via cron job

TARGET="target.com"
PREV_RESULTS="results/${TARGET}/subdomains_prev.txt"
CURR_RESULTS="results/${TARGET}/subdomains_curr.txt"
DIFF_FILE="results/${TARGET}/new_subdomains.txt"

# Enumerate current subdomains
subfinder -d ${TARGET} -silent | sort -u > ${CURR_RESULTS}

# Compare with previous results
if [ -f "${PREV_RESULTS}" ]; then
    comm -13 ${PREV_RESULTS} ${CURR_RESULTS} > ${DIFF_FILE}
    NEW_COUNT=$(wc -l < ${DIFF_FILE})
    
    if [ ${NEW_COUNT} -gt 0 ]; then
        echo "[!] ${NEW_COUNT} NEW subdomains found!"
        cat ${DIFF_FILE}
        # Send alert (email, Slack, Discord)
        # curl -X POST "https://hooks.slack.com/..." \
        #   -d "{\"text\": \"New subdomains: $(cat ${DIFF_FILE})\"}"
    fi
fi

# Save current as previous for next run
cp ${CURR_RESULTS} ${PREV_RESULTS}

# === CRON JOB ===
# Run daily at 2 AM:
# crontab -e
# 0 2 * * * /opt/recon/monitor.sh >> /var/log/recon.log 2>&1
```

---

## 3. Certificate Transparency Monitoring

```bash
# === CT LOG MONITORING ===
# Certificate Transparency logs record ALL SSL certs
# New cert issued = new subdomain likely!

# Certspotter (API):
curl "https://api.certspotter.com/v1/issuances?domain=target.com&include_subdomains=true"

# Sublert — monitor CT logs for new subdomains
# sublert.py -u target.com

# crt.sh continuous monitoring:
curl "https://crt.sh/?q=%.target.com&output=json" | \
  jq -r '.[].name_value' | sort -u > ct_subdomains.txt

# === WHAT CT LOGS REVEAL ===
# ├── New subdomains being activated
# ├── Internal subdomains accidentally exposed
# ├── Staging/development environments
# ├── Certificate misconfigurations
# └── Wildcard certificate usage
```

---

## 4. Monitoring Tools

| Tool | Monitors | Alert Method |
|------|---------|:---:|
| **Amass Track** | Subdomain changes | CLI output |
| **Sublert** | CT log new certs | Email, Slack |
| **Shodan Monitor** | Port/service changes | Email, webhook |
| **Censys ASM** | Full attack surface | Dashboard |
| **SpiderFoot HX** | Continuous OSINT | Web UI |
| **Nuclei** | New vulnerabilities | CLI, webhook |
| **ProjectDiscovery Cloud** | Full pipeline | Dashboard |

```bash
# === AMASS TRACK ===
amass track -d target.com
# Shows subdomains added/removed since last scan

# === SHODAN MONITOR ===
shodan alert create "Target Monitoring" 203.0.113.0/24
# Alerts on new ports, services, vulnerabilities

# === NUCLEI CONTINUOUS ===
# Re-scan periodically for new vulns
nuclei -l alive_hosts.txt -severity critical,high \
  -new-templates-version
# Checks against newly released templates
```

---

## 5. Continuous Recon Architecture

```
CONTINUOUS RECON PIPELINE:

SCHEDULED SCANS (Daily/Weekly):
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ Subdomain    │───→│ DNS          │───→│ HTTP         │
│ Enumeration  │    │ Resolution   │    │ Probing      │
└──────────────┘    └──────────────┘    └──────────────┘
                                              │
                    ┌──────────────┐           │
                    │ Port         │←──────────┘
                    │ Scanning     │
                    └──────┬───────┘
                           │
                    ┌──────▼───────┐
                    │ Diff Engine  │
                    │ (New vs Old) │
                    └──────┬───────┘
                           │
              ┌────────────┴────────────┐
              │                         │
        ┌─────▼─────┐           ┌──────▼──────┐
        │ ALERT      │           │ DATABASE     │
        │ Slack/Email │           │ Store results│
        └────────────┘           └─────────────┘

EVENT-DRIVEN (Real-time):
├── CT Log monitoring → new certificate → alert
├── Shodan Monitor → new port open → alert
├── DNS monitoring → record changed → alert
└── GitHub monitoring → new repo/code → alert
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Continuous recon** | Ongoing monitoring vs one-time scan |
| **Subdomain monitoring** | Detect new subdomains daily |
| **CT logs** | Real-time certificate issuance tracking |
| **Diff engine** | Compare current vs previous results |
| **Alerting** | Slack, email, webhook notifications |
| **Automation** | Cron jobs, scheduled pipelines |

---

## Quick Revision Questions

1. **Why is continuous recon better than point-in-time scanning?**
2. **How do Certificate Transparency logs help monitoring?**
3. **What tool tracks subdomain changes over time?**
4. **How do you set up automated recon with cron?**
5. **What events should trigger continuous recon alerts?**

---

[← Previous: API Integration](03-api-integration.md) | [Next: Data Correlation →](05-data-correlation.md)

---

*Unit 9 - Topic 4 of 5 | [Back to README](../README.md)*
