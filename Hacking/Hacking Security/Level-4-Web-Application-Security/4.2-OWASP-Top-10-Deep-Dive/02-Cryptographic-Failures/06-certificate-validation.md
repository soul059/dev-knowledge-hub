# Unit 2: A02 - Cryptographic Failures — Topic 6: Certificate Validation

## Overview

X.509 certificates are the backbone of **trust on the internet**, enabling browsers and applications to verify server identity and establish encrypted connections. Improper certificate validation — accepting expired certificates, ignoring hostname mismatches, disabling verification entirely, or trusting self-signed certificates — allows man-in-the-middle attacks that completely bypass TLS encryption. This topic covers certificate structures, validation chains, common failures, certificate transparency, and mutual TLS (mTLS).

---

## 1. X.509 Certificate Structure

```
┌─────────────────────────────────────────────────────────────────┐
│                   X.509 CERTIFICATE FIELDS                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Version: v3                                                   │
│  Serial Number: 04:B3:2F:...                                  │
│  Signature Algorithm: sha256WithRSAEncryption                  │
│                                                                 │
│  Issuer: CN=Let's Encrypt Authority X3, O=Let's Encrypt       │
│  Validity:                                                      │
│    Not Before: Jan 1 00:00:00 2024 UTC                         │
│    Not After:  Apr 1 00:00:00 2024 UTC                         │
│                                                                 │
│  Subject: CN=example.com                                       │
│  Subject Public Key Info:                                       │
│    Algorithm: RSA (2048 bit)                                   │
│    Public Key: 30:82:01:0a:02:82...                            │
│                                                                 │
│  Extensions:                                                    │
│    Subject Alternative Name (SAN):                              │
│      DNS: example.com, www.example.com, *.example.com          │
│    Basic Constraints: CA:FALSE                                 │
│    Key Usage: Digital Signature, Key Encipherment              │
│    Extended Key Usage: TLS Web Server Authentication           │
│    Authority Information Access:                                │
│      OCSP: http://ocsp.letsencrypt.org                         │
│    Certificate Transparency SCTs: (embedded)                   │
│                                                                 │
│  Signature: 2f:3a:b1:...                                      │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Certificate Chain of Trust

```
┌──────────────────────────────────────────────────────────────┐
│                CERTIFICATE CHAIN OF TRUST                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────────────┐                                │
│  │   ROOT CA Certificate   │  Pre-installed in browsers/OS  │
│  │   (Self-signed)         │  ~150 trusted root CAs         │
│  │   e.g., DigiCert Root   │  Validity: 20-30 years         │
│  └───────────┬─────────────┘                                │
│              │ Signs                                         │
│              ▼                                               │
│  ┌─────────────────────────┐                                │
│  │  INTERMEDIATE CA Cert   │  Signed by root CA             │
│  │  e.g., DigiCert SHA2   │  Validity: 5-10 years          │
│  │  Extended Validation    │  Provides isolation layer      │
│  └───────────┬─────────────┘                                │
│              │ Signs                                         │
│              ▼                                               │
│  ┌─────────────────────────┐                                │
│  │  SERVER (Leaf) Cert     │  Signed by intermediate        │
│  │  CN=www.example.com     │  Validity: 90 days-1 year      │
│  │  Your website's cert    │  Presented during TLS handshake│
│  └─────────────────────────┘                                │
│                                                              │
│  Validation: Browser walks chain from leaf → intermediate   │
│  → root, verifying each signature until it reaches a        │
│  trusted root in its certificate store                       │
└──────────────────────────────────────────────────────────────┘
```

---

## 3. Certificate Validation Process

### What Must Be Checked

| Check | Description | Failure Impact |
|-------|-------------|---------------|
| **Chain validity** | Each cert signed by its issuer up to trusted root | MITM with rogue cert |
| **Expiration** | Certificate within validity period | Expired cert → insecure |
| **Hostname match** | CN or SAN matches requested domain | MITM with valid cert for different domain |
| **Revocation** | Certificate not revoked (OCSP/CRL) | Compromised cert still trusted |
| **Key usage** | Certificate authorized for TLS server auth | Wrong cert type used |
| **Algorithm** | Signature uses strong algorithm (SHA-256+) | Forgeable with weak algorithm |

### Common Validation Failures

```python
# ❌ CRITICAL: Disabling certificate verification
import requests
response = requests.get('https://api.example.com', verify=False)
# This accepts ANY certificate — even attacker's!

