# Unit 4: Android Vulnerabilities — Topic 2: Insecure Communication

## Overview

**Insecure communication** occurs when mobile apps transmit sensitive data over unprotected or inadequately protected network channels. This includes using HTTP instead of HTTPS, accepting invalid certificates, using weak TLS configurations, or failing to implement certificate pinning — all of which enable Man-in-the-Middle (MitM) attacks.

---

## 1. Common Communication Vulnerabilities

```
INSECURE COMMUNICATION ISSUES:

1. CLEARTEXT HTTP:
   → Sensitive data sent over HTTP (unencrypted)
   → Interceptable on same network
   → WiFi sniffing, ARP spoofing

2. WEAK TLS CONFIGURATION:
   → TLS 1.0/1.1 (deprecated, vulnerable)
   → Weak cipher suites (RC4, DES, NULL)
   → No certificate validation

3. CERTIFICATE VALIDATION BYPASS:
   → TrustManager that accepts all certificates
   → HostnameVerifier that accepts all hostnames
   → App connects to ANY server presenting ANY cert

4. MISSING CERTIFICATE PINNING:
   → App trusts any CA-signed certificate
   → Attacker with CA cert can intercept
   → MitM via proxy or rogue CA

5. SENSITIVE DATA IN URLS:
   → Tokens/passwords in query parameters
   → Logged by web servers, proxies, ISPs
   → Example: https://api.com/login?token=secret123

6. INSECURE WEBSOCKET:
   → ws:// instead of wss://
   → No certificate validation on WSS
```

---

## 2. Testing for Insecure Communication

```bash
# Check AndroidManifest.xml
grep -i "cleartextTrafficPermitted\|usesCleartextTraffic" AndroidManifest.xml

# Check Network Security Config
cat res/xml/network_security_config.xml
# Look for: cleartextTrafficPermitted="true"

# Search code for HTTP URLs
grep -rn "http://" --include="*.java" decompiled_source/
# Any non-localhost HTTP URL = potential finding

# Search for certificate bypass code
grep -rn "TrustAllCerts\|ALLOW_ALL\|AllowAllHostname\|trustAllCerts" --include="*.java" decompiled_source/
grep -rn "X509TrustManager\|checkServerTrusted\|HostnameVerifier" --include="*.java" decompiled_source/

# Test with Burp Suite
# Configure proxy → Intercept traffic
# Look for:
# □ HTTP requests with sensitive data
# □ Missing HSTS headers
# □ Sensitive data in URL parameters
# □ API keys in headers
# □ Unencrypted PII in responses
```

---

## 3. Vulnerable Code Examples

```java
// VULNERABLE: TrustManager that trusts everything
TrustManager[] trustAllCerts = new TrustManager[] {
    new X509TrustManager() {
        public void checkClientTrusted(X509Certificate[] chain, String authType) {}
        public void checkServerTrusted(X509Certificate[] chain, String authType) {}
        public X509Certificate[] getAcceptedIssuers() { return new X509Certificate[0]; }
    }
};
SSLContext sc = SSLContext.getInstance("TLS");
sc.init(null, trustAllCerts, new SecureRandom());
// IMPACT: Accepts ANY certificate — MitM trivial!

// VULNERABLE: HostnameVerifier that accepts all
HostnameVerifier allHostsValid = (hostname, session) -> true;
HttpsURLConnection.setDefaultHostnameVerifier(allHostsValid);
// IMPACT: Connects to ANY hostname — doesn't verify server identity

// SECURE: Proper certificate pinning with OkHttp
CertificatePinner pinner = new CertificatePinner.Builder()
    .add("api.example.com", "sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=")
    .build();
OkHttpClient client = new OkHttpClient.Builder()
    .certificatePinner(pinner)
    .build();
```

---

## 4. Network Security Configuration

```xml
<!-- SECURE network_security_config.xml -->
<network-security-config>
    <!-- Block cleartext for all domains -->
    <base-config cleartextTrafficPermitted="false">
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </base-config>
    
    <!-- Certificate pinning for API -->
    <domain-config cleartextTrafficPermitted="false">
        <domain includeSubdomains="true">api.example.com</domain>
        <pin-set expiration="2025-01-01">
            <pin digest="SHA-256">primary_pin_base64</pin>
            <pin digest="SHA-256">backup_pin_base64</pin>
        </pin-set>
    </domain-config>
</network-security-config>
```

---

## Summary Table

| Vulnerability | Risk | Detection | Mitigation |
|--------------|------|-----------|-----------|
| HTTP traffic | Critical | Proxy intercept | HTTPS everywhere |
| Trust all certs | Critical | Code review | Proper validation |
| No pinning | High | Proxy intercept | Certificate pinning |
| Weak TLS | High | SSL scan | TLS 1.2+ only |
| Data in URLs | Medium | Traffic analysis | Use request body |

---

## Revision Questions

1. What are the main types of insecure communication vulnerabilities?
2. How do you detect a TrustManager that accepts all certificates?
3. What is the Network Security Configuration and how does it help?
4. How does certificate pinning prevent MitM attacks?
5. Why should sensitive data never be placed in URL parameters?

---

*Previous: [01-insecure-data-storage.md](01-insecure-data-storage.md) | Next: [03-insecure-authentication.md](03-insecure-authentication.md)*

---

*[Back to README](../README.md)*
