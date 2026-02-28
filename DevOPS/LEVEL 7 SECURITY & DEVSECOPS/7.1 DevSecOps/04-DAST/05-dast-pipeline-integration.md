# Chapter 4.5: DAST Pipeline Integration

## Overview

This chapter brings together DAST concepts into a production-grade pipeline integration strategy, covering environment management, result aggregation, quality gates, and reporting workflows.

---

## End-to-End DAST Pipeline

```
PRODUCTION-GRADE DAST PIPELINE:

  ┌────────┐   ┌────────┐   ┌──────────┐   ┌──────────────┐
  │ Code   │──→│ Build  │──→│ Deploy   │──→│ Health Check │
  │ Merge  │   │ + SAST │   │ Staging  │   │ Wait Ready   │
  └────────┘   └────────┘   └──────────┘   └──────┬───────┘
                                                    │
                          ┌─────────────────────────┤
                          │                         │
                     ┌────┴─────┐             ┌────┴─────┐
                     │ Nuclei   │             │ ZAP      │
                     │ Fast     │             │ Baseline │
                     │ (2 min)  │             │ (5 min)  │
                     └────┬─────┘             └────┬─────┘
                          │                         │
                          └────────┬────────────────┘
                                   │
                              ┌────┴─────┐
                              │ Aggregate│
                              │ Results  │
                              └────┬─────┘
                                   │
                              ┌────┴─────┐
                              │ Quality  │──FAIL──→ Block + Alert
                              │ Gate     │
                              └────┬─────┘
                                   │ PASS
                              ┌────┴─────┐
                              │ Deploy   │
                              │ to Prod  │
                              └──────────┘
```

---

## Complete GitHub Actions Pipeline

```yaml
name: Full Security Pipeline
on:
  push:
    branches: [main]

env:
  STAGING_URL: https://staging.myapp.com

jobs:
  # Stage 1: Build and SAST
  build-and-sast:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build
        run: docker build -t myapp:${{ github.sha }} .
      - name: Push to Registry
        run: |
          docker tag myapp:${{ github.sha }} registry.company.com/myapp:staging
          docker push registry.company.com/myapp:staging
      - name: SAST Scan
        uses: returntocorp/semgrep-action@v1
        with:
          config: p/security-audit

  # Stage 2: Deploy to Staging
  deploy-staging:
    needs: build-and-sast
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Staging
        run: |
          kubectl set image deployment/myapp \
            myapp=registry.company.com/myapp:staging
          kubectl rollout status deployment/myapp --timeout=120s
      
      - name: Wait for Application Ready
        run: |
          for i in $(seq 1 30); do
            STATUS=$(curl -s -o /dev/null -w "%{http_code}" $STAGING_URL/health)
            if [ "$STATUS" = "200" ]; then
              echo "Application is ready"
              exit 0
            fi
            echo "Waiting... attempt $i/30"
            sleep 10
          done
          echo "Application not ready after 5 minutes"
          exit 1

  # Stage 3: DAST Scans (parallel)
  dast-nuclei:
    needs: deploy-staging
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Nuclei Scan
        uses: projectdiscovery/nuclei-action@main
        with:
          target: ${{ env.STAGING_URL }}
          templates: "cves,vulnerabilities,misconfiguration,exposures"
          severity: "critical,high,medium"
          output: nuclei-results.json
          json: true
      - uses: actions/upload-artifact@v4
        with:
          name: nuclei-results
          path: nuclei-results.json

  dast-zap:
    needs: deploy-staging
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: ZAP Baseline Scan
        uses: zaproxy/action-baseline@v0.12.0
        with:
          target: ${{ env.STAGING_URL }}
          rules_file_name: '.zap/rules.tsv'
          cmd_options: '-a -j'
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: zap-results
          path: |
            report_html.html
            report_json.json

  # Stage 4: Quality Gate
  security-gate:
    needs: [dast-nuclei, dast-zap]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
      
      - name: Check Nuclei Results
        run: |
          if [ -f nuclei-results/nuclei-results.json ]; then
            CRITICAL=$(jq '[.[] | select(.info.severity=="critical")] | length' \
                       nuclei-results/nuclei-results.json)
            HIGH=$(jq '[.[] | select(.info.severity=="high")] | length' \
                   nuclei-results/nuclei-results.json)
            echo "Critical: $CRITICAL, High: $HIGH"
            if [ "$CRITICAL" -gt 0 ]; then
              echo "::error::$CRITICAL critical vulnerabilities found"
              exit 1
            fi
          fi
      
      - name: Generate Summary
        if: always()
        run: |
          echo "## DAST Security Report" >> $GITHUB_STEP_SUMMARY
          echo "| Scanner | Critical | High | Medium |" >> $GITHUB_STEP_SUMMARY
          echo "|---------|----------|------|--------|" >> $GITHUB_STEP_SUMMARY
          # Parse and add results...

  # Stage 5: Deploy to Production
  deploy-prod:
    needs: security-gate
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Deploy to Production
        run: |
          kubectl set image deployment/myapp \
            myapp=registry.company.com/myapp:${{ github.sha }}
```

