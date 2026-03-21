# Unit 4: A04 - Insecure Design — Topic 4: Attack Surface Reduction

## Overview

The attack surface is the **total sum of all points where an attacker can try to enter or extract data** from a system. Attack surface reduction is a core secure design principle — every unnecessary feature, endpoint, port, protocol, or user privilege is a potential vulnerability. Reducing the attack surface minimizes opportunities for exploitation.

---

## 1. What Constitutes the Attack Surface?

```
┌─────────────────────────────────────────────────────────────────┐
│                    ATTACK SURFACE COMPONENTS                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  NETWORK SURFACE:                                              │
│  ├── Open ports and services                                   │
│  ├── Network protocols (HTTP, FTP, SSH, RDP)                   │
│  ├── DNS entries and subdomains                                │
│  ├── SSL/TLS endpoints                                         │
│  └── VPN and remote access points                              │
│                                                                 │
│  APPLICATION SURFACE:                                          │
│  ├── API endpoints (public and internal)                       │
│  ├── Web forms and input fields                                │
│  ├── File upload mechanisms                                    │
│  ├── Authentication endpoints                                  │
│  ├── Admin interfaces                                          │
│  ├── Error messages and debug info                             │
│  └── Third-party integrations                                  │
│                                                                 │
│  DATA SURFACE:                                                 │
│  ├── Databases and data stores                                 │
│  ├── Log files                                                 │
│  ├── Configuration files                                       │
│  ├── Backup files                                              │
│  ├── Temporary files                                           │
│  └── Client-side storage (cookies, localStorage)               │
│                                                                 │
│  HUMAN SURFACE:                                                │
│  ├── User accounts (especially privileged)                     │
│  ├── Social engineering targets                                │
│  ├── Physical access points                                    │
│  └── Support/helpdesk channels                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Attack Surface Reduction Strategies

### Network Level

```bash
# Before: Many open ports
$ nmap -sS target.com
PORT     STATE  SERVICE
21/tcp   open   ftp
22/tcp   open   ssh
25/tcp   open   smtp
80/tcp   open   http
443/tcp  open   https
3306/tcp open   mysql
5432/tcp open   postgresql
8080/tcp open   http-proxy
8443/tcp open   https-alt
9090/tcp open   admin-panel

# After: Only necessary ports
PORT     STATE  SERVICE
22/tcp   open   ssh         # Only from management VLAN
443/tcp  open   https       # Public-facing
```

```
Reduction Actions:
✅ Close unnecessary ports (FTP, SMTP if not needed)
✅ Move databases to internal network (no public access)
✅ Remove debug/admin ports from production
✅ Use VPN for administrative access
✅ Implement firewall rules (default deny)
✅ Segment networks (DMZ, internal, management)
```

### Application Level

```
┌─────────────────────────────────────────────────────────────────┐
│           APPLICATION ATTACK SURFACE REDUCTION                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  REMOVE UNUSED FEATURES:                                       │
│  ❌ /api/debug/memory-dump                                     │
│  ❌ /api/v1/users (deprecated version still active)            │
│  ❌ /admin/phpinfo.php                                         │
│  ❌ /api/test/create-user (test endpoint in production)        │
│  ❌ /.git/ (source code repository exposed)                    │
│                                                                 │
│  MINIMIZE API EXPOSURE:                                        │
│  ❌ GET /api/users → returns ALL fields (SSN, password hash)   │
│  ✅ GET /api/users → returns only id, name, email              │
│                                                                 │
│  RESTRICT ACCESS METHODS:                                      │
│  ❌ All HTTP methods allowed on all endpoints                  │
│  ✅ Only GET, POST where needed; block PUT, DELETE, TRACE      │
│                                                                 │
│  LIMIT INTEGRATIONS:                                           │
│  ❌ 15 third-party JavaScript libraries loaded                 │
│  ✅ Only essential libraries, SRI hashes, CSP                  │
└─────────────────────────────────────────────────────────────────┘
```

### API Endpoint Audit

```python
# Script to enumerate and audit API endpoints

from flask import Flask
import json

