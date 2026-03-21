# Unit 6: iOS Testing — Topic 4: Traffic Interception

## Overview

**Traffic interception** on iOS involves capturing, analyzing, and modifying network communication between the app and its backend servers. This reveals API endpoints, authentication mechanisms, data exposure, and server-side vulnerabilities. iOS apps present unique challenges including ATS enforcement, certificate pinning, and non-HTTP protocols.

---

## 1. Proxy Setup for iOS

```
TRAFFIC INTERCEPTION SETUP:

┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│  iOS Device │────→│  Burp Suite  │────→│  API Server │
│  (Proxy     │     │  (Intercept) │     │             │
│   config)   │←────│              │←────│             │
└─────────────┘     └──────────────┘     └─────────────┘

STEP 1: Configure Burp
  Proxy → Proxy Settings → Add listener
  Bind to port: 8080
  Bind to address: All interfaces (or specific IP)

STEP 2: Configure iOS device
  Settings → Wi-Fi → [Network name] → Configure Proxy
  Manual → Server: <Burp IP> → Port: 8080

STEP 3: Install Burp CA certificate
  On iOS Safari: http://<burp-ip>:8080
  → Download CA Certificate
  → Settings → General → VPN & Device Management → Install
  → Settings → General → About → Certificate Trust Settings
  → Enable Full Trust for PortSwigger CA

STEP 4: Verify interception
  → Browse to https://example.com
  → Check Burp HTTP history for captured request
```

---

## 2. Handling Certificate Pinning

```
CERTIFICATE PINNING BYPASS METHODS:

1. SSL KILL SWITCH 2 (Jailbreak Tweak):
   → Install via Cydia/Sileo
   → Globally disables certificate validation
   → Works for most apps using standard APIs
   → Toggle in Settings app

2. OBJECTION (Frida-based):
   objection -g com.example.app explore
   > ios sslpinning disable
   → Hooks common pinning implementations
   → Works with: NSURLSession, AFNetworking, Alamofire

3. FRIDA CUSTOM SCRIPTS:
```

```javascript
// Frida: Universal SSL pinning bypass
// Hooks multiple TLS validation methods

// NSURLSession delegate bypass
var NSURLSessionConfiguration = ObjC.classes.NSURLSessionConfiguration;
Interceptor.attach(ObjC.classes.NSURLSession[
    '- URLSession:didReceiveChallenge:completionHandler:'].implementation, {
    onEnter: function(args) {
        var handler = new ObjC.Block(args[4]);
        handler.implementation = function(disposition, credential) {
            // Accept any certificate
            handler(0, null); // NSURLSessionAuthChallengeUseCredential
        };
    }
});

// TrustKit bypass
if (ObjC.classes.TSKPinningValidator) {
    Interceptor.attach(
        ObjC.classes.TSKPinningValidator['- evaluateTrust:forHostname:'].implementation, {
        onLeave: function(retval) {
            retval.replace(0); // TSKTrustDecisionShouldAllowConnection
        }
    });
}

// AFNetworking bypass
if (ObjC.classes.AFSecurityPolicy) {
    Interceptor.attach(
        ObjC.classes.AFSecurityPolicy['- evaluateServerTrust:forDomain:'].implementation, {
        onLeave: function(retval) {
            retval.replace(0x1); // Return YES
        }
    });
}
```

```
4. MANUAL PATCHING:
   → Decompile binary with Ghidra
   → Find pinning validation function
   → Patch to always return success
   → Re-sign and deploy

PINNING BYPASS DIFFICULTY:
┌─────────────────────┬────────────┬─────────────────┐
│ Pinning Method      │ Bypass     │ Tool            │
├─────────────────────┼────────────┼─────────────────┤
│ NSURLSession        │ Easy       │ SSL Kill Switch │
│ AFNetworking        │ Easy       │ objection       │
│ Alamofire           │ Easy       │ objection       │
│ TrustKit            │ Medium     │ Frida script    │
│ Custom (native)     │ Hard       │ Ghidra + Frida  │
│ Cert in binary      │ Medium     │ Replace cert    │
└─────────────────────┴────────────┴─────────────────┘
```

---

## 3. Non-HTTP Traffic

```
INTERCEPTING NON-HTTP PROTOCOLS:

WEBSOCKETS:
  → Burp Suite intercepts WS/WSS
  → Check Burp → WebSockets history tab
  → Modify messages in real-time

TCP/UDP (Custom protocols):
  → Wireshark for capture
  → tcpdump on device (jailbroken):
    ssh root@device
    tcpdump -i en0 -w capture.pcap
    scp capture.pcap user@mac:~/
  → Analyze in Wireshark

MQTT / XMPP / gRPC:
  → Protocol-specific proxies
  → Wireshark with protocol dissectors
  → Frida to hook socket operations

CERTIFICATE TRANSPARENCY:
  → Some apps verify CT logs
  → Additional bypass may be needed
```

---

## 4. Traffic Analysis Checklist

```
WHAT TO LOOK FOR IN CAPTURED TRAFFIC:

AUTHENTICATION:
  □ Login credentials in request body
  □ Authentication token format and strength
  □ Token storage (cookies vs headers)
  □ Token expiration and refresh mechanism
  □ Multi-factor authentication implementation

DATA EXPOSURE:
  □ PII in responses (email, phone, SSN)
  □ Excessive data in API responses
  □ Sensitive data in URL parameters
  □ Debug information in responses
  □ Stack traces or error details

API SECURITY:
  □ IDOR (change user_id in requests)
  □ Missing authorization on endpoints
  □ Rate limiting on sensitive endpoints
  □ Input validation (SQLi, XSS in API)
  □ API versioning (older versions may be vulnerable)

TRANSPORT SECURITY:
  □ HTTP requests (should be HTTPS only)
  □ TLS version (should be 1.2+)
  □ Certificate validity
  □ HSTS headers
  □ Security headers in responses

BUSINESS LOGIC:
  □ Price manipulation in purchase requests
  □ Coupon/discount bypass
  □ Feature flag manipulation
  □ Role/privilege escalation via API
```

---

## Summary Table

| Method | Tool | Jailbreak? | Covers |
|--------|------|-----------|--------|
| HTTP Proxy | Burp Suite | No | HTTP/HTTPS |
| SSL Kill Switch | Cydia tweak | Yes | Pinning bypass |
| objection | Frida-based | Yes | Pinning + analysis |
| Wireshark | Packet capture | No* | All protocols |
| tcpdump | CLI capture | Yes | All on device |
| Frida scripts | Custom hooks | Yes | Any networking |

---

## Revision Questions

1. How do you set up Burp Suite proxy for iOS traffic interception?
2. What are the main methods to bypass certificate pinning on iOS?
3. How do you intercept non-HTTP traffic from iOS apps?
4. What security issues should you look for in captured traffic?
5. Why is certificate pinning important and what happens when it's bypassable?

---

*Previous: [03-dynamic-analysis.md](03-dynamic-analysis.md) | Next: [05-jailbreak-detection-bypass.md](05-jailbreak-detection-bypass.md)*

---

*[Back to README](../README.md)*
