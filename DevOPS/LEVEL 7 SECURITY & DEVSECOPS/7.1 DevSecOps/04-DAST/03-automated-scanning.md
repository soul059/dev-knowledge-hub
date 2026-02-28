# Chapter 4.3: Automated DAST Scanning

## Overview

Automated DAST scanning integrates dynamic security testing into CI/CD pipelines to catch runtime vulnerabilities on every deployment. This chapter covers practical automation strategies, pipeline configurations, and scan optimization.

---

## Automation Architecture

```
AUTOMATED DAST PIPELINE:

  ┌──────┐    ┌───────┐    ┌────────┐    ┌──────────┐    ┌──────┐
  │ Code │───→│ Build │───→│ Deploy │───→│  DAST    │───→│ Gate │
  │ Push │    │       │    │ to QA  │    │  Scan    │    │      │
  └──────┘    └───────┘    └────────┘    └──────────┘    └──┬───┘
                                                            │
                                              ┌─────────────┤
                                              │             │
                                         Pass │        Fail │
                                              ▼             ▼
                                        ┌──────────┐  ┌──────────┐
                                        │ Deploy   │  │ Block +  │
                                        │ to Prod  │  │ Alert    │
                                        └──────────┘  └──────────┘

  KEY: Application must be RUNNING before DAST can scan
```

---

## GitHub Actions Integration

### ZAP Baseline Scan

```yaml
name: DAST Scan
on:
  push:
    branches: [main]
  schedule:
    - cron: '0 2 * * 1'  # Weekly Monday 2AM

jobs:
  deploy-staging:
    runs-on: ubuntu-latest
    outputs:
      staging_url: ${{ steps.deploy.outputs.url }}
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to Staging
        id: deploy
        run: |
          # Deploy and capture URL
          echo "url=https://staging.myapp.com" >> $GITHUB_OUTPUT

  dast-scan:
    needs: deploy-staging
    runs-on: ubuntu-latest
    steps:
      - name: ZAP Baseline Scan
        uses: zaproxy/action-baseline@v0.12.0
        with:
          target: ${{ needs.deploy-staging.outputs.staging_url }}
          rules_file_name: '.zap/rules.tsv'
          cmd_options: '-a'  # Include AJAX spider
      
      - name: Upload Report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: zap-report
          path: report_html.html

  dast-nuclei:
    needs: deploy-staging
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Nuclei Scan
        uses: projectdiscovery/nuclei-action@main
        with:
          target: ${{ needs.deploy-staging.outputs.staging_url }}
          templates: "cves,vulnerabilities,misconfiguration"
          severity: "critical,high"
          output: nuclei-results.txt
      
      - name: Check Results
        run: |
          if [ -s nuclei-results.txt ]; then
            echo "Critical/High vulnerabilities found!"
            cat nuclei-results.txt
            exit 1
          fi
```

### ZAP Rules File

```tsv
# .zap/rules.tsv
# Rule ID    Action    Name
10021    IGNORE    X-Content-Type-Options Header Missing
10036    WARN      Server Leaks Version Info
10038    FAIL      Content Security Policy Header Not Set
40012    FAIL      Cross Site Scripting (Reflected)
40014    FAIL      Cross Site Scripting (Persistent)
40018    FAIL      SQL Injection
40019    FAIL      SQL Injection (MySQL)
90022    FAIL      Application Error Disclosure
```

### ZAP Full Scan (Nightly)

```yaml
name: DAST Full Scan (Nightly)
on:
  schedule:
    - cron: '0 1 * * *'  # Every night at 1AM

jobs:
  full-dast:
    runs-on: ubuntu-latest
    timeout-minutes: 120
    steps:
      - uses: actions/checkout@v4
      
      - name: ZAP Full Scan
        uses: zaproxy/action-full-scan@v0.10.0
        with:
          target: 'https://staging.myapp.com'
          rules_file_name: '.zap/rules.tsv'
          cmd_options: >-
            -a
            -j
            -m 60
            -z "-config scanner.maxScanDurationInMins=60"
      
      - name: Parse and Alert
        if: failure()
        uses: slackapi/slack-github-action@v1.26.0
        with:
          payload: |
            {
              "text": "DAST Full Scan found vulnerabilities. Check: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

---

## GitLab CI Integration

```yaml
# .gitlab-ci.yml
stages:
  - build
  - deploy-staging
  - dast
  - deploy-prod