---

## Environment Management

```
STAGING ENVIRONMENT FOR DAST:

  REQUIREMENTS:
  ┌──────────────────────────────────────────────────────────┐
  │                                                          │
  │  1. MIRROR PRODUCTION                                    │
  │  ┌────────────────────────────────────────────────────┐  │
  │  │ Same app version, config, and architecture         │  │
  │  │ Same security headers and TLS config               │  │
  │  │ Same WAF rules (or disable WAF for full testing)  │  │
  │  └────────────────────────────────────────────────────┘  │
  │                                                          │
  │  2. ISOLATION                                            │
  │  ┌────────────────────────────────────────────────────┐  │
  │  │ Separate database (no production data)             │  │
  │  │ Test accounts only (no real user data)             │  │
  │  │ Network isolated from production                    │  │
  │  └────────────────────────────────────────────────────┘  │
  │                                                          │
  │  3. RESETTABLE                                           │
  │  ┌────────────────────────────────────────────────────┐  │
  │  │ Can reset to known state after active scans        │  │
  │  │ Seed data script for consistent testing            │  │
  │  │ Automated teardown/rebuild between scans           │  │
  │  └────────────────────────────────────────────────────┘  │
  │                                                          │
  │  4. ACCESSIBLE                                           │
  │  ┌────────────────────────────────────────────────────┐  │
  │  │ Scanner IP whitelisted                              │  │
  │  │ Rate limits increased/disabled for scanner          │  │
  │  │ Monitoring alerts suppressed for scan traffic       │  │
  │  └────────────────────────────────────────────────────┘  │
  └──────────────────────────────────────────────────────────┘
```

---

## Result Aggregation & Deduplication

```python
# aggregate_dast_results.py
import json
from collections import defaultdict

def aggregate_results(zap_file, nuclei_file):
    """Aggregate and deduplicate findings from multiple scanners."""
    findings = []
    
    # Parse ZAP results
    with open(zap_file) as f:
        zap = json.load(f)
        for site in zap.get("site", []):
            for alert in site.get("alerts", []):
                findings.append({
                    "scanner": "ZAP",
                    "name": alert["name"],
                    "severity": map_zap_risk(alert["riskcode"]),
                    "url": alert.get("instances", [{}])[0].get("uri", ""),
                    "description": alert["desc"],
                    "cwe": alert.get("cweid", ""),
                })
    
    # Parse Nuclei results
    with open(nuclei_file) as f:
        for line in f:
            result = json.loads(line)
            findings.append({
                "scanner": "Nuclei",
                "name": result["info"]["name"],
                "severity": result["info"]["severity"],
                "url": result.get("matched-at", ""),
                "description": result["info"].get("description", ""),
                "cwe": "",
            })
    
    # Deduplicate by URL + CWE
    unique = {}
    for f in findings:
        key = f"{f['url']}:{f['cwe']}:{f['name']}"
        if key not in unique or severity_rank(f['severity']) > severity_rank(unique[key]['severity']):
            unique[key] = f
    
    return list(unique.values())

def map_zap_risk(code):
    return {0: "info", 1: "low", 2: "medium", 3: "high"}[int(code)]

def severity_rank(sev):
    return {"info": 0, "low": 1, "medium": 2, "high": 3, "critical": 4}[sev]

# Generate report
results = aggregate_results("zap-report.json", "nuclei-results.json")
summary = defaultdict(int)
for r in results:
    summary[r["severity"]] += 1
    
print(f"Total unique findings: {len(results)}")
for sev in ["critical", "high", "medium", "low", "info"]:
    print(f"  {sev.upper()}: {summary[sev]}")
```

---

## DAST Quality Gate Policy

