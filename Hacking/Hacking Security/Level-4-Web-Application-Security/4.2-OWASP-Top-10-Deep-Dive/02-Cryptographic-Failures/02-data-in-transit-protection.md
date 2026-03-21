# Unit 2: A02 - Cryptographic Failures — Topic 2: Data in Transit Protection

## Overview

Data in transit (data moving between client and server, or between services) is vulnerable to **eavesdropping, man-in-the-middle attacks, and tampering**. Transport Layer Security (TLS) is the primary protocol for protecting data in transit. Misconfigured TLS — weak cipher suites, outdated protocol versions, missing HSTS headers, or improper certificate handling — is one of the most common cryptographic failures found in web applications.

---

## 1. TLS/SSL Fundamentals

### TLS Handshake (TLS 1.2)

```
┌──────────┐                                    ┌──────────┐
│  Client   │                                    │  Server   │
├──────────┤                                    ├──────────┤
│           │──── ClientHello ──────────────────▶│           │
│           │     (TLS version, cipher suites,   │           │
│           │      random, extensions)           │           │
│           │                                    │           │
│           │◀─── ServerHello ───────────────────│           │
│           │     (chosen cipher, random)        │           │
│           │◀─── Certificate ───────────────────│           │
│           │     (X.509 certificate chain)      │           │
│           │◀─── ServerKeyExchange ─────────────│           │
│           │◀─── ServerHelloDone ───────────────│           │
│           │                                    │           │
│           │──── ClientKeyExchange ────────────▶│           │
│           │     (pre-master secret)            │           │
│           │──── ChangeCipherSpec ─────────────▶│           │
│           │──── Finished ────────────────────▶│           │
│           │                                    │           │
│           │◀─── ChangeCipherSpec ──────────────│           │
│           │◀─── Finished ─────────────────────│           │
│           │                                    │           │
│           │◀═══ Encrypted Application Data ═══▶│           │
└──────────┘                                    └──────────┘
```

### TLS 1.3 (Simplified — 1-RTT)

```
Client                                     Server
  │                                          │
  │──── ClientHello + KeyShare ────────────▶│
  │     (TLS 1.3, supported groups,         │
  │      key share, cipher suites)          │
  │                                          │
  │◀─── ServerHello + KeyShare ─────────────│
  │◀─── EncryptedExtensions ────────────────│
  │◀─── Certificate ───────────────────────│
  │◀─── CertificateVerify ─────────────────│
  │◀─── Finished ──────────────────────────│
  │                                          │
  │──── Finished ─────────────────────────▶│
  │                                          │
  │◀═══ Encrypted Data (0-RTT possible) ══▶│
```

---

## 2. TLS Version Comparison

| Feature | SSL 3.0 | TLS 1.0 | TLS 1.1 | TLS 1.2 | TLS 1.3 |
|---------|---------|---------|---------|---------|---------|
| **Status** | ❌ Broken | ❌ Deprecated | ❌ Deprecated | ✅ Acceptable | ✅ Recommended |
| **Year** | 1996 | 1999 | 2006 | 2008 | 2018 |
| **POODLE** | Vulnerable | ⚠️ Variant | Safe | Safe | Safe |
| **BEAST** | Vulnerable | Vulnerable | Safe | Safe | Safe |
| **Handshake RTT** | 2 | 2 | 2 | 2 | 1 (0-RTT) |
| **Perfect Forward Secrecy** | No | Optional | Optional | Optional | **Required** |
| **Key Exchange** | RSA, DH | RSA, DH | RSA, DH | RSA, ECDHE | **ECDHE only** |
| **PCI DSS** | ❌ | ❌ | ❌ | ✅ | ✅ |

> **Rule: Only TLS 1.2+ should be used. TLS 1.3 is preferred.**

---

## 3. Cipher Suite Configuration

### Recommended Cipher Suites

```
# TLS 1.3 cipher suites (all secure)
TLS_AES_256_GCM_SHA384
TLS_AES_128_GCM_SHA256
TLS_CHACHA20_POLY1305_SHA256

# TLS 1.2 recommended cipher suites (ECDHE for PFS)
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256
TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
```

