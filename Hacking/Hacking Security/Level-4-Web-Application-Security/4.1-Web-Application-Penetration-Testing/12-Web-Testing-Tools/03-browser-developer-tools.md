# Unit 12: Web Testing Tools - Topic 3: Browser Developer Tools

## Overview

Browser Developer Tools (DevTools) are built-in debugging and analysis tools available in every modern web browser. For web application penetration testers, DevTools provide **instant access** to the client-side attack surface — including DOM manipulation, JavaScript debugging, network traffic analysis, cookie/storage inspection, and security header review. DevTools are often the **first tool** used during web testing, even before setting up a proxy like Burp Suite.

---

## 1. Opening and Navigating DevTools

### Access Methods

```
Opening DevTools:
┌─────────────────────────────────────────┐
│ Chrome / Edge:                          │
│   F12 or Ctrl+Shift+I (Cmd+Opt+I Mac)  │
│   Right-click → Inspect                │
│                                         │
│ Firefox:                                │
│   F12 or Ctrl+Shift+I                   │
│   Right-click → Inspect Element        │
│                                         │
│ Safari:                                 │
│   Cmd+Opt+I (enable in Preferences →   │
│   Advanced → Show Develop menu)         │
└─────────────────────────────────────────┘
```

### DevTools Panels for Security Testing

| Panel | Security Use |
|-------|-------------|
| **Elements** | DOM inspection, hidden fields, client-side controls |
| **Console** | JavaScript execution, XSS testing, DOM manipulation |
| **Network** | Request/response analysis, headers, cookies, API calls |
| **Sources** | JavaScript debugging, breakpoints, source maps |
| **Application** | Cookies, localStorage, sessionStorage, IndexedDB |
| **Security** | TLS certificates, mixed content, origin details |
| **Performance** | Timing analysis |
| **Lighthouse** | Security audit checks |

---

## 2. Elements Panel — DOM Inspection

### Finding Hidden Elements

```html
<!-- Look for hidden form fields -->
<input type="hidden" name="user_role" value="user">
<!-- Change value="user" to value="admin" via DevTools! -->

<input type="hidden" name="price" value="49.99">
<!-- Change price before form submission! -->

<input type="hidden" name="csrf_token" value="abc123">
<!-- Extract CSRF token for scripting -->

<!-- Look for disabled fields -->
<input type="text" name="email" value="admin@target.com" disabled>
<!-- Remove 'disabled' attribute → field becomes editable -->

<!-- Look for client-side validation -->
<input type="email" name="email" required maxlength="50" pattern="[a-z]+">
<!-- Remove validation attributes to bypass client-side checks -->

<!-- Look for hidden divs/sections -->
<div id="admin-panel" style="display: none;">
  <a href="/admin/users">Manage Users</a>
  <a href="/admin/config">Configuration</a>
</div>
<!-- Change display:none to display:block → reveals admin links! -->
```

### DOM Manipulation for Security Testing

```javascript
// In Console panel:

// Remove all client-side validation
document.querySelectorAll('[required]').forEach(el => el.removeAttribute('required'));
document.querySelectorAll('[maxlength]').forEach(el => el.removeAttribute('maxlength'));
document.querySelectorAll('[pattern]').forEach(el => el.removeAttribute('pattern'));
document.querySelectorAll('[disabled]').forEach(el => el.removeAttribute('disabled'));

// Reveal hidden elements
document.querySelectorAll('[type="hidden"]').forEach(el => {
    el.type = 'text';
    console.log(`${el.name}: ${el.value}`);
});

// Show all hidden divs
document.querySelectorAll('[style*="display: none"], [style*="display:none"], .hidden, .d-none').forEach(el => {
    el.style.display = 'block';
    el.classList.remove('hidden', 'd-none');
});

// Find all links (including hidden)
document.querySelectorAll('a[href]').forEach(a => console.log(a.href));

// Find all forms and their actions
document.querySelectorAll('form').forEach(f => {
    console.log(`Form: ${f.action} Method: ${f.method}`);
    f.querySelectorAll('input').forEach(i => console.log(`  ${i.name}: ${i.value} (${i.type})`));
});

// Monitor form submissions
document.querySelectorAll('form').forEach(f => {
    f.addEventListener('submit', e => {
        e.preventDefault();
        const data = new FormData(f);
        for (let [key, val] of data) console.log(`${key}: ${val}`);
        // f.submit(); // uncomment to actually submit
    });
});
```

