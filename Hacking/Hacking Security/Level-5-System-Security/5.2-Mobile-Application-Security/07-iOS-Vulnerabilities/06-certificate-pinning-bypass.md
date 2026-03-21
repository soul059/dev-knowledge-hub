# Unit 7: iOS Vulnerabilities — Topic 6: Certificate Pinning Bypass

## Overview

**Certificate pinning** (SSL pinning) ensures that an iOS app only trusts specific certificates or public keys when communicating with its backend server, rather than trusting any certificate signed by a trusted CA. Bypassing pinning is essential for intercepting HTTPS traffic during security assessments. This topic covers pinning implementations, bypass techniques, and reporting guidance.

---

## 1. How Certificate Pinning Works

```
WITHOUT PINNING:
  App → System Trust Store → Any valid CA cert accepted
  RISK: Attacker with CA cert (Burp) can MitM

WITH PINNING:
  App → Hardcoded pin → Only specific cert/key accepted
  PROTECTION: Even with CA cert installed, proxy rejected

PINNING TYPES:

CERTIFICATE PINNING:
  → Pin the entire server certificate
  → Must update app when cert rotates
  → More brittle, harder to maintain
  
PUBLIC KEY PINNING:
  → Pin the server's public key (or its hash)
  → Key stays same even if cert is renewed
  → Preferred approach (more flexible)

SUBJECT PUBLIC KEY INFO (SPKI) PINNING:
  → Pin hash of SubjectPublicKeyInfo structure
  → Most common modern implementation
  → Used by TrustKit, OkHttp, etc.

PINNING LOCATIONS:
  → App code (NSURLSession delegate)
  → Third-party libraries (AFNetworking, Alamofire, TrustKit)
  → Network Security Config (Android equivalent)
  → Embedded certificate files in bundle
```

---

## 2. Common Pinning Implementations

```swift
// NSURLSession delegate pinning
func urlSession(_ session: URLSession,
    didReceive challenge: URLAuthenticationChallenge,
    completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void) {
    
    guard let serverTrust = challenge.protectionSpace.serverTrust,
          let serverCert = SecTrustGetCertificateAtIndex(serverTrust, 0) else {
        completionHandler(.cancelAuthenticationChallenge, nil)
        return
    }
    
    // Get server cert data
    let serverCertData = SecCertificateCopyData(serverCert) as Data
    
    // Compare with pinned cert
    if let pinnedCertPath = Bundle.main.path(forResource: "server", ofType: "cer"),
       let pinnedCertData = try? Data(contentsOf: URL(fileURLWithPath: pinnedCertPath)) {
        if serverCertData == pinnedCertData {
            completionHandler(.useCredential, URLCredential(trust: serverTrust))
            return
        }
    }
    completionHandler(.cancelAuthenticationChallenge, nil)
}
```

```
THIRD-PARTY IMPLEMENTATIONS:

TRUSTKIT:
  → Popular iOS pinning library
  → Declarative configuration
  → Reporting on pin violations
  → Supports pin rotation

ALAMOFIRE:
  → Built-in ServerTrustEvaluating
  → PinnedCertificatesTrustEvaluator
  → PublicKeysTrustEvaluator

AFNETWORKING:
  → AFSecurityPolicy
  → pinnedCertificates mode
  → validatesCertificateChain
```

---

## 3. Bypass Techniques

```
BYPASS METHODS (Easiest to Hardest):

1. SSL KILL SWITCH 2 (Easiest):
   → Jailbreak tweak from Cydia/Sileo
   → Globally disables SSL validation
   → Works for most standard implementations
   → Toggle in Settings → SSL Kill Switch 2

2. OBJECTION:
   objection -g com.example.app explore
   > ios sslpinning disable
   → Hooks common pinning methods:
     • NSURLSession delegate methods
     • AFNetworking/AFSecurityPolicy
     • Alamofire ServerTrustEvaluating
     • TrustKit validation

3. FRIDA SCRIPTS:
```