### Cipher Suites to AVOID

```
# NEVER use these:
TLS_RSA_WITH_*                    # No forward secrecy
*_WITH_RC4_*                       # RC4 is broken
*_WITH_DES_*                       # DES is broken
*_WITH_3DES_*                      # 3DES is slow and weak (Sweet32)
*_WITH_NULL_*                      # No encryption!
*_EXPORT_*                         # Export-grade (weak)
*_WITH_*_CBC_*                     # CBC mode vulnerable to padding oracles
```

### Nginx Configuration

```nginx
# Secure TLS configuration for Nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    # Certificate
    ssl_certificate /etc/ssl/certs/example.com.pem;
    ssl_certificate_key /etc/ssl/private/example.com.key;
    
    # Protocols — TLS 1.2 and 1.3 only
    ssl_protocols TLSv1.2 TLSv1.3;
    
    # Cipher suites — server preference
    ssl_prefer_server_ciphers on;
    ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256';
    
    # ECDH curve
    ssl_ecdh_curve secp384r1;
    
    # OCSP Stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 8.8.8.8 8.8.4.4 valid=300s;
    
    # Session configuration
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:10m;
    ssl_session_tickets off;
    
    # Security headers
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name example.com;
    return 301 https://$host$request_uri;
}
```

---

## 4. HTTPS Enforcement (HSTS)

### HTTP Strict Transport Security

```
# HSTS Header
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload

Parameters:
  max-age=31536000      → Browser remembers for 1 year
  includeSubDomains     → Applies to all subdomains
  preload               → Submit to browser preload list
```

```
┌────────────────────────────────────────────────────────────────┐
│                    HSTS PROTECTION FLOW                        │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  WITHOUT HSTS:                                                │
│  User types: http://bank.com                                  │
│       │                                                        │
│       ▼                                                        │
│  ┌──────────┐  HTTP  ┌──────────┐  302 Redirect  ┌─────────┐│
│  │ Browser  │───────▶│  Server  │───────────────▶│ HTTPS   ││
│  └──────────┘        └──────────┘                └─────────┘│
│       ▲                                                       │
│       │ MITM can intercept the initial HTTP request!         │
│                                                                │
│  WITH HSTS:                                                   │
│  User types: http://bank.com                                  │
│       │                                                        │
│       ▼                                                        │
│  ┌──────────┐  Browser automatically upgrades to HTTPS        │
│  │ Browser  │───────────────────────────────────▶ HTTPS       │
│  └──────────┘  (no HTTP request ever sent)                    │
│                                                                │
│  First visit is still vulnerable → Use HSTS preload list     │
└────────────────────────────────────────────────────────────────┘
```

---

## 5. Known TLS Attacks

| Attack | Target | Description | Mitigation |
|--------|--------|-------------|------------|
| **POODLE** | SSL 3.0 | Padding oracle on downgraded legacy | Disable SSL 3.0 |
| **BEAST** | TLS 1.0 CBC | Chosen-plaintext via CBC IVs | Use TLS 1.2+ or GCM |
| **CRIME** | TLS compression | Compression side-channel | Disable TLS compression |
| **BREACH** | HTTP compression | Similar to CRIME for HTTP | Disable HTTP compression for sensitive data |
| **Heartbleed** | OpenSSL | Memory disclosure via heartbeat | Patch OpenSSL (CVE-2014-0160) |
| **DROWN** | SSLv2 | Cross-protocol attack | Disable SSLv2 everywhere |
| **FREAK** | Export ciphers | Downgrade to 512-bit RSA | Remove export cipher suites |
| **Logjam** | DHE export | Downgrade to 512-bit DH | Use 2048-bit DH or ECDHE |
| **Sweet32** | 3DES/Blowfish | Birthday attack on 64-bit blocks | Disable 3DES |
| **ROBOT** | RSA key exchange | Bleichenbacher oracle | Use ECDHE, not RSA key exchange |

---