---

## 3. Console Panel — JavaScript Execution

### Security-Relevant Console Commands

```javascript
// Extract all cookies
console.log(document.cookie);

// Extract specific cookie
console.log(document.cookie.split(';').find(c => c.includes('session')));

// Set cookies
document.cookie = "admin=true; path=/";
document.cookie = "role=administrator; path=/";

// Access localStorage
console.log(JSON.stringify(localStorage));
for (let i = 0; i < localStorage.length; i++) {
    const key = localStorage.key(i);
    console.log(`${key}: ${localStorage.getItem(key)}`);
}

// Access sessionStorage
for (let i = 0; i < sessionStorage.length; i++) {
    const key = sessionStorage.key(i);
    console.log(`${key}: ${sessionStorage.getItem(key)}`);
}

// Find JWT tokens in storage
[localStorage, sessionStorage].forEach(storage => {
    for (let i = 0; i < storage.length; i++) {
        const val = storage.getItem(storage.key(i));
        if (val && val.startsWith('eyJ')) {
            console.log(`JWT found in ${storage === localStorage ? 'local' : 'session'}Storage:`);
            console.log(`Key: ${storage.key(i)}`);
            // Decode JWT
            const parts = val.split('.');
            console.log('Header:', JSON.parse(atob(parts[0])));
            console.log('Payload:', JSON.parse(atob(parts[1])));
        }
    }
});

// Test XSS payloads
document.body.innerHTML += '<img src=x onerror="alert(document.domain)">';

// Hook XMLHttpRequest to monitor API calls
const originalXHR = XMLHttpRequest.prototype.open;
XMLHttpRequest.prototype.open = function(method, url) {
    console.log(`XHR: ${method} ${url}`);
    return originalXHR.apply(this, arguments);
};

// Hook fetch to monitor API calls
const originalFetch = window.fetch;
window.fetch = function(url, options) {
    console.log(`Fetch: ${options?.method || 'GET'} ${url}`);
    if (options?.body) console.log('Body:', options.body);
    return originalFetch.apply(this, arguments);
};

// Override JavaScript functions
// Bypass client-side checks
window.isAdmin = () => true;
window.checkPermission = () => true;
window.validateInput = () => true;
```

---

## 4. Network Panel — Traffic Analysis

### Analyzing Requests and Responses

```
Network Panel Features:
┌─────────────────────────────────────────────────────────┐
│ Filter bar:  [XHR] [JS] [CSS] [Img] [Media] [Doc]      │
│                                                         │
│ Name         │ Status │ Type │ Size  │ Time │ Initiator │
│ ─────────────┼────────┼──────┼───────┼──────┼───────────│
│ /api/login   │ 200    │ xhr  │ 1.2KB │ 50ms │ app.js:42│
│ /api/users   │ 200    │ xhr  │ 15KB  │ 120ms│ app.js:88│
│ /api/admin   │ 403    │ xhr  │ 0.1KB │ 30ms │ app.js:95│
│ style.css    │ 200    │ css  │ 45KB  │ 200ms│ index    │
└─────────────────────────────────────────────────────────┘
```

### Security Analysis via Network Tab

```
Key things to check:

1. API Endpoints
   - Filter by "XHR" or "Fetch" to see API calls
   - Note all endpoints, methods, parameters
   - Look for admin/internal endpoints

2. Authentication Headers
   - Click request → Headers tab
   - Look for: Authorization, Cookie, X-API-Key
   - Copy tokens for use in other tools

3. Response Headers (Security)
   - Strict-Transport-Security
   - Content-Security-Policy
   - X-Content-Type-Options
   - X-Frame-Options
   - Access-Control-Allow-Origin (CORS)

4. Sensitive Data in Responses
   - Search responses for: password, token, key, secret
   - Check for excessive data exposure
   - Look for stack traces in error responses

5. Mixed Content
   - HTTP resources loaded on HTTPS page
   - Insecure form submissions

6. Copy as curl/fetch
   - Right-click request → Copy → Copy as cURL
   - Paste in terminal to replay request
   - Useful for quick testing outside browser
```

