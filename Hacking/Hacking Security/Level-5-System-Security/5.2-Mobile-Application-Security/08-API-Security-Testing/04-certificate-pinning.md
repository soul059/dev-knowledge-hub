# Unit 8: API Security Testing — Topic 4: Certificate Pinning

## Overview

**Certificate pinning** for mobile APIs ensures that the app only communicates with servers presenting a specific, pre-approved certificate or public key. This prevents Man-in-the-Middle attacks even when an attacker has installed a rogue CA certificate on the device. This topic covers implementation, testing, and bypass from the API security testing perspective.

---

## 1. Why Pinning Matters for API Security

```
THREAT MODEL:

WITHOUT PINNING:
  App → [Trusts ANY CA] → Attacker's Proxy → API Server
  
  Attack scenario:
  1. Attacker on same WiFi network
  2. ARP spoof or rogue AP
  3. Burp/mitmproxy with custom CA
  4. CA installed on victim's device (or compromised CA)
  5. All API traffic visible and modifiable!

WITH PINNING:
  App → [Trusts ONLY server's cert/key] → API Server
  App → [Rejects proxy cert] ─X→ Attacker's Proxy
  
  Protection:
  ✓ Blocks rogue CA certificates
  ✓ Prevents corporate proxy interception
  ✓ Detects compromised CA authorities
  ✓ Ensures direct communication with legitimate server

WHAT TO PIN:
┌─────────────────┬─────────────┬─────────────────┐
│ Pin Target      │ Pros        │ Cons            │
├─────────────────┼─────────────┼─────────────────┤
│ Leaf cert       │ Most strict │ Breaks on renew │
│ Public key      │ Flexible    │ Key must persist│
│ Intermediate CA │ Balanced    │ CA compromise   │
│ Root CA         │ Easiest     │ Less protection │
└─────────────────┴─────────────┴─────────────────┘

RECOMMENDATION: Pin the public key (SPKI hash)
→ Survives certificate renewal if key is reused
→ Include backup pin for key rotation
```

---

## 2. Implementation Approaches

```
ANDROID PINNING:

1. Network Security Config (Recommended):
   <!-- res/xml/network_security_config.xml -->
   <network-security-config>
       <domain-config>
           <domain includeSubdomains="true">api.example.com</domain>
           <pin-set expiration="2025-06-01">
               <pin digest="SHA-256">base64_primary_pin</pin>
               <pin digest="SHA-256">base64_backup_pin</pin>
           </pin-set>
       </domain-config>
   </network-security-config>

2. OkHttp CertificatePinner:
   CertificatePinner pinner = new CertificatePinner.Builder()
       .add("api.example.com", "sha256/AAAA...")
       .add("api.example.com", "sha256/BBBB...")  // backup
       .build();

iOS PINNING:

1. TrustKit:
   let config: [String: Any] = [
       kTSKSwizzleNetworkDelegates: true,
       kTSKPinnedDomains: [
           "api.example.com": [
               kTSKPublicKeyHashes: [
                   "primary_pin_base64",
                   "backup_pin_base64"
               ],
               kTSKEnforcePinning: true,
               kTSKIncludeSubdomains: true
           ]
       ]
   ]

2. NSURLSession delegate (custom):
   → Implement URLSession:didReceiveChallenge:completionHandler:
   → Extract server public key
   → Compare against pinned hashes
   → Accept or reject connection
```

---

## 3. Testing Pinning Effectiveness

```
TESTING METHODOLOGY:

1. CONFIGURE PROXY:
   → Set up Burp Suite with CA cert
   → Install CA cert on test device
   → Configure device proxy settings

2. TEST WITHOUT BYPASS:
   → Launch app
   → Attempt to use app functionality
   → If traffic appears in Burp: NO PINNING (finding!)
   → If app shows error/refuses: Pinning IS active

3. TEST BYPASS RESILIENCE:
   → Try SSL Kill Switch 2
   → Try objection: ios sslpinning disable
   → Try Frida universal bypass script
   → If all fail: strong pinning implementation

4. VERIFY PIN ROTATION:
   → Does app have backup pins?
   → What happens when pin expires?
   → Graceful degradation or app crash?

REPORTING:
┌──────────────────────┬──────────────────────┐
│ Finding              │ Severity             │
├──────────────────────┼──────────────────────┤
│ No pinning at all    │ HIGH                 │
│ Pinning on some APIs │ MEDIUM               │
│ Weak pinning (easy   │ MEDIUM               │
│   bypass)            │                      │
│ Strong pinning       │ INFORMATIONAL (good!)│
│ No backup pins       │ LOW (operational)    │
│ Expired pins         │ MEDIUM (may break)   │
└──────────────────────┴──────────────────────┘
```

---

## 4. Operational Considerations

```
PINNING CHALLENGES:

CERTIFICATE ROTATION:
  → Certs expire (typically 1-2 years)
  → Must update pins before expiration
  → App update required (unless backup pin works)
  → Can brick app if not managed properly!
  
  SOLUTION:
  → Pin public keys (not full cert)
  → Include 2+ backup pins
  → Set pin expiration with grace period
  → Monitor certificate expiration dates

CDN AND LOAD BALANCERS:
  → Multiple servers with different certs
  → CDN (CloudFlare, AWS CloudFront) may change certs
  → Pin to intermediate CA or specific CDN cert
  
DEBUGGING AND QA:
  → Pinning blocks proxy debugging
  → Need debug builds without pinning
  → Or conditional pinning (disabled in debug)
  → RISK: Debug builds must not ship!
  
  #if DEBUG
  // No pinning in debug builds
  #else
  // Pinning enabled in production
  #endif

INCIDENT RESPONSE:
  → If private key is compromised
  → Must issue new cert with new key
  → Must push app update with new pin
  → Backup pin prevents immediate outage
```

---

## Summary Table

| Aspect | Best Practice |
|--------|--------------|
| What to pin | Public key (SPKI hash) |
| Number of pins | 2+ (primary + backup) |
| Platform (Android) | Network Security Config |
| Platform (iOS) | TrustKit or custom delegate |
| Rotation | Plan cert rotation before expiry |
| Debug | Disable only in debug builds |
| Testing | Verify with proxy, then test bypass |

---

## Revision Questions

1. Why is certificate pinning important for mobile API security?
2. What should you pin: the certificate, public key, or CA?
3. How do you test whether an app implements certificate pinning?
4. What operational challenges does pinning create?
5. How do backup pins prevent app outages during certificate rotation?

---

*Previous: [03-token-handling.md](03-token-handling.md) | Next: [05-api-vulnerabilities.md](05-api-vulnerabilities.md)*

---

*[Back to README](../README.md)*
