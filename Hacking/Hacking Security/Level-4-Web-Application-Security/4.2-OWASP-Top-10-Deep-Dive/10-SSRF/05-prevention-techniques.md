# Unit 10: A10 - Server-Side Request Forgery (SSRF) — Topic 5: Prevention Techniques

## Overview

Preventing SSRF requires a **defense-in-depth approach** because no single control is foolproof. The most effective strategy combines input validation, network-level controls, and architectural patterns that eliminate SSRF risk entirely. This topic covers comprehensive prevention techniques from application code through infrastructure.

---

## 1. Application-Level Prevention

### URL Validation (Allowlist Approach)

```python
import ipaddress
import socket
from urllib.parse import urlparse

class SSRFValidator:
    ALLOWED_SCHEMES = {'http', 'https'}
    ALLOWED_DOMAINS = {'api.example.com', 'cdn.example.com'}  # Allowlist
    
    BLOCKED_NETWORKS = [
        ipaddress.ip_network('127.0.0.0/8'),       # Loopback
        ipaddress.ip_network('10.0.0.0/8'),         # Private
        ipaddress.ip_network('172.16.0.0/12'),      # Private
        ipaddress.ip_network('192.168.0.0/16'),     # Private
        ipaddress.ip_network('169.254.0.0/16'),     # Link-local / metadata
        ipaddress.ip_network('0.0.0.0/8'),          # Current network
        ipaddress.ip_network('::1/128'),             # IPv6 loopback
        ipaddress.ip_network('fc00::/7'),            # IPv6 private
        ipaddress.ip_network('fe80::/10'),           # IPv6 link-local
    ]
    
    def validate_url(self, url):
        """Validate URL is safe to fetch"""
        # Step 1: Parse URL
        parsed = urlparse(url)
        
        # Step 2: Check scheme
        if parsed.scheme not in self.ALLOWED_SCHEMES:
            raise ValueError(f"Blocked scheme: {parsed.scheme}")
        
        # Step 3: Check for user info (http://evil@internal/)
        if parsed.username or parsed.password:
            raise ValueError("URL contains credentials")
        
        # Step 4: Resolve DNS (prevent DNS rebinding)
        hostname = parsed.hostname
        if not hostname:
            raise ValueError("No hostname in URL")
        
        try:
            resolved_ips = socket.getaddrinfo(hostname, parsed.port or 443)
        except socket.gaierror:
            raise ValueError(f"Cannot resolve: {hostname}")
        
        # Step 5: Check ALL resolved IPs against blocklist
        for family, _, _, _, sockaddr in resolved_ips:
            ip = ipaddress.ip_address(sockaddr[0])
            for network in self.BLOCKED_NETWORKS:
                if ip in network:
                    raise ValueError(f"Blocked IP: {ip}")
        
        # Step 6: Optionally check against allowlist
        if self.ALLOWED_DOMAINS and hostname not in self.ALLOWED_DOMAINS:
            raise ValueError(f"Domain not allowed: {hostname}")
        
        return True
    
    def safe_fetch(self, url, **kwargs):
        """Fetch URL with SSRF protection"""
        self.validate_url(url)
        
        # Disable redirects to prevent redirect-based bypass
        response = requests.get(url, allow_redirects=False, 
                                timeout=10, **kwargs)
        
        # If redirect, validate the redirect target too
        if response.is_redirect:
            redirect_url = response.headers.get('Location')
            self.validate_url(redirect_url)
            response = requests.get(redirect_url, allow_redirects=False,
                                    timeout=10)
        
        return response
```

---

## 2. DNS Pinning (Prevent DNS Rebinding)

```python
import socket
import requests
from urllib.parse import urlparse

def fetch_with_dns_pinning(url):
    """Resolve DNS once, validate IP, then fetch using resolved IP"""
    parsed = urlparse(url)
    hostname = parsed.hostname
    port = parsed.port or (443 if parsed.scheme == 'https' else 80)
    
    # Step 1: Resolve hostname to IP
    ip = socket.gethostbyname(hostname)
    
    # Step 2: Validate the resolved IP
    ip_obj = ipaddress.ip_address(ip)
    if ip_obj.is_private or ip_obj.is_loopback or ip_obj.is_link_local:
        raise ValueError(f"Resolved to internal IP: {ip}")
    
    # Step 3: Fetch using the resolved IP (not hostname)
    # This prevents DNS from changing between validation and fetch
    actual_url = url.replace(hostname, ip)
    response = requests.get(actual_url, 
                           headers={'Host': hostname},  # Keep Host header
                           allow_redirects=False,
                           timeout=10)
    return response
```