```javascript
// Frida: Comprehensive SSL pinning bypass

// Hook NSURLSession challenge handling
var resolver = new ApiResolver('objc');
resolver.enumerateMatches(
    '-[* URLSession:didReceiveChallenge:completionHandler:]', {
    onMatch: function(match) {
        Interceptor.attach(match.address, {
            onEnter: function(args) {
                var completionHandler = new ObjC.Block(args[4]);
                completionHandler.implementation = function(disposition, credential) {
                    // UseCredential = 0
                    var serverTrust = ObjC.Object(args[3])
                        .protectionSpace().serverTrust();
                    var cred = ObjC.classes.NSURLCredential
                        .credentialForTrust_(serverTrust);
                    completionHandler(0, cred);
                };
            }
        });
    },
    onComplete: function() {}
});

// Hook TrustKit
if (ObjC.classes.TSKPinningValidator) {
    Interceptor.attach(
        ObjC.classes.TSKPinningValidator[
            '- evaluateTrust:forHostname:'].implementation, {
        onLeave: function(retval) {
            retval.replace(0); // TSKTrustDecisionShouldAllowConnection
        }
    });
}

// Hook AFNetworking
if (ObjC.classes.AFSecurityPolicy) {
    Interceptor.attach(
        ObjC.classes.AFSecurityPolicy[
            '- evaluateServerTrust:forDomain:'].implementation, {
        onLeave: function(retval) {
            retval.replace(0x1); // YES
        }
    });
}

// Hook SecTrustEvaluateWithError (iOS 12+)
Interceptor.attach(
    Module.findExportByName('Security', 'SecTrustEvaluateWithError'), {
    onLeave: function(retval) {
        retval.replace(0x1); // true = trusted
    }
});

// Hook SSLHandshake for lower-level bypass
Interceptor.attach(
    Module.findExportByName('libsystem_coretls.dylib', 'tls_handshake_set_callbacks'), {
    onEnter: function(args) {
        // Override verification callback
    }
});
```

```
4. BINARY PATCHING (Hard):
   → Disassemble with Ghidra/Hopper
   → Find certificate comparison function
   → Patch conditional branch to unconditional
   → Re-sign and deploy modified binary
   → Permanent modification (survives restart)

5. CERTIFICATE REPLACEMENT (Medium):
   → Find pinned certificate in app bundle
   → Replace with Burp's CA certificate
   → Re-sign app
   → App now pins to your proxy cert
```

---

## 4. Troubleshooting Pinning Bypass

```
WHEN BYPASS DOESN'T WORK:

CHECK:
  □ Burp CA certificate installed AND trusted?
  □ SSL Kill Switch / objection actually running?
  □ App using custom networking library?
  □ Pinning in native code (.framework, .dylib)?
  □ Multiple pinning checks (redundant)?
  □ Certificate Transparency enforcement?

SOLUTIONS:
  → Try multiple bypass methods simultaneously
  → Monitor with Frida which methods are called
  → Check for custom SSLContext/SecTrust usage
  → Look for pinning in embedded frameworks
  → Patch binary if all runtime methods fail

DEBUGGING:
  // Log all SSL-related method calls
  frida-trace -U -f com.app -m "*[*SSL*]*" -m "*[*Trust*]*" -m "*[*Certificate*]*"
  // Shows exactly which methods handle TLS
```

---

## Summary Table

| Bypass Method | Difficulty | Requires Jailbreak | Success Rate |
|--------------|-----------|-------------------|-------------|
| SSL Kill Switch 2 | Easy | Yes | ~70% |
| objection | Easy | Yes | ~75% |
| Frida script | Medium | Yes | ~85% |
| Binary patching | Hard | No (re-sign) | ~95% |
| Cert replacement | Medium | No (re-sign) | ~80% |

---

## Revision Questions

1. What is the difference between certificate pinning and public key pinning?
2. How does SSL Kill Switch 2 bypass pinning?
3. What Frida hooks are needed for comprehensive pinning bypass?
4. When would binary patching be necessary over runtime bypass?
5. How do you troubleshoot when pinning bypass doesn't work?

---

*Previous: [05-binary-analysis.md](05-binary-analysis.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*