deploy-staging:
  stage: deploy-staging
  script:
    - deploy_to_staging.sh
  environment:
    name: staging
    url: https://staging.myapp.com

dast:
  stage: dast
  image: ghcr.io/zaproxy/zaproxy:stable
  variables:
    DAST_TARGET: https://staging.myapp.com
  script:
    - mkdir -p /zap/wrk/
    - zap-baseline.py -t $DAST_TARGET
        -g gen.conf
        -r report.html
        -J report.json
        -a  # AJAX spider
  artifacts:
    when: always
    reports:
      dast: report.json
    paths:
      - report.html
  needs: ["deploy-staging"]

nuclei-scan:
  stage: dast
  image: projectdiscovery/nuclei:latest
  script:
    - nuclei -u https://staging.myapp.com
        -t cves/ -t vulnerabilities/ -t misconfiguration/
        -severity critical,high
        -json -o nuclei-results.json
  artifacts:
    paths:
      - nuclei-results.json
  needs: ["deploy-staging"]
  allow_failure: true  # Start as non-blocking, tighten over time

deploy-prod:
  stage: deploy-prod
  script:
    - deploy_to_prod.sh
  needs: ["dast"]
  when: on_success
```

---

## Handling Authentication in Automated Scans

```
AUTHENTICATION STRATEGIES FOR DAST:

  Strategy 1: FORM-BASED LOGIN
  ┌────────────────────────────────────────────────────┐
  │ ZAP Automation Framework handles login form         │
  │ Config: loginPageUrl, loginRequestUrl, credentials  │
  │ Best for: Traditional web apps                      │
  └────────────────────────────────────────────────────┘

  Strategy 2: API TOKEN / BEARER
  ┌────────────────────────────────────────────────────┐
  │ Pass auth header to scanner                         │
  │                                                      │
  │ # ZAP with auth header                               │
  │ zap-api-scan.py -t https://api.app.com/openapi.json │
  │   -z "-config replacer.full_list(0).enabled=true     │
  │       -config replacer.full_list(0).matchtype=REQ_HDR│
  │       -config replacer.full_list(0).matchstr=Auth..  │
  │       -config replacer.full_list(0).replacement=     │
  │               Bearer <token>"                         │
  │                                                      │
  │ # Nuclei with header                                 │
  │ nuclei -u https://api.app.com                        │
  │   -H "Authorization: Bearer <token>"                  │
  └────────────────────────────────────────────────────┘

  Strategy 3: PRE-AUTH SCRIPT
  ┌────────────────────────────────────────────────────┐
  │ Script authenticates before scan, passes cookies   │
  │                                                      │
  │ # pre-auth.sh                                       │
  │ TOKEN=$(curl -s -X POST https://api.app.com/login  │
  │   -d '{"user":"test","pass":"test123"}'             │
  │   | jq -r '.token')                                 │
  │ export AUTH_TOKEN=$TOKEN                             │
  │ # Pass to scanner                                   │
  └────────────────────────────────────────────────────┘
```

---

## Scan Scheduling Strategy

```
DAST SCAN SCHEDULING:

  ┌─────────────┬──────────────┬────────────┬──────────────┐
  │ Trigger      │ Scan Type    │ Duration   │ Blocking?    │
  ├─────────────┼──────────────┼────────────┼──────────────┤
  │ Every PR    │ Nothing      │ N/A        │ N/A          │
  │ (DAST needs │ (Use SAST)   │            │              │
  │  running app)│              │            │              │
  ├─────────────┼──────────────┼────────────┼──────────────┤
  │ Merge to    │ ZAP Baseline │ 5-10 min   │ Yes          │
  │ main        │ + Nuclei     │            │ (critical)   │
  ├─────────────┼──────────────┼────────────┼──────────────┤
  │ Nightly     │ ZAP Full     │ 1-2 hours  │ Report only  │
  │             │ Scan         │            │              │
  ├─────────────┼──────────────┼────────────┼──────────────┤
  │ Weekly      │ ZAP Full +   │ 4-8 hours  │ Report +     │
  │             │ Authenticated│            │ Ticket        │
  ├─────────────┼──────────────┼────────────┼──────────────┤
  │ On-demand   │ Targeted     │ Varies     │ Depends      │
  │             │ scan         │            │              │
  └─────────────┴──────────────┴────────────┴──────────────┘