## 6. Testing TLS Configuration

### Using testssl.sh

```bash
# Comprehensive TLS test
testssl.sh https://target.com

# Quick check
testssl.sh --fast https://target.com

# Check specific issues
testssl.sh --vulnerable https://target.com    # Known vulnerabilities
testssl.sh --protocols https://target.com      # Protocol versions
testssl.sh --ciphers https://target.com        # Cipher suites
testssl.sh --headers https://target.com        # Security headers
testssl.sh --pfs https://target.com            # Forward secrecy

# JSON output for automation
testssl.sh --jsonfile results.json https://target.com
```

### Using sslyze

```bash
# Full scan
sslyze target.com

# Check specific protocols
sslyze --tlsv1 --tlsv1_1 --tlsv1_2 --tlsv1_3 target.com

# Check for vulnerabilities
sslyze --heartbleed --robot --openssl_ccs target.com

# JSON output
sslyze --json_out results.json target.com
```

### Using Nmap

```bash
# Enumerate ciphers
nmap --script ssl-enum-ciphers -p 443 target.com

# Check certificate
nmap --script ssl-cert -p 443 target.com

# Check for known vulnerabilities
nmap --script ssl-poodle,ssl-heartbleed,ssl-dh-params -p 443 target.com
```

### Quick curl Check

```bash
# Check TLS version
curl -v --tls-max 1.0 https://target.com 2>&1 | grep "SSL connection"
curl -v --tls-max 1.1 https://target.com 2>&1 | grep "SSL connection"

# Check HSTS header
curl -sI https://target.com | grep -i strict-transport

# Check certificate details
echo | openssl s_client -connect target.com:443 2>/dev/null | openssl x509 -text -noout
```

---

## 7. Certificate Pinning

```
┌────────────────────────────────────────────────────────────────┐
│                   CERTIFICATE PINNING                          │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  Without Pinning:                                             │
│  Browser trusts ANY CA → Rogue CA can issue fake certs        │
│                                                                │
│  With Pinning:                                                │
│  App/browser only trusts SPECIFIC certificate or public key   │
│                                                                │
│  Types:                                                       │
│  1. Certificate pinning: Pin the exact certificate            │
│  2. Public key pinning: Pin the public key (survives renewal) │
│  3. SPKI hash pinning: Pin hash of SubjectPublicKeyInfo      │
│                                                                │
│  HTTP Public Key Pinning (HPKP) — DEPRECATED                 │
│  Public-Key-Pins: pin-sha256="base64hash"; max-age=5184000   │
│  ← Removed from browsers (risk of bricking sites)            │
│                                                                │
│  Modern Alternative: Certificate Transparency (CT)            │
│  Expect-CT: max-age=86400, enforce                            │
└────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Control | Purpose | Implementation |
|---------|---------|---------------|
| **TLS 1.2+** | Encrypt data in transit | Server configuration |
| **Strong ciphers** | Prevent cryptanalysis | ECDHE + AES-GCM |
| **HSTS** | Force HTTPS | Response header |
| **Certificate management** | Authenticate server | CA-signed certs, auto-renewal |
| **PFS** | Protect past sessions | ECDHE key exchange |
| **OCSP stapling** | Certificate status | Server-side stapling |
| **Regular testing** | Verify configuration | testssl.sh, sslyze |

---

## Revision Questions

1. Explain the TLS 1.2 handshake step by step. How does TLS 1.3 improve upon it?
2. What cipher suites should be used and which must be avoided? Why is forward secrecy important?
3. What is HSTS and how does it prevent SSL stripping attacks? What is the HSTS preload list?
4. Describe three historical TLS attacks (e.g., POODLE, Heartbleed, BEAST) and their mitigations.
5. How would you use testssl.sh to comprehensively audit a server's TLS configuration?
6. Write a secure Nginx TLS configuration block and explain each directive.

---

*Previous: [01-data-classification.md](01-data-classification.md) | Next: [03-data-at-rest-protection.md](03-data-at-rest-protection.md)*

---

*[Back to README](../README.md)*
