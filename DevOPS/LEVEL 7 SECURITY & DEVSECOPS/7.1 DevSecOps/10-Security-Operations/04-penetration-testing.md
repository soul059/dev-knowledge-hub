# Chapter 10.4: Penetration Testing

## Overview

Penetration testing (pentesting) is the authorized simulation of real-world attacks against systems, applications, and infrastructure to identify exploitable vulnerabilities before adversaries do. In DevSecOps, pentesting is integrated into the SDLC — from automated testing in CI/CD to periodic manual assessments. This chapter covers pentesting methodologies, tools, and integration patterns.

---

## Penetration Testing Types

```
PENTEST TYPES:

  ┌──────────────────────────────────────────────────────┐
  │                    KNOWLEDGE LEVELS                    │
  ├──────────────────┬─────────────────┬─────────────────┤
  │   BLACK BOX      │   GREY BOX      │   WHITE BOX     │
  │                  │                 │                 │
  │  No internal     │  Partial info   │  Full access    │
  │  knowledge       │  (some docs,    │  (source code,  │
  │                  │  credentials)   │  architecture)  │
  │                  │                 │                 │
  │  Simulates:      │  Simulates:     │  Simulates:     │
  │  External        │  Insider with   │  Code review +  │
  │  attacker        │  limited access │  exploitation   │
  │                  │                 │                 │
  │  Most realistic  │  Most balanced  │  Most thorough  │
  │  Least coverage  │  Good coverage  │  Best coverage  │
  └──────────────────┴─────────────────┴─────────────────┘

  ┌──────────────────────────────────────────────────────┐
  │                    SCOPE TYPES                         │
  ├────────────────┬─────────────────┬───────────────────┤
  │  NETWORK       │  WEB APP        │  CLOUD / INFRA    │
  │                │                 │                   │
  │ • External     │ • OWASP Top 10  │ • AWS/Azure/GCP  │
  │ • Internal     │ • API testing   │ • K8s clusters   │
  │ • Wireless     │ • Auth bypass   │ • IaC misconfig  │
  │ • IoT          │ • Business logic│ • Container esc  │
  └────────────────┴─────────────────┴───────────────────┘
```

---

## Pentest Methodology (PTES)

```
PENETRATION TESTING EXECUTION STANDARD:

  ┌──────────────┐
  │ 1. PRE-       │
  │ ENGAGEMENT   │  Scope, rules of engagement,
  │              │  authorization, emergency contacts
  └──────┬───────┘
         │
         ▼
  ┌──────────────┐
  │ 2. INTEL     │  OSINT, DNS recon, tech stack
  │ GATHERING    │  fingerprinting, subdomain enum
  └──────┬───────┘
         │
         ▼
  ┌──────────────┐
  │ 3. THREAT    │  Identify attack vectors,
  │ MODELING     │  prioritize targets, map
  └──────┬───────┘  attack surface
         │
         ▼
  ┌──────────────┐
  │ 4. VULN      │  Automated scanning + manual
  │ ANALYSIS     │  testing, CVE research
  └──────┬───────┘
         │
         ▼
  ┌──────────────┐
  │ 5. EXPLOIT   │  Exploit vulnerabilities,
  │              │  gain access, prove impact
  └──────┬───────┘
         │
         ▼
  ┌──────────────┐
  │ 6. POST-     │  Privilege escalation,
  │ EXPLOIT      │  lateral movement, data access
  └──────┬───────┘
         │
         ▼
  ┌──────────────┐
  │ 7. REPORTING │  Findings, severity, evidence,
  │              │  remediation recommendations
  └──────────────┘
```

---

## Reconnaissance Tools

```bash
# --- Subdomain Enumeration ---
# Subfinder
subfinder -d example.com -o subdomains.txt

# Amass
amass enum -d example.com -o amass-results.txt

# --- Port Scanning ---
# Nmap — service detection
nmap -sV -sC -oA scan-results target.example.com

# Nmap — comprehensive scan
nmap -p- -A -T4 --script=vuln target.example.com

# --- Web Technology Detection ---
# Wappalyzer / whatweb
whatweb https://example.com

# --- Directory/File Discovery ---
# Gobuster
gobuster dir -u https://example.com \
    -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
    -t 50 -o dirs.txt

# ffuf (faster)
ffuf -u https://example.com/FUZZ \
    -w /usr/share/wordlists/common.txt \
    -mc 200,301,302
```

---

## Web Application Testing

