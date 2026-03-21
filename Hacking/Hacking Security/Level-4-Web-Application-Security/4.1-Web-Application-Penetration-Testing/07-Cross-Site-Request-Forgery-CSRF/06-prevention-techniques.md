# Unit 7: Cross-Site Request Forgery (CSRF) - Topic 6: Prevention Techniques

## Overview

Effective CSRF prevention requires a layered defense strategy. No single mechanism is foolproof — modern applications should combine token-based protections, SameSite cookies, header validation, and proper CORS configuration. This topic covers framework-specific implementations, custom header strategies, origin validation, defense-in-depth architecture, and CORS security.

---

## 1. Framework-Specific CSRF Protection

| Framework      | Default CSRF | Mechanism          | Usage                                     |
|----------------|--------------|--------------------|-------------------------------------------|
| **Django**     | Enabled      | Synchronizer token | `{% csrf_token %}` in templates           |
| **Rails**      | Enabled      | Synchronizer token | `protect_from_forgery` in controller      |
| **Laravel**    | Enabled      | Synchronizer token | `@csrf` Blade directive                   |
| **Spring MVC** | Enabled      | Synchronizer token | Auto-configured by Spring Security        |
| **ASP.NET**    | Opt-in       | Antiforgery token  | `[ValidateAntiForgeryToken]` attribute    |
| **Express.js** | Not built-in | Requires middleware| Manual setup with csurf/csrf-csrf         |

### Django Implementation

```python
# settings.py
CSRF_COOKIE_SECURE = True
CSRF_COOKIE_SAMESITE = 'Lax'
CSRF_TRUSTED_ORIGINS = ['https://trusted-partner.com']
```

```html
<form method="POST" action="/transfer/">
    {% csrf_token %}
    <input type="text" name="amount" />
    <button type="submit">Transfer</button>
</form>
```

```javascript
// Django AJAX — read token from cookie
fetch('/api/action/', {
    method: 'POST',
    headers: {
        'X-CSRFToken': document.cookie.match(/csrftoken=([^;]+)/)[1],
        'Content-Type': 'application/json',
    },
    body: JSON.stringify({ key: 'value' })
});
```

### Spring Security

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.csrf(csrf -> csrf
            .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
        );
        return http.build();
    }
}
```

---

## 2. Custom Header Validation

Custom headers trigger CORS preflight checks, blocking cross-origin requests unless explicitly allowed.

```
Legitimate (same-origin):               Cross-origin attack:
POST /api/transfer                      POST /api/transfer
X-Requested-With: XMLHttpRequest ✓      (no custom header) ✗
Cookie: session=abc                     Cookie: session=abc
→ Server: header present → ALLOW        → Server: header missing → 403
```

```javascript
// Server-side (Express.js)
function csrfHeaderCheck(req, res, next) {
    if (['POST','PUT','DELETE','PATCH'].includes(req.method)) {
        if (!req.headers['x-requested-with']) {
            return res.status(403).json({ error: 'Missing CSRF header' });
        }
    }
    next();
}
app.use(csrfHeaderCheck);

// Client-side — include header in all requests
fetch('/api/action', {
    method: 'POST',
    headers: {
        'X-Requested-With': 'XMLHttpRequest',
        'Content-Type': 'application/json'
    },
    body: JSON.stringify(data)
});
```

**Limitation:** Only works for AJAX endpoints (not HTML forms). Requires CORS to NOT allow the custom header from untrusted origins.

---

## 3. Origin/Referer Header Checking

Validate request origin to ensure it matches expected domains:

```python
ALLOWED_ORIGINS = {'https://myapp.com', 'https://www.myapp.com'}

@app.before_request
def validate_origin():
    if request.method in ('POST', 'PUT', 'DELETE', 'PATCH'):
        origin = request.headers.get('Origin')
        referer = request.headers.get('Referer')
        if origin:
            if origin not in ALLOWED_ORIGINS:
                abort(403, 'Invalid origin')
        elif referer:
            from urllib.parse import urlparse
            ref_origin = f"{urlparse(referer).scheme}://{urlparse(referer).netloc}"
            if ref_origin not in ALLOWED_ORIGINS:
                abort(403, 'Invalid referer')
        else:
            abort(403, 'Missing origin information')
```

```
Validation Flow:
                Origin present?
                /            \
              YES              NO
              /                  \
         Origin in           Referer present?
         whitelist?          /            \
         /      \          YES              NO
       YES      NO        /                  \
        |        |    Referer origin       REJECT
      ALLOW   REJECT  in whitelist?
                       /      \
                     YES      NO → REJECT
                      |
                    ALLOW
