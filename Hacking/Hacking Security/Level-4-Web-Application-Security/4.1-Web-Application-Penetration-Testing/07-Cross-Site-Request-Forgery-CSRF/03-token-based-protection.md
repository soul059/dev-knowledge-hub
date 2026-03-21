# Unit 7: Cross-Site Request Forgery (CSRF) - Topic 3: Token-Based Protection

## Overview

Token-based protection is the most widely deployed defense against CSRF. The principle is simple: include a secret, unpredictable value in each state-changing request that the server validates. Since an attacker on a different origin cannot read responses from the target domain (Same-Origin Policy), they cannot obtain the token and therefore cannot forge valid requests.

---

## 1. Synchronizer Token Pattern

The classic CSRF defense. The server generates a unique token per session, embeds it in forms, and validates it on submission.

```
+----------------+                        +----------------+
| User Browser   |                        | Web Server     |
+----------------+                        +----------------+
       |  1. GET /form                           |
       |---------------------------------------->|
       |  2. Token generated, stored in session  |
       |     Embedded in HTML form               |
       |<----------------------------------------|
       |  3. POST /action                        |
       |     Body: data + csrf_token=a1b2c3      |
       |     Cookie: session_id=xyz789           |
       |---------------------------------------->|
       |  4. Validate: body token == session token|
       |     Match → process | Mismatch → 403    |
       |<----------------------------------------|
```

```python
# Flask implementation
import secrets
from flask import Flask, session, request, abort

app = Flask(__name__)
app.secret_key = 'your-secret-key'

@app.before_request
def generate_csrf():
    if 'csrf_token' not in session:
        session['csrf_token'] = secrets.token_hex(32)

@app.route('/transfer', methods=['POST'])
def transfer():
    token = request.form.get('csrf_token')
    if not token or token != session['csrf_token']:
        abort(403)
    return "Transfer successful"
```

```html
<form action="/transfer" method="POST">
    <input type="hidden" name="csrf_token" value="{{ csrf_token }}" />
    <input type="text" name="to_account" />
    <button type="submit">Transfer</button>
</form>
```

---

## 2. Double Submit Cookie

A **stateless** alternative — the token is sent as both a cookie and a request parameter. The server verifies both match.

```
Browser sends:
  Cookie: csrf=RANDOM_VALUE        ← automatic
  Body:   csrf_token=RANDOM_VALUE  ← embedded in form

Server checks: cookie value == body value?
```

**Why it works:** An attacker can trigger the browser to *send* the cookie, but cannot *read* it (Same-Origin Policy), so they cannot include the matching value in the body.

```javascript
// Express.js implementation
app.get('/form', (req, res) => {
    const token = require('crypto').randomBytes(32).toString('hex');
    res.cookie('csrf_token', token, { secure: true, sameSite: 'Strict' });
    res.render('form', { csrfToken: token });
});

app.post('/action', (req, res) => {
    const cookieToken = req.cookies.csrf_token;
    const bodyToken = req.body.csrf_token;
    if (!cookieToken || !bodyToken || cookieToken !== bodyToken)
        return res.status(403).json({ error: 'CSRF validation failed' });
    res.json({ success: true });
});
```

| Aspect             | Synchronizer Token       | Double Submit Cookie       |
|--------------------|--------------------------|----------------------------|
| **Server State**   | Requires session storage | Stateless                  |
| **Scalability**    | Session affinity needed  | Works across load balancers|
| **Subdomain Safe** | Yes                      | No — subdomain can set cookies |
| **Complexity**     | Moderate                 | Lower                      |

---

## 3. Token Generation and Validation

```
SECURE TOKEN PROPERTIES
+------------------------------------------------------------+
| [✓] Cryptographically random (CSPRNG)                      |
| [✓] Minimum 128 bits (32 hex characters)                   |
| [✓] Unique per session (minimum) or per request (ideal)    |
| [✓] Not derived from session ID or known values            |
| [✓] Transmitted via form field or header, NOT in URL       |
+------------------------------------------------------------+
```

**Generation across languages:**

```python
# Python
import secrets
token = secrets.token_hex(32)
```