```
QUALITY GATE DECISION MATRIX:

  ┌────────────────┬──────────────┬──────────────┬───────────┐
  │ Severity        │ PR Merge     │ Prod Deploy  │ Weekly    │
  ├────────────────┼──────────────┼──────────────┼───────────┤
  │ Critical        │ BLOCK        │ BLOCK        │ Ticket P1 │
  │ High            │ BLOCK        │ BLOCK        │ Ticket P2 │
  │ Medium          │ WARN         │ WARN         │ Ticket P3 │
  │ Low             │ INFO         │ INFO         │ Backlog   │
  │ Info            │ (suppress)   │ (suppress)   │ (ignore)  │
  └────────────────┴──────────────┴──────────────┴───────────┘

  GRADUAL ROLLOUT:
  ┌──────────────────────────────────────────────────────┐
  │ Month 1: DAST runs, reports only (no blocking)      │
  │ Month 2: Block on CRITICAL findings                  │
  │ Month 3: Block on CRITICAL + HIGH                    │
  │ Month 4: Full enforcement with all severities        │
  └──────────────────────────────────────────────────────┘
```

---

## Reporting & Notifications

```yaml
# Slack notification on DAST failure
- name: Notify Security Team
  if: failure()
  uses: slackapi/slack-github-action@v1.26.0
  with:
    payload: |
      {
        "blocks": [
          {
            "type": "header",
            "text": {"type": "plain_text", "text": "DAST Scan Failed"}
          },
          {
            "type": "section",
            "fields": [
              {"type": "mrkdwn", "text": "*Repository:*\n${{ github.repository }}"},
              {"type": "mrkdwn", "text": "*Branch:*\n${{ github.ref_name }}"},
              {"type": "mrkdwn", "text": "*Commit:*\n${{ github.sha }}"},
              {"type": "mrkdwn", "text": "*Report:*\n<${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}|View Report>"}
            ]
          }
        ]
      }
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SECURITY_SLACK_WEBHOOK }}
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Staging not ready when DAST starts | Race condition in deployment | Add health check wait step with retry loop before DAST |
| Different results than manual testing | Scanner can't reach all pages | Compare coverage: check ZAP site tree vs actual sitemap |
| Quality gate too noisy | Too many medium/low findings | Start by blocking only critical; gradually add severity levels |
| Results differ between runs | Non-deterministic scan | Use ZAP automation framework for consistent config; seed data |
| Can't correlate findings across scanners | Different naming/format | Use CWE IDs as common identifier; build aggregation script |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Health Check Gate** | Verify app is running before starting DAST |
| **Parallel Scanning** | Run ZAP + Nuclei simultaneously for speed |
| **Result Aggregation** | Combine and deduplicate findings from multiple scanners |
| **Quality Gate** | Severity-based pass/fail criteria for deployments |
| **Gradual Rollout** | Start reporting-only, progressively add blocking |
| **Environment Parity** | Staging must mirror production configuration |
| **Scan Isolation** | Use separate data and network for staging scans |

---

## Quick Revision Questions

1. **Why must the staging environment mirror production for DAST?**
   > DAST tests runtime behavior — if staging has different security headers, TLS config, or WAF rules than production, scan results won't reflect real-world security posture. Environment parity ensures findings are relevant.

2. **How do you prevent race conditions between deployment and DAST scans?**
   > Add a health check step that polls the application's health endpoint with retries before starting DAST. Use `needs:` dependency in CI to ensure deployment completes first, then verify the app is responding.

3. **Why use multiple DAST scanners instead of one?**
   > Different tools have different strengths: ZAP excels at comprehensive crawling and active scanning, Nuclei is fast for known CVE/misconfiguration checks. Running both in parallel provides better coverage with minimal extra time.

4. **What is the recommended gradual rollout strategy for DAST quality gates?**
   > Month 1: Run DAST but only report findings (non-blocking). Month 2: Block on critical findings. Month 3: Block on critical + high. Month 4: Full enforcement. This builds team confidence and allows time to tune false positives.

5. **How do you aggregate results from multiple DAST tools?**
   > Parse each tool's output format, normalize severity levels and vulnerability types, deduplicate by URL + CWE ID (keeping the highest severity when duplicates exist), then generate a unified report with severity counts.

---

[← Previous: 4.4 API Security Testing](04-api-security-testing.md) | [Next: 5.1 Third-Party Risks →](../05-SCA/01-third-party-risks.md)

[Back to Table of Contents](../README.md)