```

---

## Incremental DAST (Scan Only Changed Endpoints)

```bash
#!/bin/bash
# incremental-dast.sh
# Only scan endpoints that were modified in this deployment

# 1. Extract changed API routes from git diff
CHANGED_FILES=$(git diff --name-only HEAD~1 -- 'src/routes/' 'src/controllers/')

# 2. Map file paths to API endpoints
ENDPOINTS=""
for file in $CHANGED_FILES; do
    # Extract route paths from annotated controllers
    routes=$(grep -oP '@(Get|Post|Put|Delete)\("([^"]+)"\)' "$file" | \
             grep -oP '"[^"]+"' | tr -d '"')
    ENDPOINTS="$ENDPOINTS $routes"
done

# 3. Create target list
BASE_URL="https://staging.myapp.com"
for endpoint in $ENDPOINTS; do
    echo "${BASE_URL}${endpoint}" >> targets.txt
done

# 4. Run focused scan
if [ -s targets.txt ]; then
    nuclei -l targets.txt \
        -t vulnerabilities/ \
        -severity critical,high \
        -json -o results.json
    echo "Scanned $(wc -l < targets.txt) changed endpoints"
else
    echo "No API endpoints changed — skipping DAST"
fi
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Staging environment not ready | Deploy not finished before scan starts | Use `needs:` dependencies; add health check wait loop |
| Scan times out in CI | Full scan too slow for pipeline | Use baseline scan in CI, full scan as nightly job |
| Auth token expires mid-scan | Long scans outlast token TTL | Use refresh tokens; increase staging token TTL; break into smaller scans |
| Different results each run | Non-deterministic crawling | Seed the spider, use automation framework for consistent config |
| DAST finds 0 issues | WAF/rate limiting blocking scanner | Whitelist scanner IP; disable rate limiting on staging |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Automated DAST** | DAST integrated into CI/CD for every deployment |
| **Baseline Scan** | Quick passive scan suitable for every merge |
| **Full Scan** | Comprehensive active scan for nightly/weekly |
| **Rules File** | TSV configuration defining which alerts to FAIL/WARN/IGNORE |
| **Authenticated Scan** | Scanner logs in to test post-auth endpoints |
| **Incremental DAST** | Scan only endpoints affected by recent changes |
| **Scan Scheduling** | Different scan depths based on pipeline trigger |

---

## Quick Revision Questions

1. **Why can't DAST run on every PR like SAST?**
   > DAST requires a running application to test. Deploying a full environment for every PR is expensive and slow. Instead, SAST runs on PRs, and DAST runs after deployment to staging (on merge to main or nightly).

2. **What is the difference between ZAP baseline and full scan?**
   > Baseline scan is passive only (observes responses, no attack payloads), runs in 2-10 minutes, and is safe for any environment. Full scan includes active scanning (sends attack payloads), takes 1-4+ hours, and should only target staging.

3. **How do you handle DAST scan authentication in CI/CD?**
   > Three strategies: form-based login (configured in ZAP automation framework), API token/bearer header (passed as scanner config), or pre-authentication script that obtains a token and passes it to the scanner.

4. **What is the recommended DAST scheduling strategy?**
   > Baseline + Nuclei on every merge to main (5-10 min, blocking on critical), ZAP full scan nightly (1-2 hours, non-blocking with reports), full authenticated scan weekly (4-8 hours, creates tickets).

5. **How can you make DAST faster in CI/CD?**
   > Use baseline (passive) scans instead of full scans, implement incremental scanning (only changed endpoints), limit spider depth and scan duration, use Nuclei for fast template-based checks, and run full scans as nightly jobs outside the deployment pipeline.

---

[← Previous: 4.2 DAST Tools](02-dast-tools.md) | [Next: 4.4 API Security Testing →](04-api-security-testing.md)

[Back to Table of Contents](../README.md)
