# Chapter 2: HTTP/HTTPS

## Overview

HTTP (HyperText Transfer Protocol) and HTTPS (HTTP Secure) are the protocols that power the web. As a DevOps engineer, you'll configure web servers, reverse proxies, load balancers, API gateways, and CDNs — all of which rely heavily on HTTP/HTTPS. Understanding headers, status codes, methods, and TLS is essential for debugging, performance tuning, and security.

---

## 2.1 HTTP Basics

```
┌──────────────────────────────────────────────────────────────┐
│                 HTTP REQUEST/RESPONSE CYCLE                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│    Client (Browser)                    Server (Nginx)        │
│    ────────────────                    ──────────────         │
│         │                                   │                │
│         │─── HTTP Request ────────────────▶│                │
│         │    GET /index.html HTTP/1.1       │                │
│         │    Host: example.com              │                │
│         │    User-Agent: Chrome/120         │                │
│         │                                   │                │
│         │◀── HTTP Response ─────────────────│                │
│         │    HTTP/1.1 200 OK                │                │
│         │    Content-Type: text/html        │                │
│         │    Content-Length: 1234            │                │
│         │                                   │                │
│         │    <html>...</html>               │                │
│         │                                   │                │
└──────────────────────────────────────────────────────────────┘
```

---

## 2.2 HTTP Methods

```
┌─────────┬────────────────────────────────────────────────────┐
│ Method  │ Purpose                          │ Idempotent │Safe│
├─────────┼──────────────────────────────────┼────────────┼────┤
│ GET     │ Retrieve a resource              │ Yes        │ Yes│
│ POST    │ Create a new resource            │ No         │ No │
│ PUT     │ Update/replace a resource        │ Yes        │ No │
│ PATCH   │ Partially update a resource      │ No         │ No │
│ DELETE  │ Remove a resource                │ Yes        │ No │
│ HEAD    │ GET without body (headers only)  │ Yes        │ Yes│
│ OPTIONS │ Supported methods (CORS preflight)│ Yes       │ Yes│
└─────────┴──────────────────────────────────┴────────────┴────┘
```

---

## 2.3 HTTP Status Codes

```
┌──────────────────────────────────────────────────────────────┐
│                  HTTP STATUS CODES                           │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1xx — Informational                                         │
│  ├── 100 Continue                                            │
│  └── 101 Switching Protocols (WebSocket upgrade)             │
│                                                              │
│  2xx — Success                                               │
│  ├── 200 OK                                                  │
│  ├── 201 Created                                             │
│  ├── 202 Accepted (async processing)                         │
│  └── 204 No Content                                          │
│                                                              │
│  3xx — Redirection                                           │
│  ├── 301 Moved Permanently                                   │
│  ├── 302 Found (temporary redirect)                          │
│  ├── 304 Not Modified (cached)                               │
│  └── 307 Temporary Redirect (preserve method)                │
│                                                              │
│  4xx — Client Error                                          │
│  ├── 400 Bad Request                                         │
│  ├── 401 Unauthorized (needs auth)                           │
│  ├── 403 Forbidden (no permission)                           │
│  ├── 404 Not Found                                           │
│  ├── 405 Method Not Allowed                                  │
│  ├── 408 Request Timeout                                     │
│  ├── 429 Too Many Requests (rate limited)                    │
│  └── 499 Client Closed Request (Nginx specific)              │
│                                                              │
│  5xx — Server Error                                          │
│  ├── 500 Internal Server Error                               │
│  ├── 502 Bad Gateway (upstream failure)                      │
│  ├── 503 Service Unavailable                                 │
│  └── 504 Gateway Timeout                                     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### DevOps-Critical Status Codes

| Code | What It Means for DevOps |
|------|--------------------------|
| 200 | Health check passing |
| 301/302 | HTTP→HTTPS redirect configured correctly |
| 401 | API key or token missing/invalid |
| 403 | IAM/RBAC permission issue |
| 404 | Wrong path in proxy/route config |
| 429 | Rate limiting triggered — scale up or configure limits |
| 502 | Backend app crashed or not started |
| 503 | Backend overloaded, scaling needed |
| 504 | Backend too slow, increase timeout |

---

## 2.4 HTTP vs HTTPS

```
┌──────────────────────────────────────────────────────────────┐
│                  HTTP vs HTTPS                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  HTTP (Port 80)                                              │
│  ┌────────┐     PLAIN TEXT      ┌────────┐                   │
│  │ Client │ ──────────────────▶ │ Server │                   │
│  └────────┘   "password123"     └────────┘                   │
│                 ▲                                             │
│                 │ Attacker can read!                          │
│                 🔓                                            │
│                                                              │
│  HTTPS (Port 443)                                            │
│  ┌────────┐  TLS ENCRYPTED     ┌────────┐                   │
│  │ Client │ ══════════════════▶ │ Server │                   │
│  └────────┘  "x#k9$mQ2!p"      └────────┘                   │
│                 ▲                                             │
│                 │ Attacker sees gibberish!                    │
│                 🔒                                            │
│                                                              │
│  HTTPS = HTTP + TLS (Transport Layer Security)               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### TLS Handshake (Simplified)