### Network Throttling for Testing

```
Chrome: Network tab → Throttling dropdown
- Fast 3G, Slow 3G, Offline
- Custom profiles for testing timeout behavior

Use Case: Test how app handles slow responses
- Do tokens expire during slow connections?
- Are there timeout-based vulnerabilities?
- Does the app handle offline gracefully?
```

---

## 5. Application Panel — Storage Analysis

### Cookie Analysis

```
Application tab → Cookies → domain

Check each cookie for:
┌──────────┬─────────────┬──────────────────────────────────┐
│ Flag     │ Expected    │ Security Impact if Missing        │
├──────────┼─────────────┼──────────────────────────────────┤
│ HttpOnly │ ✓ Set       │ XSS can steal session cookie     │
│ Secure   │ ✓ Set       │ Cookie sent over HTTP (MitM)     │
│ SameSite │ Strict/Lax  │ CSRF attacks possible            │
│ Path     │ Restricted  │ Cookie scope too broad           │
│ Domain   │ Specific    │ Subdomain access                 │
│ Expires  │ Reasonable  │ Persistent sessions              │
└──────────┴─────────────┴──────────────────────────────────┘

// Quick cookie audit in Console:
document.cookie.split(';').forEach(c => {
    console.log(c.trim());
    // If you can read it, HttpOnly is NOT set
});
```

### Local Storage and Session Storage

```javascript
// Check for sensitive data in storage
// Application tab → Local Storage / Session Storage

// Common sensitive items found:
// - JWT tokens
// - API keys
// - User PII
// - Session data
// - Authentication state
// - Feature flags

// Security concerns:
// - localStorage persists after browser close
// - Any JavaScript (including XSS) can access storage
// - No HttpOnly equivalent for localStorage
// - Data not automatically sent with requests (unlike cookies)

// Quick audit:
console.log("=== Local Storage ===");
for (let i = 0; i < localStorage.length; i++) {
    const key = localStorage.key(i);
    const value = localStorage.getItem(key);
    if (value.length > 100) {
        console.log(`${key}: ${value.substring(0, 100)}...`);
    } else {
        console.log(`${key}: ${value}`);
    }
}
```

---

## 6. Sources Panel — JavaScript Debugging

### Setting Breakpoints for Security

```
Security-relevant debugging:

1. XHR/Fetch Breakpoints
   Sources → XHR/fetch Breakpoints → Add
   - Break on URL pattern: "/api/admin"
   - Inspect request before it's sent
   - Modify request headers/body

2. Event Listener Breakpoints
   Sources → Event Listener Breakpoints
   - ✓ Submit (catch form submissions)
   - ✓ XHR → readystatechange
   - ✓ Clipboard (catch copy/paste restrictions)

3. DOM Breakpoints
   Elements → Right-click element → Break on
   - Subtree modifications
   - Attribute modifications
   - Node removal
   
   Use to catch dynamic DOM manipulation

4. Conditional Breakpoints
   Right-click line number → Add conditional breakpoint
   - Break only when condition is true
   - Example: break when role === 'admin'

5. Source Maps
   - If source maps available, see original (unminified) code
   - Search for: API keys, endpoints, credentials, logic
   - Cmd+Shift+O to search by function name
```

### Finding Secrets in JavaScript

```
Sources → Search (Cmd+Shift+F):

Search for:
- "api_key"
- "apiKey"
- "secret"
- "password"
- "token"
- "bearer"
- "admin"
- "authorization"
- "internal"
- "debug"
- "test"
- "staging"
- "https://" (find API endpoints)
- "aws" or "s3" (cloud resources)

# Using Console for source analysis:
// Find all script sources
document.querySelectorAll('script[src]').forEach(s => console.log(s.src));

// Fetch and search all inline scripts
document.querySelectorAll('script:not([src])').forEach(s => {
    if (s.textContent.match(/api|key|secret|password|token/i)) {
        console.log('Sensitive script found:', s.textContent.substring(0, 200));
    }
});
```

