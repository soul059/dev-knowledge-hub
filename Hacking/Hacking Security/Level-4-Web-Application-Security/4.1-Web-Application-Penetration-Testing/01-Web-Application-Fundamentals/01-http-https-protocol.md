# HTTP/HTTPS Protocol

## Unit 1: Web Application Fundamentals — Topic 1

## 🎯 Overview

HTTP (HyperText Transfer Protocol) and HTTPS (HTTP Secure) are the foundation of web communication. Understanding these protocols is essential for web application penetration testing, as every attack vector involves manipulating HTTP traffic. This topic covers the protocol mechanics, TLS/SSL encryption, and security implications from an offensive perspective.

---

## 1. HTTP Protocol Fundamentals

HTTP is a stateless, application-layer protocol operating on TCP port 80 by default.

```
┌──────────────┐         HTTP Request          ┌──────────────┐
│              │ ──────────────────────────────→│              │
│    Client    │    GET /index.html HTTP/1.1    │    Server    │
│   (Browser)  │    Host: example.com          │  (Web Server)│
│              │                                │              │
│              │←──────────────────────────────│              │
│              │         HTTP Response          │              │
│              │   HTTP/1.1 200 OK             │              │
│              │   Content-Type: text/html     │              │
└──────────────┘                                └──────────────┘
         :80                                          :80
```

### HTTP Versions

| Version | Year | Key Features |
|---------|------|-------------|
| HTTP/0.9 | 1991 | Single-line protocol, GET only |
| HTTP/1.0 | 1996 | Headers, status codes, content types |
| HTTP/1.1 | 1997 | Persistent connections, chunked transfer, Host header |
| HTTP/2 | 2015 | Binary framing, multiplexing, header compression |
| HTTP/3 | 2022 | QUIC (UDP-based), reduced latency |

### HTTP Communication Flow

```
Client                                              Server
  │                                                    │
  │──── TCP SYN ──────────────────────────────────────→│
  │←─── TCP SYN-ACK ─────────────────────────────────│
  │──── TCP ACK ──────────────────────────────────────→│
  │                                                    │
  │──── HTTP Request ─────────────────────────────────→│
  │     GET /login HTTP/1.1                            │
  │     Host: target.com                               │
  │                                                    │
  │←─── HTTP Response ────────────────────────────────│
  │     HTTP/1.1 200 OK                               │
  │     <html>...</html>                              │
  │                                                    │
  │──── TCP FIN ──────────────────────────────────────→│
  │←─── TCP FIN-ACK ─────────────────────────────────│
```

---

## 2. HTTPS and TLS/SSL

HTTPS encrypts HTTP traffic using TLS (Transport Layer Security), operating on TCP port 443.

### TLS Handshake Process

```
Client                                              Server
  │                                                    │
  │──── ClientHello ──────────────────────────────────→│
  │     (TLS version, cipher suites, random)           │
  │                                                    │
  │←─── ServerHello ──────────────────────────────────│
  │     (chosen cipher, certificate, random)           │
  │                                                    │
  │──── Key Exchange ─────────────────────────────────→│
  │     (pre-master secret encrypted with              │
  │      server's public key)                          │
  │                                                    │
  │←─── Finished ─────────────────────────────────────│
  │                                                    │
  │════ Encrypted HTTP Traffic ═══════════════════════│
```

### TLS Versions

| Version | Status | Security |
|---------|--------|----------|
| SSL 2.0 | Deprecated | ❌ Insecure — multiple vulnerabilities |
| SSL 3.0 | Deprecated | ❌ POODLE attack |
| TLS 1.0 | Deprecated | ⚠️ BEAST, weak ciphers |
| TLS 1.1 | Deprecated | ⚠️ No longer recommended |
| TLS 1.2 | Active | ✅ Secure with proper config |
| TLS 1.3 | Active | ✅ Most secure, simplified handshake |

---

## 3. HTTP vs HTTPS — Security Comparison

```
┌─────────────────────────────────────────────────────────────┐
│                    HTTP (Port 80)                            │
│                                                             │
│  Client ──── [Plaintext Traffic] ──── Server                │
│              Attacker can:                                  │
│              ✗ Sniff credentials                            │
│              ✗ Modify requests/responses                    │
│              ✗ Inject content (MitM)                        │
│              ✗ Session hijacking                            │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                   HTTPS (Port 443)                           │
│                                                             │
│  Client ════ [Encrypted Traffic] ════ Server                │
│              Attacker cannot:                               │
│              ✓ Read traffic content                         │
│              ✓ Modify in transit                            │
│              ✓ Impersonate server (cert validation)         │
└─────────────────────────────────────────────────────────────┘
```

---

## 4. Security Testing Considerations

### Common TLS/SSL Vulnerabilities

```bash
# Test SSL/TLS configuration with testssl.sh
testssl.sh https://target.com

# Check for specific vulnerabilities
testssl.sh --heartbleed https://target.com
testssl.sh --poodle https://target.com
testssl.sh --beast https://target.com

# Nmap SSL enumeration
nmap --script ssl-enum-ciphers -p 443 target.com

# OpenSSL manual testing
openssl s_client -connect target.com:443 -tls1
openssl s_client -connect target.com:443 -tls1_2
```

### Certificate Validation Issues

| Issue | Risk | Testing |
|-------|------|---------|
| Self-signed certificate | MitM possible | Browser warnings |
| Expired certificate | Trust issues | `openssl s_client` |
| Wrong hostname | Impersonation | Certificate CN/SAN check |
| Weak key size | Brute force | `testssl.sh` |
| Missing HSTS | Downgrade attacks | Header inspection |

### HTTP Downgrade Attacks

```
┌──────────┐    HTTPS     ┌──────────┐    HTTP      ┌──────────┐
│  Client  │ ────────────→│ Attacker │ ────────────→│  Server  │
│          │              │  (MitM)  │              │          │
│          │←────────────│ SSLStrip │←────────────│          │
│          │    HTTP      │          │    HTTPS     │          │
└──────────┘              └──────────┘              └──────────┘

# SSLStrip intercepts HTTPS links and serves HTTP
# HSTS prevents this by forcing HTTPS in the browser
```

---

## 5. Intercepting HTTP/HTTPS Traffic

```bash
# Burp Suite proxy setup
# Browser → Proxy (127.0.0.1:8080) → Target

# Install Burp CA certificate for HTTPS interception
# Export: Proxy → Options → Import/Export CA Certificate
# Import into browser's certificate store

# cURL through proxy
curl -x http://127.0.0.1:8080 https://target.com -k

# mitmproxy for transparent interception
mitmproxy --mode transparent --listen-port 8080
```

---

## 📊 Summary Table

| Aspect | HTTP | HTTPS |
|--------|------|-------|
| Port | 80 | 443 |
| Encryption | None | TLS/SSL |
| Data Visibility | Plaintext | Encrypted |
| Authentication | None | Certificate-based |
| MitM Risk | High | Low (with valid cert) |
| Performance | Faster | Slight overhead |
| SEO Impact | Neutral | Positive ranking boost |

---

## ❓ Revision Questions

1. What are the key differences between HTTP/1.1 and HTTP/2 from a security perspective?
2. Describe the TLS 1.3 handshake process and how it differs from TLS 1.2.
3. How does SSLStrip work and what prevents it?
4. What tools would you use to assess a target's TLS configuration?
5. Why is HTTP/2 multiplexing relevant to security testing?
6. What certificate validation checks should a penetration tester perform?

---

*Next: [02-request-response-structure.md](02-request-response-structure.md)*

---

*[Back to README](../README.md) | [Next →](02-request-response-structure.md)*
