# Chapter 2: Certificate Authorities

## Overview

A **Certificate Authority (CA)** is a trusted entity that issues digital certificates. CAs are the foundation of internet trust — they verify domain ownership and sign certificates that browsers trust. Understanding CA types, validation processes, and how to manage certificates is essential for maintaining secure production systems.

---

## 2.1 How CAs Work

```
┌──────────────────────────────────────────────────────────────┐
│             CERTIFICATE AUTHORITY WORKFLOW                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Step 1: Generate Key Pair                                   │
│  ┌──────────────┐                                            │
│  │ Server       │ → openssl genrsa -out private.key 2048     │
│  │              │ → Creates public/private key pair           │
│  └──────┬───────┘                                            │
│         │                                                    │
│  Step 2: Create CSR (Certificate Signing Request)            │
│  ┌──────▼───────┐                                            │
│  │ CSR Contains:│ → openssl req -new -key private.key        │
│  │ - Public key │      -out request.csr                      │
│  │ - Domain name│                                            │
│  │ - Org info   │                                            │
│  └──────┬───────┘                                            │
│         │                                                    │
│  Step 3: Submit CSR to CA                                    │
│         ▼                                                    │
│  ┌──────────────────────┐                                    │
│  │ Certificate Authority│ ← Validates domain ownership       │
│  │ (Let's Encrypt,      │ ← Signs certificate with CA's key  │
│  │  DigiCert, etc.)     │                                    │
│  └──────┬───────────────┘                                    │
│         │                                                    │
│  Step 4: Receive Signed Certificate                          │
│  ┌──────▼───────┐                                            │
│  │ Certificate  │ → Install on server with private key       │
│  │ (.crt/.pem)  │ → Include intermediate CA cert             │
│  └──────────────┘                                            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2.2 Popular Certificate Authorities

| CA | Type | Cost | Key Features |
|----|------|------|-------------|
| Let's Encrypt | Free, automated | Free | Auto-renewal, DV only, 90-day certs |
| DigiCert | Commercial | $$$ | EV/OV, enterprise support, warranty |
| Sectigo (Comodo) | Commercial | $$ | Wide range, affordable |
| GlobalSign | Commercial | $$$ | Enterprise, managed PKI |
| AWS ACM | Cloud-managed | Free* | Auto-renewing, AWS integration |
| GoDaddy | Commercial | $$ | Bundled with hosting |
| ZeroSSL | Freemium | Free-$$ | Free DV, dashboard, API |

*Free for use with AWS services (ALB, CloudFront, API Gateway)

---

## 2.3 Validation Methods

```
┌──────────────────────────────────────────────────────────────┐
│           DOMAIN VALIDATION METHODS                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. HTTP CHALLENGE (Let's Encrypt / ACM)                     │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ CA: "Place this file at this URL"                      │  │
│  │                                                        │  │
│  │ http://example.com/.well-known/acme-challenge/TOKEN    │  │
│  │                                                        │  │
│  │ CA verifies → you control the domain → cert issued     │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  2. DNS CHALLENGE                                            │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ CA: "Create this DNS TXT record"                       │  │
│  │                                                        │  │
│  │ _acme-challenge.example.com TXT "random-token-value"   │  │
│  │                                                        │  │
│  │ Required for: wildcard certs, domains without web      │  │
│  │ Preferred for: AWS ACM (auto-renewing with Route 53)   │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  3. EMAIL VALIDATION                                         │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ CA sends email to admin@example.com or                 │  │
│  │ webmaster@example.com                                  │  │
│  │                                                        │  │
│  │ You click approval link → cert issued                  │  │
│  │ Used by: Commercial CAs                                │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2.4 Generate CSR and Private Key

```bash
# Generate private key (RSA 2048-bit)
openssl genrsa -out private.key 2048

# Generate private key (ECDSA — faster, smaller)
openssl ecparam -genkey -name prime256v1 -out private.key

# Generate CSR
openssl req -new -key private.key -out request.csr \
  -subj "/C=US/ST=California/L=San Francisco/O=MyCompany/CN=example.com"

# Generate CSR with SANs (multiple domains)
openssl req -new -key private.key -out request.csr \
  -config <(cat <<EOF
[req]
default_bits = 2048
prompt = no
distinguished_name = dn
req_extensions = san

[dn]
CN = example.com
O = MyCompany

[san]
subjectAltName = DNS:example.com,DNS:www.example.com,DNS:api.example.com
EOF
)

# View CSR contents
openssl req -in request.csr -text -noout

# Generate self-signed certificate (development only!)
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem \
  -days 365 -nodes -subj "/CN=localhost"
```

---

## 2.5 Certificate Formats and Conversion

| Format | Extension | Encoding | Used By |
|--------|----------|----------|---------|
| PEM | .pem, .crt, .cer | Base64 text | Linux, Nginx, Apache |
| DER | .der, .cer | Binary | Java, Windows |
| PKCS#12 | .pfx, .p12 | Binary (key+cert) | Windows, IIS |
| PKCS#7 | .p7b, .p7c | Base64 (chain only) | Windows, Java |

```bash
# PEM to DER
openssl x509 -in cert.pem -outform DER -out cert.der

# DER to PEM
openssl x509 -in cert.der -inform DER -outform PEM -out cert.pem

# PEM to PKCS#12 (bundle key + cert for import)
openssl pkcs12 -export -out cert.pfx \
  -inkey private.key -in cert.pem -certfile chain.pem

# PKCS#12 to PEM
openssl pkcs12 -in cert.pfx -out cert.pem -nodes

# Extract private key from PFX
openssl pkcs12 -in cert.pfx -nocerts -out key.pem -nodes

# Extract certificate only from PFX
openssl pkcs12 -in cert.pfx -clcerts -nokeys -out cert.pem
```

---

## 2.6 Private CA (Internal Certificates)

```
┌──────────────────────────────────────────────────────────────┐
│            PRIVATE CA (INTERNAL PKI)                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Use Cases:                                                  │
│  - Internal service-to-service mTLS                          │
│  - Development environments                                  │
│  - IoT device certificates                                   │
│  - Corporate VPN authentication                              │
│                                                              │
│  AWS Private CA:                                             │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ aws acm-pca create-certificate-authority \              │  │
│  │   --certificate-authority-type ROOT \                   │  │
│  │   --certificate-authority-configuration \               │  │
│  │     '{"Subject":{"CommonName":"Internal CA"}}'          │  │
│  │                                                        │  │
│  │ Cost: $400/month per CA                                 │  │
│  │ + $0.75 per certificate issued                          │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  Alternatives:                                               │
│  - HashiCorp Vault PKI                                       │
│  - step-ca (smallstep)                                       │
│  - cfssl (CloudFlare)                                        │
│  - EJBCA                                                     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2.7 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| "Certificate not trusted" | Unknown CA or self-signed | Use public CA or add CA to trust store |
| CSR rejected by CA | Missing or wrong fields | Regenerate CSR with correct subject |
| Private key doesn't match cert | Wrong key used | Verify: `openssl x509 -modulus` matches `openssl rsa -modulus` |
| PFX import fails | Wrong password or corrupt | Re-export with correct password |
| ACM validation stuck "Pending" | DNS CNAME not created | Add validation CNAME to DNS |
| Certificate chain incomplete | Intermediate cert missing | Include fullchain.pem in server config |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| CA | Trusted entity that signs certificates |
| CSR | Certificate Signing Request (public key + domain info) |
| Root CA | Self-signed, in browser trust stores |
| Intermediate CA | Signed by root, signs server certs |
| HTTP Challenge | Prove domain ownership via web server |
| DNS Challenge | Prove ownership via DNS TXT record |
| PEM | Base64 text format (most common on Linux) |
| PKCS#12/PFX | Binary bundle of key + certificate |
| Private CA | Internal certificates for mTLS, dev, IoT |

---

## Quick Revision Questions

1. **Describe the CSR process from key generation to receiving a signed certificate.**
2. **What's the difference between HTTP and DNS domain validation?**
3. **Why can't you use a self-signed certificate in production?**
4. **How do you verify that a private key matches a certificate?**
5. **When would you set up a Private CA instead of using Let's Encrypt?**
6. **Convert a PEM certificate and key into a PKCS#12 (PFX) file. What command do you use?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← SSL/TLS Certificates](01-ssl-tls-certificates.md) | [README](../README.md) | [HTTPS Encryption →](03-https-encryption.md) |