```

| Aspect              | Origin Header           | Referer Header              |
|----------------------|-------------------------|-----------------------------|
| **Reliability**      | High — always on POST   | May be stripped by privacy   |
| **Information**      | Scheme + host + port    | Full URL (privacy concern)  |
| **Null Origin Risk** | Sandboxed iframes, redirects | N/A                     |

> **Never whitelist `null`** as a valid origin — sandboxed iframes and data: URLs send `Origin: null`.

---

## 4. Defense-in-Depth Approach

```
DEFENSE LAYERS
+==============================================================+
| Layer 1: SameSite Cookies (Browser Level)                    |
|   → Set Lax/Strict on session cookies                        |
+==============================================================+
| Layer 2: CSRF Tokens (Application Level)                     |
|   → Synchronizer token or double-submit cookie               |
+==============================================================+
| Layer 3: Origin Validation (Request Level)                   |
|   → Check Origin header; fall back to Referer                |
+==============================================================+
| Layer 4: Custom Headers (API Level)                          |
|   → Require X-Requested-With for AJAX endpoints              |
+==============================================================+
| Layer 5: CORS Configuration (Transport Level)                |
|   → Strict whitelist, no reflected origins with credentials  |
+==============================================================+
| Layer 6: Re-authentication (Critical Actions)                |
|   → Password/MFA for transfers, password changes             |
+==============================================================+
```

| Priority | Defense              | Effort | Impact   | Coverage              |
|----------|----------------------|--------|----------|-----------------------|
| 1        | SameSite Cookies     | Low    | High     | All cookie-based apps |
| 2        | Framework CSRF Tokens| Low    | High     | All form endpoints    |
| 3        | Origin Validation    | Medium | Medium   | All state-changing ops|
| 4        | Strict CORS          | Medium | High     | API endpoints         |
| 5        | Custom Headers       | Low    | Medium   | AJAX/API endpoints    |
| 6        | Re-authentication    | High   | Critical | High-value operations |

---

## 5. CORS Configuration

### Secure Setup

```javascript
const corsOptions = {
    origin: function(origin, callback) {
        const whitelist = ['https://myapp.com', 'https://admin.myapp.com'];
        if (!origin || whitelist.includes(origin))
            callback(null, true);
        else
            callback(new Error('CORS violation'));
    },
    credentials: true,
    methods: ['GET','POST','PUT','DELETE'],
    allowedHeaders: ['Content-Type','X-Requested-With','X-CSRF-Token'],
    maxAge: 86400
};
app.use(cors(corsOptions));
```

### Dangerous Misconfigurations

```
❌ Reflected Origin: Access-Control-Allow-Origin: [from request]
   + Access-Control-Allow-Credentials: true
   → Attacker origin trusted; full CSRF + data theft

❌ Null Origin: Access-Control-Allow-Origin: null
   → Sandboxed iframes exploit this

❌ Broad subdomain match: if origin.endsWith('.myapp.com')
   → attacker registers evil-myapp.com

✅ Strict whitelist: Access-Control-Allow-Origin: https://myapp.com
```

```bash
# Test for reflected origin
curl -H "Origin: https://evil.com" -I https://target.com/api/data 2>&1 \
  | grep -i "access-control"

# Test for null origin
curl -H "Origin: null" -I https://target.com/api/data 2>&1 \
  | grep -i "access-control"
```

---

## 6. Prevention Checklist

```
[ ] Set SameSite=Lax (minimum) on all session cookies
[ ] Use framework-provided CSRF tokens on all forms
[ ] Validate Origin header on state-changing requests
[ ] Configure strict CORS (whitelist, no reflected origins)
[ ] Require custom header for AJAX/API endpoints
[ ] Never change state via GET requests
[ ] Require re-authentication for critical operations
[ ] Set Secure and HttpOnly on session cookies
[ ] Log and monitor CSRF validation failures
[ ] Audit subdomain security (compromised subdomain risk)
```

---

## Summary Table

| Concept                    | Key Takeaway                                               |
|----------------------------|------------------------------------------------------------|
| **Framework Protection**   | Use built-in CSRF; Django/Rails/Laravel/Spring default on  |
| **Custom Headers**         | Effective for AJAX; relies on CORS preflight enforcement   |
| **Origin Validation**      | Check Origin header; never whitelist "null"                |
| **Defense in Depth**       | Combine SameSite + tokens + headers + CORS + re-auth      |
| **CORS Dangers**           | Reflected origin with credentials = full bypass            |
| **Re-authentication**      | Required for high-value operations                         |

---

## Revision Questions

1. Compare Django, Rails, and Spring CSRF protection. Which is most comprehensive out-of-the-box?
2. Why does requiring `X-Requested-With` provide CSRF protection? Under what CORS misconfig does it fail?
3. A developer allows requests when neither Origin nor Referer is present. How can an attacker exploit this?
4. Design a defense-in-depth strategy for a banking app with both traditional forms and a SPA API.
5. An app's CORS reflects the request Origin with `Allow-Credentials: true`. Explain the severity and attack.
6. Why should `null` never be whitelisted in CORS? Provide two scenarios where the browser sends `Origin: null`.

---

*Previous: [05-testing-methodology.md](05-testing-methodology.md)*

---

*[Back to README](../README.md)*