```javascript
// Node.js
const token = require('crypto').randomBytes(32).toString('hex');
```

```java
// Java
SecureRandom random = new SecureRandom();
byte[] bytes = new byte[32];
random.nextBytes(bytes);
String token = Base64.getUrlEncoder().withoutPadding().encodeToString(bytes);
```

**Validation must use constant-time comparison** to prevent timing attacks:

```python
import hmac
def validate_csrf(provided, stored):
    if not provided or not stored:
        return False
    return hmac.compare_digest(provided, stored)
```

---

## 4. Stateless vs Stateful Tokens

**Stateful** — stored in server session; simple and reliable.

**Stateless (HMAC-based)** — self-contained, signed token requiring no server storage:

```
Token = Base64(session_id : timestamp : HMAC-SHA256(secret, session_id:timestamp))
```

```python
import hmac, hashlib, time, base64

SECRET = b'server-secret-key'

def generate_stateless_token(session_id):
    ts = str(int(time.time()))
    msg = f"{session_id}:{ts}"
    sig = hmac.new(SECRET, msg.encode(), hashlib.sha256).hexdigest()
    return base64.urlsafe_b64encode(f"{msg}:{sig}".encode()).decode()

def validate_stateless_token(token, session_id, max_age=3600):
    try:
        decoded = base64.urlsafe_b64decode(token).decode()
        sid, ts, sig = decoded.rsplit(':', 2)
        if sid != session_id or int(time.time()) - int(ts) > max_age:
            return False
        expected = hmac.new(SECRET, f"{sid}:{ts}".encode(), hashlib.sha256).hexdigest()
        return hmac.compare_digest(sig, expected)
    except Exception:
        return False
```

| Feature          | Stateful (Session)       | Stateless (HMAC)           |
|------------------|--------------------------|----------------------------|
| **Storage**      | Server-side session      | None — self-contained      |
| **Scalability**  | Requires shared session  | Horizontally scalable      |
| **Revocation**   | Delete from session      | Cannot revoke individually |
| **Best For**     | Traditional web apps     | APIs, microservices, SPAs  |

---

## 5. Framework Implementation Examples

| Framework      | Mechanism            | Usage                                       |
|----------------|----------------------|---------------------------------------------|
| **Django**     | Synchronizer token   | `{% csrf_token %}` in templates             |
| **Rails**      | Synchronizer token   | `protect_from_forgery with: :exception`     |
| **Laravel**    | Synchronizer token   | `@csrf` Blade directive                     |
| **Spring**     | Synchronizer token   | Auto-configured by Spring Security          |
| **ASP.NET**    | Antiforgery token    | `@Html.AntiForgeryToken()` + attribute      |
| **Express**    | Double submit cookie | Manual middleware (`csurf` / `csrf-csrf`)   |

---

## Summary Table

| Concept                  | Key Takeaway                                              |
|--------------------------|-----------------------------------------------------------|
| **Synchronizer Token**   | Server stores token in session; validates each request    |
| **Double Submit Cookie** | Token in cookie + body; stateless but subdomain-vulnerable|
| **Token Generation**     | Must use CSPRNG, minimum 128 bits, never predictable      |
| **Validation**           | Constant-time comparison; reject empty/null tokens        |
| **Stateful Tokens**      | Simple, reliable; requires session storage                |
| **Stateless Tokens**     | HMAC-signed; scalable, no storage needed                  |

---

## Revision Questions

1. Why is the Same-Origin Policy the foundation that makes CSRF tokens effective?
2. Under what conditions might Double Submit Cookie be vulnerable even when properly implemented?
3. Why must tokens use a CSPRNG? Describe a scenario where predictable generation is exploited.
4. What information must be in an HMAC-based stateless token, and why is a timestamp important?
5. Why should `hmac.compare_digest` be used instead of `==` for token validation?
6. Are per-request tokens necessary, or are per-session tokens sufficient? Discuss the trade-offs.

---

*Previous: [02-csrf-vs-xss.md](02-csrf-vs-xss.md) | Next: [04-samesite-cookies.md](04-samesite-cookies.md)*

---

*[Back to README](../README.md)*
