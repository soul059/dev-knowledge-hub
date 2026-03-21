# Unit 12: Web Testing Tools - Topic 6: Tool Chaining and Automation Workflows

## Overview

Tool chaining is the practice of combining multiple security tools into coordinated workflows where the output of one tool feeds into the next, creating powerful automated pipelines. Effective tool chaining transforms individual reconnaissance, scanning, and exploitation tools into **end-to-end penetration testing workflows** that can process hundreds of targets efficiently. This topic covers building pipelines with tools like Burp Suite, Nuclei, ffuf, httpx, subfinder, and custom scripts.

---

## 1. Tool Chaining Fundamentals

### The Pipeline Concept

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Discovery  │───▶│  Filtering   │───▶│   Scanning   │───▶│  Reporting   │
│              │    │              │    │              │    │              │
│  subfinder   │    │  httpx       │    │  nuclei      │    │  notify      │
│  amass       │    │  grep/awk    │    │  ffuf         │    │  custom      │
│  assetfinder │    │  anew        │    │  burp         │    │  markdown    │
└──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
```

### Data Flow Between Tools

| Source Tool | Output Format | Destination Tool | Input Method |
|-------------|--------------|-----------------|--------------|
| subfinder | domains (text) | httpx | stdin pipe |
| httpx | URLs (text/JSON) | nuclei | stdin pipe |
| ffuf | JSON results | custom scripts | file input |
| burp | XML export | custom parser | file input |
| nmap | XML/grepable | nuclei/custom | file input |
| waybackurls | URLs (text) | gf/grep | stdin pipe |
| gau | URLs (text) | ffuf | file input |

---

## 2. Reconnaissance Pipelines

### Full Subdomain Enumeration Pipeline

```bash
#!/bin/bash
# Full subdomain enumeration and probing pipeline
DOMAIN=$1
OUTPUT="results/$DOMAIN"
mkdir -p "$OUTPUT"

echo "[*] Stage 1: Subdomain Discovery"
# Passive enumeration from multiple sources
subfinder -d "$DOMAIN" -silent > "$OUTPUT/subs_subfinder.txt"
amass enum -passive -d "$DOMAIN" -o "$OUTPUT/subs_amass.txt" 2>/dev/null
assetfinder -subs-only "$DOMAIN" > "$OUTPUT/subs_assetfinder.txt"

# Merge and deduplicate
cat "$OUTPUT"/subs_*.txt | sort -u > "$OUTPUT/all_subdomains.txt"
echo "[+] Found $(wc -l < "$OUTPUT/all_subdomains.txt") unique subdomains"

echo "[*] Stage 2: DNS Resolution"
# Resolve DNS to filter live subdomains
puredns resolve "$OUTPUT/all_subdomains.txt" \
    -r resolvers.txt \
    -w "$OUTPUT/resolved.txt" 2>/dev/null

echo "[*] Stage 3: HTTP Probing"
# Probe for live HTTP services
cat "$OUTPUT/resolved.txt" | httpx -silent \
    -status-code -title -tech-detect \
    -json -o "$OUTPUT/httpx_results.json" \
    > "$OUTPUT/live_urls.txt"

echo "[+] Live hosts: $(wc -l < "$OUTPUT/live_urls.txt")"

echo "[*] Stage 4: Screenshot"
# Take screenshots of live hosts
cat "$OUTPUT/live_urls.txt" | gowitness file -f - \
    --screenshot-path "$OUTPUT/screenshots/" 2>/dev/null

echo "[*] Stage 5: Vulnerability Scanning"
# Run nuclei against live hosts
nuclei -l "$OUTPUT/live_urls.txt" \
    -t ~/nuclei-templates/ \
    -severity critical,high,medium \
    -json -o "$OUTPUT/nuclei_results.json" \
    -silent

echo "[*] Pipeline complete! Results in $OUTPUT/"
```

### URL Collection Pipeline

```bash
#!/bin/bash
# Collect and analyze all URLs for a target
DOMAIN=$1
OUTPUT="results/$DOMAIN/urls"
mkdir -p "$OUTPUT"