---

## 3. Network-Level Controls

```
INFRASTRUCTURE DEFENSES:

1. NETWORK SEGMENTATION
   ┌──────────┐     ┌──────────┐     ┌──────────┐
   │ Internet │ ──→ │ DMZ      │ ──→ │ Internal │
   │          │     │ (Web App)│     │ (DB,Redis)│
   └──────────┘     └──────────┘     └──────────┘
                         │
                    Firewall rules:
                    ALLOW: Web App → Internet (specific domains)
                    DENY:  Web App → Internal (except specific ports)
                    DENY:  Web App → 169.254.169.254

2. EGRESS FILTERING
   Only allow outbound connections to:
   ├── Specific external APIs (allowlist)
   ├── Specific ports (80, 443)
   └── Block ALL other outbound traffic

3. AWS SECURITY GROUPS
   # Block metadata access from application instances
   aws ec2 modify-instance-metadata-options \
     --instance-id i-1234567890 \
     --http-tokens required           # Enforce IMDSv2
   
   # Egress rules — only allow needed destinations
   Outbound: ALLOW 443 → api.stripe.com
   Outbound: DENY ALL → 169.254.169.254
   Outbound: DENY ALL → 10.0.0.0/8

4. KUBERNETES NETWORK POLICIES
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: deny-metadata
   spec:
     podSelector:
       matchLabels:
         app: web-app
     egress:
       - to:
         - ipBlock:
             cidr: 0.0.0.0/0
             except:
               - 169.254.169.254/32
               - 10.0.0.0/8
               - 172.16.0.0/12
               - 192.168.0.0/16
```

---

## 4. Architectural Patterns

```
PATTERN 1: PROXY SERVICE
  Instead of: App fetches user-provided URL directly
  Use: Dedicated proxy service with strict controls

  [User] → [App] → [Fetch Proxy] → [Internet]
                        │
                   Validates URL
                   Resolves DNS + checks IP
                   Enforces allowlist
                   Rate limits
                   Logs all requests

PATTERN 2: ASYNC PROCESSING
  Instead of: Synchronous URL fetch in request handler
  Use: Queue-based async processing

  [User submits URL] → [Queue] → [Worker (isolated network)]
                                        │
                                   Sandboxed environment
                                   No access to internal network
                                   Results stored, returned later

PATTERN 3: ELIMINATE THE FEATURE
  Ask: Does the app REALLY need to fetch user-provided URLs?
  Often: The feature can be redesigned to avoid SSRF entirely
  Example: Instead of "import from URL", allow "upload file"
```

---

## 5. Prevention Checklist

```
APPLICATION LAYER:
□ Allowlist permitted URL schemes (http, https only)
□ Allowlist permitted domains (if possible)
□ Resolve DNS and validate ALL resolved IPs
□ Block private, loopback, and link-local IPs
□ Disable HTTP redirects (or validate each hop)
□ Implement request timeout (prevent slow-loris)
□ Strip credentials from URLs (user:pass@host)
□ Use structured URL parsing (not regex)

NETWORK LAYER:
□ Enforce egress filtering (block internal ranges)
□ Block 169.254.169.254 from application layer
□ Enforce IMDSv2 on cloud instances
□ Network segmentation between tiers
□ Internal services require authentication

MONITORING:
□ Log all outbound requests from application servers
□ Alert on requests to internal IP ranges
□ Alert on requests to cloud metadata endpoints
□ Monitor for unusual outbound traffic patterns
```

---

## Summary Table

| Defense Layer | Technique | Effectiveness |
|--------------|-----------|---------------|
| Application | URL allowlist | High (if feasible) |
| Application | DNS pinning + IP validation | High |
| Application | Disable redirects | Medium |
| Network | Egress filtering | High |
| Network | Block metadata endpoint | High |
| Cloud | IMDSv2 enforcement | High |
| Architecture | Proxy service | Very High |
| Architecture | Eliminate the feature | Complete |

---

## Revision Questions

1. Implement a complete SSRF validation function with DNS pinning and IP checking.
2. Why is an allowlist approach more secure than a blocklist for SSRF prevention?
3. Design network-level controls to prevent SSRF in a cloud-hosted application.
4. Explain the proxy service pattern and how it mitigates SSRF risk.
5. What is DNS rebinding and how does DNS pinning prevent it?
6. Create a comprehensive SSRF prevention checklist for a security review.

---

*Previous: [04-ssrf-filter-bypass.md](04-ssrf-filter-bypass.md) | Next: [06-network-segmentation.md](06-network-segmentation.md)*

---

*[Back to README](../README.md)*
