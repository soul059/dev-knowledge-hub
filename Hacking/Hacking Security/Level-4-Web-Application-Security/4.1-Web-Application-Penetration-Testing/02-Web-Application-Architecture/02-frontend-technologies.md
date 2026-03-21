# Frontend Technologies

## Unit 2: Web Application Architecture — Topic 2

## 🎯 Overview

Frontend technologies (HTML, CSS, JavaScript, frameworks) run in the user's browser and represent a significant attack surface. Client-side code is fully visible and modifiable by attackers. Understanding frontend technologies helps pentesters identify DOM-based vulnerabilities, client-side logic flaws, and exposed sensitive data in JavaScript bundles.

---

## 1. Core Frontend Technologies

```
┌──────────────────────────────────────────────────────────────┐
│                   FRONTEND STACK                              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌────────────────────────────────────────┐                  │
│  │              HTML                       │                  │
│  │  Structure & Content                    │  ← XSS, HTML    │
│  │  Forms, inputs, hidden fields           │    injection     │
│  └────────────────────────────────────────┘                  │
│  ┌────────────────────────────────────────┐                  │
│  │              CSS                        │                  │
│  │  Styling & Layout                       │  ← CSS injection,│
│  │  Animations, responsive design          │    data exfil    │
│  └────────────────────────────────────────┘                  │
│  ┌────────────────────────────────────────┐                  │
│  │           JavaScript                    │                  │
│  │  Behavior & Logic                       │  ← DOM XSS,     │
│  │  API calls, DOM manipulation            │    prototype     │
│  │  Client-side routing                    │    pollution     │
│  └────────────────────────────────────────┘                  │
│  ┌────────────────────────────────────────┐                  │
│  │        Frameworks & Libraries           │                  │
│  │  React, Angular, Vue, jQuery            │  ← Known CVEs,  │
│  │  Component-based architecture           │    template      │
│  │  State management                       │    injection     │
│  └────────────────────────────────────────┘                  │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. HTML Security Concerns

### Hidden Fields and Comments

```html
<!-- Developer comment: admin panel at /secret-admin -->
<!-- TODO: Remove debug endpoint before production -->
<!-- API Key: sk_test_abc123 -->

<form action="/transfer" method="POST">
  <input type="hidden" name="csrf_token" value="abc123">
  <input type="hidden" name="user_role" value="user">
  <!-- Change to "admin" in Burp? -->
  <input type="hidden" name="account_id" value="12345">
  <input type="hidden" name="max_amount" value="10000">
</form>
```

```bash
# Extract hidden fields
curl -s https://target.com/page | grep -i "type=\"hidden\""

# Extract HTML comments
curl -s https://target.com/page | grep -oP '<!--.*?-->'

# Check for sensitive paths in HTML
curl -s https://target.com | grep -oE '(href|src|action)="[^"]*"'
```

---

## 3. JavaScript Analysis

### Sensitive Data in JS Files

```bash
# Find API keys and secrets in JavaScript
curl -s https://target.com/app.js | grep -iE \
  "(api[_-]?key|secret|token|password|auth|credential)"

# Extract API endpoints
curl -s https://target.com/app.js | grep -oE \
  '["'"'"'](/api/[a-zA-Z0-9/_-]+)["'"'"']'

# Find hardcoded URLs
curl -s https://target.com/app.js | grep -oE \
  'https?://[a-zA-Z0-9./-]+'

# Use LinkFinder for endpoint extraction
python3 linkfinder.py -i https://target.com/app.js -o cli

# JS Beautify for minified code
js-beautify app.min.js > app.readable.js
```

### Source Map Files

```bash
# Source maps expose original (readable) source code
# Check for .map files
curl -s https://target.com/app.js.map
curl -s https://target.com/main.chunk.js.map

# Look for sourceMappingURL in JS files
curl -s https://target.com/app.js | grep sourceMappingURL

# Extract original source from maps
# Use tools like unwebpack-sourcemap
```

---

## 4. DOM-Based Vulnerabilities

```javascript
// DOM XSS — user input flows into dangerous sinks
// Sources (user-controlled data):
//   document.URL, document.location, document.referrer
//   window.name, location.hash, location.search

// Sinks (dangerous operations):
//   eval(), innerHTML, document.write(), 
//   setTimeout(), setInterval(), Function()

// Vulnerable example:
var search = location.hash.substring(1);
document.getElementById('result').innerHTML = search;
// URL: https://target.com#<img src=x onerror=alert(1)>

// Prototype pollution
// Object.prototype is modified via user input
const obj = {};
obj.__proto__.isAdmin = true;  
// Now ALL objects have isAdmin = true
```

---

## 5. Frontend Frameworks Security

| Framework | Common Vulnerabilities | Testing Focus |
|-----------|----------------------|---------------|
| React | `dangerouslySetInnerHTML`, SSR XSS | Props injection |
| Angular | Template injection, bypass sanitizer | `bypassSecurityTrust*` |
| Vue.js | `v-html` directive, template injection | Server-side rendering |
| jQuery | `.html()`, `.append()` with user input | DOM manipulation sinks |
| Handlebars | Triple-stache `{{{ }}}` bypass | Template injection |

```javascript
// React: Dangerous HTML rendering
<div dangerouslySetInnerHTML={{__html: userInput}} />

// Angular: Template injection
// If user controls template: {{constructor.constructor('alert(1)')()}}

// Vue: Unsafe HTML rendering
<div v-html="userContent"></div>
```

---

## 6. Client-Side Storage

```
┌──────────────────────────────────────────────────────────────┐
│              CLIENT-SIDE STORAGE MECHANISMS                    │
├──────────────┬────────────┬───────────────┬─────────────────┤
│  Mechanism   │  Capacity  │  Accessible   │  Security Risk  │
├──────────────┼────────────┼───────────────┼─────────────────┤
│  Cookies     │  4KB       │  Server + JS  │  Session theft  │
│  localStorage│  5-10MB    │  JS only      │  XSS data theft │
│  sessionStorage│ 5-10MB   │  JS only      │  XSS data theft │
│  IndexedDB   │  Unlimited │  JS only      │  Data exposure  │
│  Web SQL     │  Unlimited │  JS only      │  SQL injection  │
└──────────────┴────────────┴───────────────┴─────────────────┘
```

```javascript
// Inspect storage in browser console
console.log(localStorage);
console.log(sessionStorage);

// Common findings:
// - JWT tokens in localStorage (XSS can steal them)
// - User profile data
// - API keys
// - Cached sensitive responses
```

---

## 📊 Summary Table

| Technology | Attack Surface | Key Checks |
|-----------|---------------|------------|
| HTML | Hidden fields, comments, forms | View source, inspect elements |
| CSS | Data exfiltration, UI redressing | CSS injection testing |
| JavaScript | DOM XSS, exposed secrets, logic | JS file analysis, source maps |
| Frameworks | Framework-specific vulns | Version detection, known CVEs |
| Storage | Tokens, PII in localStorage | Console inspection |

---

## ❓ Revision Questions

1. Why is client-side validation never sufficient for security?
2. How can source map files expose sensitive application code?
3. What are DOM XSS sources and sinks?
4. Why is storing JWT tokens in localStorage risky?
5. How do you find hidden API endpoints in JavaScript bundles?
6. What framework-specific vulnerabilities should a pentester look for?

---

*Previous: [01-client-server-model.md](01-client-server-model.md) | Next: [03-backend-technologies.md](03-backend-technologies.md)*

---

*[Back to README](../README.md)*