---

## 7. Security Panel

### TLS/SSL Analysis

```
Security Panel (Chrome):
- Certificate details (issuer, validity, algorithm)
- Connection info (TLS version, cipher suite)
- Certificate chain
- Mixed content warnings

Check for:
□ TLS 1.2+ (not TLS 1.0/1.1)
□ Strong cipher suites
□ Valid certificate (not self-signed in production)
□ Correct domain name on certificate
□ No mixed content (HTTP resources on HTTPS page)
□ HSTS header present
```

---

## 8. DevTools Tips for Pentesters

### Quick Reference Commands

```javascript
// === RECONNAISSANCE ===
// List all JavaScript files loaded
performance.getEntriesByType('resource')
    .filter(r => r.name.endsWith('.js'))
    .forEach(r => console.log(r.name));

// List all external domains
[...new Set(
    performance.getEntriesByType('resource')
    .map(r => new URL(r.name).hostname)
)].forEach(d => console.log(d));

// === COOKIE MANIPULATION ===
// Decode JWT from cookie
const jwt = document.cookie.split(';')
    .find(c => c.includes('token'))?.split('=')[1];
if (jwt) {
    const [h, p] = jwt.split('.').slice(0,2).map(x => JSON.parse(atob(x)));
    console.log('Header:', h, 'Payload:', p);
}

// === FORM MANIPULATION ===
// Auto-fill and submit forms with custom data
const form = document.querySelector('form');
form.querySelector('[name="username"]').value = "admin' OR '1'='1";
form.querySelector('[name="password"]').value = "test";
// form.submit();

// === MONITORING ===
// Log all form submissions
new MutationObserver((mutations) => {
    mutations.forEach(m => {
        m.addedNodes.forEach(node => {
            if (node.tagName === 'FORM') {
                console.log('New form added:', node.action);
            }
        });
    });
}).observe(document.body, {childList: true, subtree: true});
```

### Browser-Specific Features

| Feature | Chrome | Firefox |
|---------|--------|---------|
| **Network search** | Ctrl+F in Network | Filter in Network |
| **Copy as cURL** | ✅ Right-click request | ✅ Right-click request |
| **Edit and resend** | ❌ (use Copy as fetch) | ✅ Edit and Resend button |
| **Service Workers** | Application tab | Application tab |
| **WebSocket inspect** | Network → WS filter | Network → WS filter |
| **Responsive mode** | Ctrl+Shift+M | Ctrl+Shift+M |
| **Screenshot** | Ctrl+Shift+P → Screenshot | Settings → Screenshot |

---

## Summary Table

| Panel | Security Use |
|-------|-------------|
| **Elements** | Find hidden fields, remove validation, reveal hidden UI |
| **Console** | Execute JavaScript, test XSS, extract tokens, monitor APIs |
| **Network** | Analyze API calls, check headers, find sensitive responses |
| **Sources** | Debug JavaScript, set breakpoints, find secrets in code |
| **Application** | Audit cookies (flags), check storage for sensitive data |
| **Security** | Verify TLS, check certificates, find mixed content |

---

## Revision Questions

1. How would you use DevTools to bypass client-side form validation? List three specific techniques.
2. Describe how to use the Console to monitor all API calls made by a single-page application.
3. What security-relevant information can you find in the Application panel? What should you check for in cookies?
4. How would you use the Sources panel to find API keys or sensitive data in JavaScript files?
5. Explain the security implications of storing JWT tokens in localStorage vs httpOnly cookies.
6. How would you use the Network panel to identify excessive data exposure in API responses?

---

*Previous: [02-owasp-zap.md](02-owasp-zap.md) | Next: [04-automated-scanners.md](04-automated-scanners.md)*

---

*[Back to README](../README.md)*