# ❌ CRITICAL: Python SSL context with no verification
import ssl
context = ssl.create_default_context()
context.check_hostname = False
context.verify_mode = ssl.CERT_NONE  # Accepts everything!

# ❌ CRITICAL: Node.js disabling verification
process.env.NODE_TLS_REJECT_UNAUTHORIZED = '0';  // Global disable!

# ❌ CRITICAL: Java trust all certificates
TrustManager[] trustAll = new TrustManager[] {
    new X509TrustManager() {
        public void checkClientTrusted(X509Certificate[] chain, String authType) {}
        public void checkServerTrusted(X509Certificate[] chain, String authType) {}
        public X509Certificate[] getAcceptedIssuers() { return new X509Certificate[0]; }
    }
};
// This TrustManager accepts EVERY certificate

# ✅ SECURE: Always use default verification
response = requests.get('https://api.example.com')  # verify=True is default
# OR specify custom CA bundle:
response = requests.get('https://api.example.com', verify='/path/to/ca-bundle.crt')
```

---

## 4. Certificate Revocation

### OCSP vs CRL

```
CRL (Certificate Revocation List):
  ┌──────────────────────────────────────────────┐
  │  CA publishes a list of all revoked certs    │
  │  Client downloads the entire list            │
  │  Updated periodically (hours to days)        │
  │  Problem: Large downloads, stale data        │
  └──────────────────────────────────────────────┘

OCSP (Online Certificate Status Protocol):
  ┌──────────────────────────────────────────────┐
  │  Client asks CA: "Is cert X valid?"          │
  │  CA responds: Good / Revoked / Unknown       │
  │  Real-time check per certificate             │
  │  Problem: Privacy (CA knows what you visit)  │
  └──────────────────────────────────────────────┘

OCSP Stapling (Best approach):
  ┌──────────────────────────────────────────────┐
  │  Server pre-fetches OCSP response from CA    │
  │  Includes signed OCSP response in TLS        │
  │  handshake (stapled to certificate)          │
  │  Client verifies without contacting CA       │
  │  Solves: Privacy, performance, availability  │
  └──────────────────────────────────────────────┘
```

---

## 5. Certificate Transparency (CT)

```
┌────────────────────────────────────────────────────────────────┐
│               CERTIFICATE TRANSPARENCY                        │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  Problem: A rogue CA could issue a fake cert for google.com  │
│           without anyone knowing                               │
│                                                                │
│  Solution: All certificates must be logged publicly           │
│                                                                │
│  CA issues cert → Submits to CT Log → Gets SCT               │
│                                                                │
│  ┌────────┐    ┌──────────┐    ┌──────────────┐              │
│  │   CA   │───▶│  CT Log  │───▶│  SCT (Signed │              │
│  │        │    │ (public) │    │  Certificate │              │
│  │        │    │          │    │  Timestamp)  │              │
│  └────────┘    └──────────┘    └──────────────┘              │
│                                                                │
│  Monitors watch CT logs for unauthorized certificates         │
│  Organizations can detect rogue certs for their domains       │
│                                                                │
│  Tools:                                                        │
│  • crt.sh — Search CT logs                                   │
│  • Google Transparency Report                                 │
│  • Certstream — Real-time CT log monitoring                  │
└────────────────────────────────────────────────────────────────┘
```

```bash
# Search CT logs for a domain
curl -s "https://crt.sh/?q=%.example.com&output=json" | jq '.[].name_value' | sort -u