```bash
# --- OWASP ZAP (automated + manual) ---
# Automated scan
zap-cli quick-scan --self-contained \
    --start-options '-config api.disablekey=true' \
    https://target.example.com

# API scan with OpenAPI spec
zap-cli openapi-scan \
    https://target.example.com/openapi.json

# --- SQLMap (SQL injection) ---
# Test parameter for SQLi
sqlmap -u "https://target.example.com/search?q=test" \
    --batch --level=3 --risk=2

# With authentication
sqlmap -u "https://target.example.com/api/users?id=1" \
    --cookie="session=abc123" \
    --dbs  # enumerate databases

# --- Nuclei (template-based scanning) ---
# Run all templates
nuclei -u https://target.example.com -o results.txt

# Specific severity
nuclei -u https://target.example.com \
    -severity critical,high

# Custom template
nuclei -u https://target.example.com \
    -t custom-templates/
```

---

## Infrastructure & Cloud Pentesting

```bash
# --- Kubernetes Pentesting ---
# kube-hunter — K8s penetration testing
kube-hunter --remote target-cluster.example.com
kube-hunter --pod  # run from within cluster

# Checks:
# • API server exposure
# • Dashboard access
# • etcd access
# • kubelet API access
# • Pod escape paths

# --- AWS Pentesting ---
# Pacu — AWS exploitation framework
pacu

# Within Pacu:
# > set_keys  (configure AWS creds)
# > run iam__enum_permissions
# > run iam__privesc_scan
# > run ec2__enum
# > run s3__bucket_finder
# > run lambda__enum

# ScoutSuite — multi-cloud audit
scout aws --report-dir ./scout-report

# --- Container Escape Testing ---
# CDK — Container escape toolkit
cdk evaluate  # assess possible escapes

# deepce — Docker enumeration & escape
deepce.sh  # run inside container
```

---

## Automated Pentesting in CI/CD

```yaml
# .github/workflows/security-pentest.yml
name: Automated Security Testing
on:
  schedule:
    - cron: '0 2 * * 1'  # Weekly Monday 2am
  workflow_dispatch:       # Manual trigger

jobs:
  dast-scan:
    runs-on: ubuntu-latest
    services:
      webapp:
        image: myapp:${{ github.sha }}
        ports:
          - 8080:8080
    steps:
      - uses: actions/checkout@v4

      - name: Wait for app startup
        run: |
          timeout 60 bash -c 'until curl -s http://localhost:8080/health; do sleep 2; done'

      # OWASP ZAP Full Scan
      - name: ZAP Full Scan
        uses: zaproxy/action-full-scan@v0.10.0
        with:
          target: 'http://localhost:8080'
          rules_file_name: '.zap/rules.tsv'
          cmd_options: '-a'

      # Nuclei Scan
      - name: Nuclei Scan
        uses: projectdiscovery/nuclei-action@main
        with:
          target: http://localhost:8080
          flags: "-severity critical,high -stats"
          output: nuclei-results.txt

      # Upload results
      - name: Upload Reports
        uses: actions/upload-artifact@v4
        with:
          name: pentest-reports
          path: |
            report_html.html
            nuclei-results.txt

  k8s-pentest:
    runs-on: ubuntu-latest
    steps:
      - name: kube-hunter Scan
        run: |
          pip install kube-hunter
          kube-hunter --remote ${{ secrets.K8S_CLUSTER_URL }} \
            --report json > kube-hunter-results.json
```

---

## Pentest Report Structure

```
PENETRATION TEST REPORT:

  ┌──────────────────────────────────────────────────────┐
  │ EXECUTIVE SUMMARY                                      │
  │ • Overall risk rating: CRITICAL / HIGH / MEDIUM / LOW │
  │ • Key findings count by severity                      │
  │ • Business impact summary                             │
  │ • Top 3 recommendations                               │
  ├──────────────────────────────────────────────────────┤
  │ METHODOLOGY                                            │
  │ • Scope and rules of engagement                       │
  │ • Testing timeline                                     │
  │ • Tools and techniques used                           │
  ├──────────────────────────────────────────────────────┤
  │ FINDINGS (per finding):                                │
  │ ┌────────────────────────────────────────────────────┐│
  │ │ Title: SQL Injection in Search API                 ││
  │ │ Severity: CRITICAL (CVSS 9.8)                     ││
  │ │ Affected: /api/v1/search?q=                       ││
  │ │ Description: Parameterized queries not used       ││
  │ │ Evidence: [screenshots, HTTP requests/responses]  ││
  │ │ Impact: Full database read/write access           ││
  │ │ Remediation: Use prepared statements/ORM         ││
  │ │ References: CWE-89, OWASP A03:2021               ││
  │ └────────────────────────────────────────────────────┘│
  ├──────────────────────────────────────────────────────┤
  │ APPENDIX                                               │
  │ • Full tool output                                     │
  │ • Request/response captures                           │
  │ • Network diagrams                                     │
  └──────────────────────────────────────────────────────┘
```