echo "[*] Collecting URLs from multiple sources..."

# Historical URLs
echo "$DOMAIN" | waybackurls > "$OUTPUT/wayback.txt" 2>/dev/null
echo "$DOMAIN" | gau --threads 5 > "$OUTPUT/gau.txt" 2>/dev/null

# Crawling
gospider -s "https://$DOMAIN" -d 3 -c 10 --other-source \
    | grep -oP 'https?://[^\s]+' > "$OUTPUT/crawled.txt" 2>/dev/null

# Merge all URLs
cat "$OUTPUT"/*.txt | sort -u > "$OUTPUT/all_urls.txt"
echo "[+] Total unique URLs: $(wc -l < "$OUTPUT/all_urls.txt")"

# Filter interesting URLs using gf patterns
echo "[*] Filtering for interesting patterns..."
cat "$OUTPUT/all_urls.txt" | gf sqli > "$OUTPUT/potential_sqli.txt"
cat "$OUTPUT/all_urls.txt" | gf xss > "$OUTPUT/potential_xss.txt"
cat "$OUTPUT/all_urls.txt" | gf ssrf > "$OUTPUT/potential_ssrf.txt"
cat "$OUTPUT/all_urls.txt" | gf lfi > "$OUTPUT/potential_lfi.txt"
cat "$OUTPUT/all_urls.txt" | gf redirect > "$OUTPUT/potential_redirect.txt"

# Extract unique parameters
cat "$OUTPUT/all_urls.txt" | grep "?" | cut -d'?' -f2 | tr '&' '\n' | \
    cut -d'=' -f1 | sort -u > "$OUTPUT/unique_params.txt"

echo "[+] Unique parameters found: $(wc -l < "$OUTPUT/unique_params.txt")"
echo "[+] Potential SQLi targets: $(wc -l < "$OUTPUT/potential_sqli.txt")"
echo "[+] Potential XSS targets: $(wc -l < "$OUTPUT/potential_xss.txt")"
```

---

## 3. Scanning Pipelines

### Nuclei Multi-Target Workflow

```bash
#!/bin/bash
# Comprehensive nuclei scanning pipeline

TARGETS_FILE=$1
OUTPUT_DIR="nuclei_results/$(date +%Y%m%d)"
mkdir -p "$OUTPUT_DIR"

echo "[*] Running nuclei scan phases..."

# Phase 1: Technology detection
echo "[*] Phase 1: Technology Detection"
nuclei -l "$TARGETS_FILE" \
    -t ~/nuclei-templates/technologies/ \
    -json -o "$OUTPUT_DIR/tech_detect.json" -silent

# Phase 2: Misconfigurations
echo "[*] Phase 2: Misconfiguration Checks"
nuclei -l "$TARGETS_FILE" \
    -t ~/nuclei-templates/misconfiguration/ \
    -json -o "$OUTPUT_DIR/misconfig.json" -silent

# Phase 3: CVE checks
echo "[*] Phase 3: Known CVE Checks"
nuclei -l "$TARGETS_FILE" \
    -t ~/nuclei-templates/cves/ \
    -severity critical,high \
    -json -o "$OUTPUT_DIR/cves.json" -silent

# Phase 4: Exposed panels
echo "[*] Phase 4: Exposed Panels"
nuclei -l "$TARGETS_FILE" \
    -t ~/nuclei-templates/exposed-panels/ \
    -json -o "$OUTPUT_DIR/panels.json" -silent

# Phase 5: Default credentials
echo "[*] Phase 5: Default Credentials"
nuclei -l "$TARGETS_FILE" \
    -t ~/nuclei-templates/default-logins/ \
    -json -o "$OUTPUT_DIR/default_logins.json" -silent

# Generate summary report
echo ""
echo "════════════════════════════════════════"
echo "         SCAN RESULTS SUMMARY"
echo "════════════════════════════════════════"
echo "Technologies:   $(wc -l < "$OUTPUT_DIR/tech_detect.json") findings"
echo "Misconfigs:     $(wc -l < "$OUTPUT_DIR/misconfig.json") findings"
echo "CVEs:           $(wc -l < "$OUTPUT_DIR/cves.json") findings"
echo "Exposed panels: $(wc -l < "$OUTPUT_DIR/panels.json") findings"
echo "Default logins: $(wc -l < "$OUTPUT_DIR/default_logins.json") findings"
echo "════════════════════════════════════════"
```

### Directory and Content Discovery Pipeline

```bash
#!/bin/bash
# Multi-phase directory discovery pipeline

TARGET=$1
OUTPUT="discovery_results"
mkdir -p "$OUTPUT"

echo "[*] Phase 1: Quick discovery with common paths"
ffuf -u "$TARGET/FUZZ" \
    -w /usr/share/seclists/Discovery/Web-Content/common.txt \
    -mc 200,301,302,403 \
    -o "$OUTPUT/common.json" \
    -of json -s

echo "[*] Phase 2: Extended discovery"
ffuf -u "$TARGET/FUZZ" \
    -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
    -mc 200,301,302 \
    -e .php,.asp,.aspx,.jsp,.html,.js,.json,.xml,.txt,.bak,.sql \
    -o "$OUTPUT/extended.json" \
    -of json -s -t 100

echo "[*] Phase 3: API endpoint discovery"
ffuf -u "$TARGET/FUZZ" \
    -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt \
    -mc 200,201,401,403,405 \
    -o "$OUTPUT/api.json" \
    -of json -s

echo "[*] Phase 4: Parameter discovery on found pages"
# Extract discovered paths
cat "$OUTPUT/common.json" | jq -r '.results[].input.FUZZ' 2>/dev/null | while read path; do
    echo "[*] Fuzzing parameters on /$path"
    ffuf -u "$TARGET/$path?FUZZ=test" \
        -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt \
        -mc 200 -ms 0 -fw 1 \
        -o "$OUTPUT/params_${path//\//_}.json" \
        -of json -s 2>/dev/null
done

echo "[+] Discovery complete! Results in $OUTPUT/"
```

---

## 4. Python Automation Framework

### Complete Pipeline Orchestrator

```python
#!/usr/bin/env python3
"""
Web Penetration Testing Pipeline Orchestrator
Chains multiple tools into automated workflows
"""
import subprocess
import json
import os
import sys
import time
from datetime import datetime
from pathlib import Path
from concurrent.futures import ThreadPoolExecutor, as_completed

class PipelineOrchestrator:
    def __init__(self, target, output_dir=None):
        self.target = target
        self.output_dir = output_dir or f"results/{target}/{datetime.now():%Y%m%d_%H%M}"
        Path(self.output_dir).mkdir(parents=True, exist_ok=True)
        self.results = {}
        
    def run_tool(self, name, command, parse_output=None):
        """Run a tool and capture output"""
        print(f"\n{'='*60}")
        print(f"[*] Running: {name}")
        print(f"[*] Command: {command}")
        print(f"{'='*60}")
        
        start_time = time.time()
        try:
            result = subprocess.run(
                command, shell=True, capture_output=True, 
                text=True, timeout=600
            )
            elapsed = time.time() - start_time
            
            output = result.stdout
            if parse_output:
                output = parse_output(output)
            
            self.results[name] = {
                'status': 'success',
                'output': output,
                'elapsed': elapsed,
                'returncode': result.returncode
            }
            print(f"[+] {name} completed in {elapsed:.1f}s")
            return output
            
        except subprocess.TimeoutExpired:
            print(f"[-] {name} timed out!")
            self.results[name] = {'status': 'timeout'}
            return None
        except Exception as e:
            print(f"[-] {name} failed: {e}")
            self.results[name] = {'status': 'error', 'error': str(e)}
            return None
    
    def recon_pipeline(self):
        """Full reconnaissance pipeline"""
        print("\n" + "█"*60)
        print("  RECONNAISSANCE PIPELINE")
        print("█"*60)
        
        # Subdomain enumeration
        subs_file = f"{self.output_dir}/subdomains.txt"
        self.run_tool(
            "Subfinder",
            f"subfinder -d {self.target} -silent -o {subs_file}"
        )
        
        # HTTP probing
        live_file = f"{self.output_dir}/live_hosts.txt"
        self.run_tool(
            "HTTPX",
            f"cat {subs_file} | httpx -silent -status-code -title "
            f"-json -o {self.output_dir}/httpx.json > {live_file}"
        )
        
        # Technology detection
        self.run_tool(
            "Wappalyzer",
            f"cat {live_file} | httpx -silent -tech-detect "
            f"-json -o {self.output_dir}/tech.json"
        )
        
        return live_file
    
    def scan_pipeline(self, targets_file):
        """Vulnerability scanning pipeline"""
        print("\n" + "█"*60)
        print("  VULNERABILITY SCANNING PIPELINE")
        print("█"*60)
        
        # Nuclei scan
        self.run_tool(
            "Nuclei",
            f"nuclei -l {targets_file} -severity critical,high,medium "
            f"-json -o {self.output_dir}/nuclei.json -silent"
        )
        
        # Directory discovery on each target
        with open(targets_file) as f:
            targets = [l.strip() for l in f if l.strip()]
        
        for target in targets[:20]:  # Limit to first 20
            safe_name = target.replace('://', '_').replace('/', '_')
            self.run_tool(
                f"ffuf-{safe_name}",
                f"ffuf -u {target}/FUZZ "
                f"-w /usr/share/seclists/Discovery/Web-Content/common.txt "
                f"-mc 200,301,302,403 -o {self.output_dir}/ffuf_{safe_name}.json "
                f"-of json -s"
            )
    
    def generate_report(self):
        """Generate final report"""
        report_path = f"{self.output_dir}/report.md"
        
        with open(report_path, 'w') as f:
            f.write(f"# Penetration Test Report\n\n")
            f.write(f"**Target:** {self.target}\n")
            f.write(f"**Date:** {datetime.now():%Y-%m-%d %H:%M}\n\n")
            
            f.write("## Tool Results Summary\n\n")
            f.write("| Tool | Status | Duration |\n")
            f.write("|------|--------|----------|\n")
            
            for name, result in self.results.items():
                status = result.get('status', 'unknown')
                elapsed = result.get('elapsed', 0)
                f.write(f"| {name} | {status} | {elapsed:.1f}s |\n")
            
            # Include nuclei findings
            nuclei_file = f"{self.output_dir}/nuclei.json"
            if os.path.exists(nuclei_file):
                f.write("\n## Vulnerability Findings\n\n")
                with open(nuclei_file) as nf:
                    for line in nf:
                        try:
                            finding = json.loads(line)
                            severity = finding.get('info', {}).get('severity', 'unknown')
                            name = finding.get('info', {}).get('name', 'Unknown')
                            host = finding.get('host', '')
                            f.write(f"- **[{severity.upper()}]** {name} - {host}\n")
                        except json.JSONDecodeError:
                            pass
        
        print(f"\n[+] Report saved to: {report_path}")
        return report_path
    
    def run_full_pipeline(self):
        """Execute the complete pipeline"""
        live_hosts = self.recon_pipeline()
        if live_hosts and os.path.exists(live_hosts):
            self.scan_pipeline(live_hosts)
        self.generate_report()

if __name__ == '__main__':
    if len(sys.argv) < 2:
        print(f"Usage: {sys.argv[0]} <domain>")
        sys.exit(1)
    
    pipeline = PipelineOrchestrator(sys.argv[1])
    pipeline.run_full_pipeline()
```

---

## 5. Burp Suite Integration

### Extending Burp with External Tools

```
┌────────────────────────────────────────────────────────────────┐
│                    BURP SUITE INTEGRATION                      │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ┌──────────┐    ┌────────────┐    ┌────────────────────────┐ │
│  │  Browser  │───▶│  Burp Proxy │───▶│  Target Application   │ │
│  └──────────┘    └──────┬─────┘    └────────────────────────┘ │
│                         │                                      │
│                    ┌────▼─────┐                                │
│                    │  Logger  │    Export to:                   │
│                    └────┬─────┘   ┌──────────────────┐        │
│                         │         │ • XML for Nuclei  │        │
│                         ├────────▶│ • JSON for Python │        │
│                         │         │ • CSV for Excel   │        │
│                         │         └──────────────────┘        │
│                    ┌────▼─────┐                                │
│                    │Extensions│   Custom extensions:           │
│                    └──────────┘   ┌──────────────────┐        │
│                                   │ • Autorize (IDOR) │        │
│                                   │ • Logger++        │        │
│                                   │ • Turbo Intruder  │        │
│                                   │ • Custom Python   │        │
│                                   └──────────────────┘        │
└────────────────────────────────────────────────────────────────┘
```

### Exporting Burp Data for Tool Chaining

```python
#!/usr/bin/env python3
"""Parse Burp Suite XML export and feed to other tools"""
import xml.etree.ElementTree as ET
import base64
import json

def parse_burp_xml(xml_file):
    """Parse Burp Suite XML export"""
    tree = ET.parse(xml_file)
    root = tree.getroot()
    
    requests = []
    for item in root.findall('item'):
        entry = {
            'url': item.find('url').text,
            'host': item.find('host').text,
            'port': int(item.find('port').text),
            'protocol': item.find('protocol').text,
            'method': item.find('method').text,
            'path': item.find('path').text,
            'status': int(item.find('status').text),
            'mime_type': item.find('mimetype').text,
        }
        
        # Decode request body
        req_elem = item.find('request')
        if req_elem is not None and req_elem.text:
            if req_elem.get('base64') == 'true':
                entry['request'] = base64.b64decode(req_elem.text).decode('utf-8', errors='ignore')
            else:
                entry['request'] = req_elem.text
        
        # Decode response body
        resp_elem = item.find('response')
        if resp_elem is not None and resp_elem.text:
            if resp_elem.get('base64') == 'true':
                entry['response'] = base64.b64decode(resp_elem.text).decode('utf-8', errors='ignore')
            else:
                entry['response'] = resp_elem.text
        
        requests.append(entry)
    
    return requests

def export_for_nuclei(requests, output_file):
    """Export unique URLs for Nuclei scanning"""
    urls = set()
    for req in requests:
        urls.add(req['url'])
    
    with open(output_file, 'w') as f:
        for url in sorted(urls):
            f.write(url + '\n')
    
    print(f"[+] Exported {len(urls)} URLs to {output_file}")

def export_for_ffuf(requests, output_file):
    """Export paths for ffuf fuzzing"""
    paths = set()
    for req in requests:
        path = req.get('path', '/')
        # Get directory paths for fuzzing
        parts = path.split('/')
        for i in range(1, len(parts)):
            paths.add('/'.join(parts[:i]) + '/')
    
    with open(output_file, 'w') as f:
        for path in sorted(paths):
            f.write(path + '\n')
    
    print(f"[+] Exported {len(paths)} paths to {output_file}")

# Usage
requests = parse_burp_xml('burp_export.xml')
export_for_nuclei(requests, 'nuclei_targets.txt')
export_for_ffuf(requests, 'ffuf_paths.txt')
```

---

## 6. CI/CD Security Pipeline Integration

### GitHub Actions Security Pipeline

```yaml
# .github/workflows/security-scan.yml
name: Security Scan Pipeline

on:
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 2 * * 1'  # Weekly Monday 2 AM

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Install tools
        run: |
          go install github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest
          go install github.com/projectdiscovery/httpx/cmd/httpx@latest
          pip install safety bandit semgrep
      
      - name: SAST - Semgrep
        run: semgrep --config=auto --json -o semgrep-results.json .
      
      - name: Dependency Check
        run: safety check -r requirements.txt --json > safety-results.json
      
      - name: DAST - Nuclei
        if: github.event_name == 'schedule'
        run: |
          nuclei -u ${{ secrets.STAGING_URL }} \
            -t ~/nuclei-templates/ \
            -severity critical,high \
            -json -o nuclei-results.json
      
      - name: Upload Results
        uses: actions/upload-artifact@v3
        with:
          name: security-results
          path: '*-results.json'
```

---

## 7. Real-World Tool Chain Examples

### Example 1: Bug Bounty Recon-to-Exploit Chain

```bash
# Step 1: Asset discovery
echo "target.com" | subfinder -silent | httpx -silent > live.txt

# Step 2: Collect all endpoints
cat live.txt | waybackurls | sort -u > endpoints.txt
cat live.txt | gau --threads 5 >> endpoints.txt
sort -u endpoints.txt -o endpoints.txt

# Step 3: Filter for vulnerable patterns
cat endpoints.txt | gf sqli | tee sqli_targets.txt
cat endpoints.txt | gf xss | tee xss_targets.txt

# Step 4: Test SQLi targets with sqlmap
while read url; do
    sqlmap -u "$url" --batch --level=3 --risk=2 \
        --output-dir=sqlmap_results/ --random-agent
done < sqli_targets.txt

# Step 5: Test XSS targets with dalfox
cat xss_targets.txt | dalfox pipe --silence \
    --output dalfox_results.txt
```

### Example 2: API Security Testing Chain

```bash
# Step 1: Discover API endpoints
ffuf -u "https://api.target.com/FUZZ" \
    -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt \
    -mc 200,201,401,403,405 \
    -o api_endpoints.json -of json -s

# Step 2: Extract found endpoints
cat api_endpoints.json | jq -r '.results[].url' > api_urls.txt

# Step 3: Test authentication on each endpoint
while read url; do
    # Test without auth
    code=$(curl -s -o /dev/null -w "%{http_code}" "$url")
    echo "$code $url" >> auth_test_results.txt
    
    # Test with invalid token
    code=$(curl -s -o /dev/null -w "%{http_code}" \
        -H "Authorization: Bearer invalid" "$url")
    echo "$code (invalid token) $url" >> auth_test_results.txt
done < api_urls.txt

# Step 4: Run nuclei with API templates
nuclei -l api_urls.txt \
    -t ~/nuclei-templates/http/misconfiguration/ \
    -t ~/nuclei-templates/http/vulnerabilities/ \
    -json -o api_nuclei_results.json
```

---

## 8. Notification and Reporting Automation

### Automated Report Generation

```python
#!/usr/bin/env python3
"""Generate HTML report from tool chain results"""
import json
from datetime import datetime

def generate_html_report(nuclei_file, httpx_file, output_file):
    """Combine results into a single HTML report"""
    
    # Parse nuclei results
    findings = []
    with open(nuclei_file) as f:
        for line in f:
            try:
                findings.append(json.loads(line))
            except json.JSONDecodeError:
                pass
    
    # Severity counts
    severity_counts = {}
    for f_item in findings:
        sev = f_item.get('info', {}).get('severity', 'unknown')
        severity_counts[sev] = severity_counts.get(sev, 0) + 1
    
    html = f"""<!DOCTYPE html>
<html>
<head>
    <title>Security Scan Report - {datetime.now():%Y-%m-%d}</title>
    <style>
        body {{ font-family: Arial, sans-serif; margin: 40px; }}
        .critical {{ color: #d32f2f; }}
        .high {{ color: #f57c00; }}
        .medium {{ color: #fbc02d; }}
        .low {{ color: #388e3c; }}
        table {{ border-collapse: collapse; width: 100%; }}
        th, td {{ border: 1px solid #ddd; padding: 8px; text-align: left; }}
        th {{ background-color: #333; color: white; }}
    </style>
</head>
<body>
    <h1>Security Scan Report</h1>
    <p>Generated: {datetime.now():%Y-%m-%d %H:%M:%S}</p>
    
    <h2>Summary</h2>
    <table>
        <tr><th>Severity</th><th>Count</th></tr>
"""
    for sev in ['critical', 'high', 'medium', 'low', 'info']:
        count = severity_counts.get(sev, 0)
        html += f'        <tr><td class="{sev}">{sev.upper()}</td><td>{count}</td></tr>\n'
    
    html += """    </table>
    
    <h2>Findings</h2>
    <table>
        <tr><th>Severity</th><th>Name</th><th>Host</th><th>Matched</th></tr>
"""
    for f_item in sorted(findings, key=lambda x: 
        ['critical','high','medium','low','info'].index(
            x.get('info',{}).get('severity','info'))):
        sev = f_item.get('info', {}).get('severity', 'unknown')
        name = f_item.get('info', {}).get('name', 'Unknown')
        host = f_item.get('host', '')
        matched = f_item.get('matched-at', '')
        html += f'        <tr><td class="{sev}">{sev.upper()}</td>'
        html += f'<td>{name}</td><td>{host}</td><td>{matched}</td></tr>\n'
    
    html += """    </table>
</body>
</html>"""
    
    with open(output_file, 'w') as f:
        f.write(html)
    
    print(f"[+] Report saved to: {output_file}")

# Usage
generate_html_report('nuclei.json', 'httpx.json', 'report.html')
```

### Slack/Discord Notifications

```bash
#!/bin/bash
# Send critical findings to Slack
WEBHOOK_URL="https://hooks.slack.com/services/YOUR/WEBHOOK/URL"
NUCLEI_RESULTS="nuclei_results.json"

# Count critical and high findings
CRITICAL=$(grep -c '"critical"' "$NUCLEI_RESULTS" 2>/dev/null || echo 0)
HIGH=$(grep -c '"high"' "$NUCLEI_RESULTS" 2>/dev/null || echo 0)

if [ "$CRITICAL" -gt 0 ] || [ "$HIGH" -gt 0 ]; then
    PAYLOAD=$(cat <<EOF
{
    "text": "🚨 *Security Scan Alert*",
    "blocks": [
        {
            "type": "section",
            "text": {
                "type": "mrkdwn",
                "text": "🚨 *Security Scan Complete*\n• Critical: $CRITICAL\n• High: $HIGH"
            }
        }
    ]
}
EOF
)
    curl -s -X POST -H 'Content-type: application/json' \
        --data "$PAYLOAD" "$WEBHOOK_URL"
    echo "[+] Alert sent to Slack"
fi
```

---

## Summary Table

| Pipeline Type | Tools Used | Purpose |
|---------------|-----------|---------|
| **Recon Pipeline** | subfinder → httpx → nuclei | Discover and probe assets |
| **URL Collection** | waybackurls → gau → gf | Gather and filter endpoints |
| **Scanning Pipeline** | nuclei + ffuf + custom | Vulnerability detection |
| **API Testing** | ffuf → curl → nuclei | API security assessment |
| **CI/CD Security** | semgrep + safety + nuclei | Automated DevSecOps |
| **Reporting** | JSON → Python → HTML/Slack | Findings communication |

---

## Revision Questions

1. Describe a complete recon-to-vulnerability-scan pipeline using at least 4 tools. What does each tool contribute?
2. How would you parse Burp Suite XML exports and feed the data into Nuclei for further scanning?
3. Write a Bash pipeline that collects URLs from multiple sources, filters for SQLi-vulnerable patterns, and tests them with sqlmap.
4. What are the advantages of using a Python orchestrator over plain Bash scripts for complex tool chains?
5. How would you integrate security scanning tools into a CI/CD pipeline using GitHub Actions?
6. Describe how you would set up automated notifications for critical findings using Slack or Discord webhooks.

---

*Previous: [05-custom-scripts.md](05-custom-scripts.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*