# Monitor new certificates in real-time
pip install certstream
python -c "
import certstream
def callback(message, context):
    if message['message_type'] == 'certificate_update':
        domains = message['data']['leaf_cert']['all_domains']
        for domain in domains:
            if 'example.com' in domain:
                print(f'New cert: {domain}')
certstream.listen_for_events(callback)
"
```

---

## 6. Mutual TLS (mTLS)

```
Standard TLS:
  Client ────────────────────▶ Server
         Server presents cert
         Client verifies server
         (One-way authentication)

Mutual TLS (mTLS):
  Client ◀───────────────────▶ Server
         Server presents cert     ← Client verifies server
         Client presents cert     ← Server verifies client
         (Two-way authentication)

Use cases:
  • API-to-API communication (microservices)
  • Zero-trust network architecture
  • IoT device authentication
  • Financial/banking APIs
```

### mTLS Nginx Configuration

```nginx
server {
    listen 443 ssl;
    
    # Server certificate
    ssl_certificate /etc/ssl/server.pem;
    ssl_certificate_key /etc/ssl/server.key;
    
    # Client certificate verification
    ssl_client_certificate /etc/ssl/client-ca.pem;
    ssl_verify_client on;       # Require client certificate
    ssl_verify_depth 2;         # Chain verification depth
    
    location /api/ {
        # Pass client cert info to application
        proxy_set_header X-Client-DN $ssl_client_s_dn;
        proxy_set_header X-Client-Verify $ssl_client_verify;
        proxy_pass http://backend;
    }
}
```

---

## 7. Testing Certificate Validation

```bash
# Check certificate details
echo | openssl s_client -connect target.com:443 2>/dev/null | openssl x509 -text -noout

# Check certificate chain
echo | openssl s_client -connect target.com:443 -showcerts 2>/dev/null

# Check expiration
echo | openssl s_client -connect target.com:443 2>/dev/null | openssl x509 -noout -dates

# Check hostname match
echo | openssl s_client -connect target.com:443 -servername target.com 2>/dev/null | \
    openssl x509 -noout -text | grep -A1 "Subject Alternative Name"

# Test with wrong hostname
curl -v --resolve wrong.com:443:$(dig +short target.com) https://wrong.com 2>&1 | grep "SSL"

# Check OCSP
openssl s_client -connect target.com:443 -status 2>/dev/null | grep "OCSP Response Status"

# Check Certificate Transparency
curl -s "https://crt.sh/?q=target.com&output=json" | python3 -m json.tool | head -50
```

---

## Summary Table

| Concept | Description | Failure Impact |
|---------|-------------|---------------|
| **Chain validation** | Verify signatures up to trusted root | Accept rogue certificates |
| **Hostname checking** | Match CN/SAN to requested domain | MITM with valid cert |
| **Expiration** | Reject expired certificates | Insecure connections |
| **Revocation (OCSP)** | Check if cert is revoked | Trust compromised certs |
| **CT Logs** | Public logging of all certificates | Detect rogue issuance |
| **mTLS** | Both client and server authenticate | Unauthorized API access |
| **Pinning** | Trust only specific cert/key | CA compromise mitigation |

---

## Revision Questions

1. Describe the X.509 certificate chain of trust from root CA to leaf certificate. How does a browser validate this chain?
2. What are the six checks that must be performed during certificate validation? What happens if each is skipped?
3. Compare CRL, OCSP, and OCSP Stapling for certificate revocation checking. Which is preferred and why?
4. What is Certificate Transparency? How does it help detect rogue certificates?
5. Explain mutual TLS (mTLS). How does it differ from standard TLS and when should it be used?
6. Show examples of insecure certificate validation in Python, Java, and Node.js. What is the secure alternative in each?

---

*Previous: [05-key-management.md](05-key-management.md) | Next: [07-prevention-strategies.md](07-prevention-strategies.md)*

---

*[Back to README](../README.md)*