def audit_routes(app):
    """List all registered routes with their methods"""
    routes = []
    for rule in app.url_map.iter_rules():
        routes.append({
            'endpoint': rule.endpoint,
            'methods': list(rule.methods - {'OPTIONS', 'HEAD'}),
            'path': rule.rule,
            'requires_auth': hasattr(app.view_functions[rule.endpoint], '_auth_required'),
        })
    
    # Flag potential issues
    for route in routes:
        issues = []
        if not route['requires_auth'] and route['path'] != '/health':
            issues.append("⚠️  No authentication")
        if 'debug' in route['path'].lower():
            issues.append("🔴 Debug endpoint")
        if 'test' in route['path'].lower():
            issues.append("🔴 Test endpoint")
        if 'admin' in route['path'].lower() and 'DELETE' in route['methods']:
            issues.append("⚠️  Admin DELETE endpoint")
        
        route['issues'] = issues
    
    return routes
```

---

## 3. Least Functionality Principle

```
Install ONLY what is needed. Enable ONLY what is used.

Server Hardening Checklist:
┌────────────────────────────────────────┬──────────────────────┐
│ Action                                 │ Command / Method     │
├────────────────────────────────────────┼──────────────────────┤
│ Remove unused packages                 │ apt remove <package> │
│ Disable unnecessary services           │ systemctl disable    │
│ Remove default/sample applications     │ rm -rf /var/www/html │
│ Disable unused network protocols       │ modprobe -r <module> │
│ Remove development tools from prod     │ apt remove gcc make  │
│ Disable directory listing              │ Options -Indexes     │
│ Remove default credentials             │ Change on first use  │
│ Disable unused HTTP methods            │ Server config        │
│ Remove version banners                 │ ServerTokens Prod    │
└────────────────────────────────────────┴──────────────────────┘
```

---

## 4. Data Minimization

```
COLLECT ONLY WHAT YOU NEED:

❌ Registration form asks for:
   Name, Email, Password, Phone, Address, SSN, 
   Date of Birth, Mother's Maiden Name, Favorite Color

✅ Registration form asks for:
   Email, Password

Collect additional data ONLY when needed for a specific feature.

DATA RETENTION:
- Define retention periods for all data types
- Auto-delete data past retention period
- Anonymize data used for analytics
- Don't log sensitive data (passwords, tokens, PII)
```

---

## 5. Attack Surface Analysis Tools

| Tool | Purpose | Type |
|------|---------|------|
| **OWASP Attack Surface Detector** | Enumerate API endpoints from source | SAST |
| **Nmap** | Network port scanning | Network |
| **Burp Suite Spider** | Web application crawling | DAST |
| **Amass/Subfinder** | Subdomain enumeration | Recon |
| **Shodan** | Internet-exposed services | Recon |
| **nuclei** | Template-based vulnerability scanning | DAST |

---

## 6. Attack Surface Metrics

```
Track these metrics over time:

1. NUMBER OF ENDPOINTS
   API endpoints, web pages, services
   Trend: Should decrease or stay stable

2. NUMBER OF OPEN PORTS  
   Per server, per environment
   Trend: Minimize

3. THIRD-PARTY DEPENDENCIES
   Libraries, APIs, services
   Trend: Audit and minimize

4. PRIVILEGED ACCOUNTS
   Admin accounts, service accounts
   Trend: Minimize and rotate

5. DATA STORES
   Databases, file shares, caches
   Trend: Consolidate and encrypt

6. ENTRY POINTS
   Public URLs, forms, upload mechanisms
   Trend: Remove unused
```

---

## Summary Table

| Surface | Reduction Strategy | Example |
|---------|-------------------|---------|
| Network | Close ports, segment, VPN | Only 443 public |
| Application | Remove unused endpoints | Delete debug routes |
| Data | Minimize collection and retention | Only email for signup |
| API | Restrict methods and fields | GET-only, filtered response |
| Dependencies | Remove unused libraries | Audit package.json |
| Human | Reduce privileged accounts | MFA for all admins |

---

## Revision Questions

1. Define "attack surface" and list all four categories of attack surface components.
2. How do you reduce the network attack surface for a production web application?
3. What is the Least Functionality principle and how does it apply to server hardening?
4. Why is data minimization important for security? Give examples.
5. Design an attack surface audit process for a microservices architecture.
6. What metrics would you track to measure attack surface changes over time?

---

*Previous: [03-secure-design-patterns.md](03-secure-design-patterns.md) | Next: [05-defense-in-depth.md](05-defense-in-depth.md)*

---

*[Back to README](../README.md)*