```
┌──────────────────────────────────────────────────────────────┐
│              TLS HANDSHAKE (Simplified)                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Client                                 Server               │
│    │                                      │                  │
│    │── ClientHello ─────────────────────▶│                  │
│    │   (supported ciphers, TLS version)   │                  │
│    │                                      │                  │
│    │◀─ ServerHello + Certificate ─────────│                  │
│    │   (chosen cipher, server's cert)     │                  │
│    │                                      │                  │
│    │   Client verifies certificate         │                  │
│    │   against trusted CA list             │                  │
│    │                                      │                  │
│    │── Key Exchange ────────────────────▶│                  │
│    │   (generate shared secret)           │                  │
│    │                                      │                  │
│    │◀═══ ENCRYPTED DATA ════════════════▶│                  │
│    │     (using shared secret key)        │                  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2.5 HTTP Versions

```
┌──────────────────────────────────────────────────────────────┐
│                  HTTP VERSION COMPARISON                      │
├──────────┬────────────────────────────────────────────────────┤
│ Version  │ Key Features                                      │
├──────────┼────────────────────────────────────────────────────┤
│ HTTP/1.0 │ One request per TCP connection                     │
│          │ No persistent connections                         │
│          │                                                    │
│ HTTP/1.1 │ Persistent connections (keep-alive)                │
│          │ Pipelining (limited)                               │
│          │ Host header (virtual hosting)                      │
│          │ Chunked transfer encoding                          │
│          │                                                    │
│ HTTP/2   │ Multiplexing (parallel requests, one connection)   │
│          │ Header compression (HPACK)                         │
│          │ Server push                                        │
│          │ Binary protocol (not text)                         │
│          │                                                    │
│ HTTP/3   │ Built on QUIC (UDP, not TCP)                       │
│          │ Faster connection setup (0-RTT)                    │
│          │ No head-of-line blocking                           │
│          │ Built-in encryption                                │
└──────────┴────────────────────────────────────────────────────┘
```

---

## 2.6 HTTP Headers (DevOps Essentials)

```
┌──────────────────────────────────────────────────────────────┐
│              IMPORTANT HTTP HEADERS                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Request Headers:                                            │
│  ├── Host: example.com          (which domain?)              │
│  ├── User-Agent: Chrome/120     (which client?)              │
│  ├── Authorization: Bearer xyz  (authentication)             │
│  ├── Accept: application/json   (desired response format)    │
│  ├── X-Forwarded-For: 1.2.3.4  (original client IP)         │
│  ├── X-Forwarded-Proto: https   (original protocol)          │
│  └── Content-Type: application/json (payload format)         │
│                                                              │
│  Response Headers:                                           │
│  ├── Content-Type: text/html    (response format)            │
│  ├── Cache-Control: max-age=3600 (caching directive)         │
│  ├── Set-Cookie: session=abc    (session management)          │
│  ├── Strict-Transport-Security  (force HTTPS / HSTS)         │
│  ├── X-Content-Type-Options     (prevent MIME sniffing)       │
│  ├── X-Frame-Options            (prevent clickjacking)        │
│  └── Access-Control-Allow-Origin (CORS)                      │
│                                                              │
│  Load Balancer Headers:                                      │
│  ├── X-Forwarded-For            (client IP chain)            │
│  ├── X-Forwarded-Proto          (http or https)              │
│  ├── X-Forwarded-Port           (original port)              │
│  └── X-Real-IP                  (actual client IP)           │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2.7 Configuration Examples

### Nginx — HTTPS with HTTP→HTTPS Redirect

```nginx
# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name example.com;
    return 301 https://$server_name$request_uri;
}

# HTTPS server
server {
    listen 443 ssl http2;
    server_name example.com;

    ssl_certificate     /etc/ssl/certs/example.com.crt;
    ssl_certificate_key /etc/ssl/private/example.com.key;

    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "DENY" always;

    # Reverse proxy to backend
    location /api/ {
        proxy_pass http://backend:8080/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Let's Encrypt Certificate (Certbot)

```bash
# Install certbot
sudo apt install certbot python3-certbot-nginx