---

## Red Team vs Blue Team vs Purple Team

```
TEAM OPERATIONS:

  ┌─────────────┐         ┌─────────────┐
  │  RED TEAM   │ ◀─────▶ │  BLUE TEAM  │
  │             │ attack  │             │
  │ • Offensive │ vs      │ • Defensive │
  │ • Simulate  │ defend  │ • Detect    │
  │   real      │         │   attacks   │
  │   attacks   │         │ • Respond   │
  │ • Test      │         │ • Improve   │
  │   defenses  │         │   controls  │
  └──────┬──────┘         └──────┬──────┘
         │                       │
         └──────────┬────────────┘
                    │
                    ▼
           ┌─────────────┐
           │ PURPLE TEAM │
           │             │
           │ Collaborative:
           │ Red attacks,
           │ Blue observes,
           │ Both improve
           │ detections
           │ together
           └─────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Pentest causes production outage | Aggressive scanning against production | Always test in staging; use rate limits; have rollback plan; exclude DoS tests unless approved |
| Too many low-severity findings | Automated scanners report everything | Focus report on exploitable vulnerabilities; rate by actual business impact not just CVSS |
| Findings not remediated | No ownership or deadline assigned | Assign findings to specific teams; set SLA by severity; track in issue tracker |
| Cloud provider blocks scanning | AWS/Azure/GCP pentest policies | Register pentest with cloud provider; use approved scope; check provider's pentest policy |
| Retest shows same vulnerabilities | Fixes incomplete or regression | Integrate fix verification into CI/CD; add regression tests; schedule periodic retests |

---

## Summary Table

| Testing Type | Frequency | Automation | Purpose |
|-------------|-----------|-----------|---------|
| **Automated DAST** | Every build / weekly | Full | Baseline web vulnerability scanning |
| **Nuclei templates** | Weekly / on deploy | Full | Known vulnerability / misconfiguration |
| **Manual web pentest** | Quarterly / annually | None | Business logic, complex attacks |
| **Infrastructure pentest** | Annually | Partial | Network, cloud, K8s security |
| **Red team exercise** | Annually | None | Full attack simulation, test response |
| **Bug bounty** | Continuous | N/A | Crowdsourced vulnerability discovery |

---

## Quick Revision Questions

1. **What is the difference between black box, grey box, and white box testing?**
   > Black box: tester has no internal knowledge, simulating an external attacker. Grey box: tester has partial information (e.g., user credentials, API docs), simulating an insider or authenticated attacker. White box: tester has full access to source code, architecture, and documentation, enabling the most thorough testing. Grey box offers the best balance of realism and coverage.

2. **What are the phases of a penetration test?**
   > Pre-engagement (scope, authorization), Intelligence Gathering (reconnaissance, OSINT), Threat Modeling (identify attack vectors), Vulnerability Analysis (scanning + manual testing), Exploitation (prove vulnerabilities are exploitable), Post-Exploitation (privilege escalation, lateral movement), and Reporting (findings, evidence, remediation advice).

3. **How should pentesting be integrated into DevSecOps?**
   > Automated DAST scanning (ZAP, Nuclei) runs in CI/CD on every build or weekly. Manual pentests are conducted quarterly or before major releases. Results feed into the development backlog with severity-based SLAs. Regression tests ensure fixes persist. Red team exercises annually test the full security program.

4. **What is the difference between Red Team, Blue Team, and Purple Team?**
   > Red Team simulates real attacks to test defenses. Blue Team detects and responds to attacks (SOC, IR). Purple Team combines both — Red attacks while Blue observes, then they collaborate to improve detections. Purple Team exercises are most effective for improving detection coverage because both sides learn together.

5. **What should a penetration test report include?**
   > Executive summary (overall risk, key findings, top recommendations), methodology (scope, timeline, tools), detailed findings (title, severity/CVSS, affected resource, evidence/screenshots, business impact, remediation steps, references to CWE/OWASP), and appendix (full tool output, request/response captures). Each finding must include proof of exploitation.

---

[← Previous: 10.3 Incident Response](03-incident-response.md) | [Next: 10.5 Bug Bounty Programs →](05-bug-bounty-programs.md)

[Back to Table of Contents](../README.md)