# Obtain and install certificate
sudo certbot --nginx -d example.com -d www.example.com

# Auto-renewal (cron)
echo "0 0 * * * root certbot renew --quiet" | sudo tee /etc/cron.d/certbot

# Test renewal
sudo certbot renew --dry-run
```

### curl — Debugging HTTP

```bash
# Simple GET request
curl https://example.com

# Verbose output (see headers, TLS handshake)
curl -v https://example.com

# Show only headers
curl -I https://example.com

# POST with JSON
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -d '{"name": "John", "email": "john@example.com"}'

# Follow redirects
curl -L http://example.com

# Check HTTP/2 support
curl --http2 -I https://example.com

# Time the request
curl -w "\nDNS: %{time_namelookup}s\nConnect: %{time_connect}s\nTLS: %{time_appconnect}s\nTotal: %{time_total}s\n" \
  -o /dev/null -s https://example.com
```

---

## 2.8 Real-World Scenarios

### [~] Scenario: Debugging 502 Bad Gateway

```
┌──────────────────────────────────────────────────────────────┐
│   CLIENT ──▶ LOAD BALANCER ──▶ NGINX ──▶ APP SERVER          │
│                                           │                  │
│              502 Bad Gateway ◀────────────┘                  │
│              (upstream failure)                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Step 1: Check if app is running                             │
│  $ systemctl status myapp                                    │
│  $ ss -tuln | grep 8080                                      │
│                                                              │
│  Step 2: Check Nginx error log                               │
│  $ tail -f /var/log/nginx/error.log                          │
│  "connect() failed (111: Connection refused)"                │
│  → App server is down!                                       │
│                                                              │
│  Step 3: Check app logs                                      │
│  $ journalctl -u myapp --since "5 minutes ago"               │
│  "OutOfMemoryError" → Increase memory limits                 │
│                                                              │
│  Step 4: Fix and verify                                      │
│  $ systemctl restart myapp                                   │
│  $ curl -I http://localhost:8080/health                      │
│  HTTP/1.1 200 OK ← Fixed!                                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### [~] Scenario: CORS Issues

```
Browser Console:
  "Access to XMLHttpRequest at 'https://api.example.com'
   from origin 'https://app.example.com' has been blocked
   by CORS policy"

Fix in Nginx:
  location /api/ {
      add_header 'Access-Control-Allow-Origin' 'https://app.example.com';
      add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
      add_header 'Access-Control-Allow-Headers' 'Authorization, Content-Type';

      if ($request_method = 'OPTIONS') {
          return 204;
      }

      proxy_pass http://backend:8080/;
  }
```

---

## 2.9 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| 502 Bad Gateway | Backend not running | Start backend, check ports |
| 504 Gateway Timeout | Backend too slow | Increase proxy timeout, optimize backend |
| ERR_CERT_AUTHORITY_INVALID | Self-signed or expired cert | Use Let's Encrypt, renew cert |
| Mixed Content warning | HTTP resources on HTTPS page | Update all URLs to HTTPS |
| CORS errors | Missing CORS headers | Add Access-Control headers |
| Redirect loop | Both HTTP and HTTPS redirect | Fix redirect logic in server config |
| Slow page load | No HTTP/2, no compression | Enable HTTP/2, gzip/brotli compression |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| HTTP | Port 80, plain text, stateless request-response protocol |
| HTTPS | Port 443, HTTP + TLS encryption |
| Methods | GET (read), POST (create), PUT (replace), DELETE (remove) |
| Status Codes | 2xx success, 3xx redirect, 4xx client error, 5xx server error |
| 502 | Backend unreachable (most common DevOps debug target) |
| HTTP/2 | Multiplexing, header compression, binary |
| HTTP/3 | QUIC/UDP, 0-RTT, no head-of-line blocking |
| TLS | Certificate-based encryption; Let's Encrypt for free certs |
| X-Forwarded-For | Preserves original client IP through proxies |

---

## Quick Revision Questions

1. **What's the difference between HTTP 401 and 403?**
2. **You see a 502 error. Walk through your debugging steps.**
3. **What does the `X-Forwarded-For` header contain, and why is it important?**
4. **Explain the TLS handshake process in simple terms.**
5. **How does HTTP/2 improve over HTTP/1.1?**
6. **What command would you use to see HTTP response headers and TLS certificate details?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← TCP vs UDP](01-tcp-vs-udp.md) | [README](../README.md) | [DNS →](03-dns.md) |
