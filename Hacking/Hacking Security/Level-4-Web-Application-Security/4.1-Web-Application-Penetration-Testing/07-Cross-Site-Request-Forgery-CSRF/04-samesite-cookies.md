# Unit 7: Cross-Site Request Forgery (CSRF) - Topic 4: SameSite Cookies

## Overview

The SameSite cookie attribute controls whether cookies are sent with cross-site requests. Since 2020, major browsers default to `SameSite=Lax`, significantly reducing the CSRF attack surface at the browser level. This topic covers the three SameSite values, browser behaviors, limitations, known bypasses, and configuration best practices.

---

## 1. SameSite Attribute Values

### Behavior Matrix

```
Cross-Site Request Type:    Strict    Lax       None
─────────────────────────   ──────    ───       ────
Top-level GET (link click)  BLOCKED   ALLOWED   ALLOWED
Form POST                   BLOCKED   BLOCKED   ALLOWED
Iframe load                 BLOCKED   BLOCKED   ALLOWED
AJAX/fetch                  BLOCKED   BLOCKED   ALLOWED
Image/script tag            BLOCKED   BLOCKED   ALLOWED
```

**Strict** — cookies never sent cross-site. Strongest protection but breaks external link login:
```http
Set-Cookie: session=abc123; SameSite=Strict; Secure; HttpOnly
```

**Lax** — cookies sent only on top-level GET navigations. Good security/UX balance:
```http
Set-Cookie: session=abc123; SameSite=Lax; Secure; HttpOnly
```

**None** — cookies sent on all cross-site requests. Requires `Secure` flag:
```http
Set-Cookie: session=abc123; SameSite=None; Secure; HttpOnly
```

---

## 2. Browser Default Behaviors

| Browser        | Default (no SameSite set) | Since              |
|----------------|---------------------------|--------------------|
| **Chrome**     | Lax                       | Chrome 80 (Feb 2020) |
| **Edge**       | Lax                       | Edge 80 (Feb 2020)   |
| **Firefox**    | Lax                       | Firefox 96 (Jan 2022)|
| **Safari**     | Varies / Strict-like      | Partial since 2020   |

### Chrome's Lax+POST Exception

Chrome allows cross-site POST with fresh `SameSite=Lax` cookies for **2 minutes** after being set:

```
COOKIE SET              2 MINUTES              AFTER 2 MIN
    |                      |                       |
    | Lax+POST allowed     | Full Lax enforced     |
    | (POST works cross-   | (POST blocked cross-  |
    |  site)               |  site)                |
    |______________________|_______________________|
```

This creates an exploitable window immediately after authentication.

---

## 3. Impact on CSRF Protection

| Attack Vector                      | No SameSite | Strict | Lax     | None       |
|------------------------------------|:-----------:|:------:|:-------:|:----------:|
| Cross-site POST form               | Vulnerable  | ✅ Safe| ✅ Safe | Vulnerable |
| Cross-site GET (link)              | Vulnerable  | ✅ Safe| ⚠ Sent  | Vulnerable |
| Cross-site image/iframe            | Vulnerable  | ✅ Safe| ✅ Safe | Vulnerable |
| Cross-site AJAX/fetch              | Vulnerable  | ✅ Safe| ✅ Safe | Vulnerable |
| GET endpoint with side effects     | Vulnerable  | ✅ Safe| ⚠ Vuln  | Vulnerable |
| Subdomain attack                   | Vulnerable  | Vulnerable | Vulnerable | Vulnerable |

---

## 4. Limitations and Bypasses

### Bypass 1: GET-Based State Changes

`SameSite=Lax` sends cookies on top-level GET navigation — dangerous if GET changes state:

```html
<!-- SameSite=Lax allows this -->
<a href="https://target.com/api/transfer?to=attacker&amount=9999">Click me!</a>
<meta http-equiv="refresh" content="0;url=https://target.com/api/delete?id=5" />
```

### Bypass 2: Subdomain Exploitation

SameSite checks the registrable domain (eTLD+1), not full hostname:

```
  app.example.com  ──┐
                     ├── Same-site → Cookie IS sent
  evil.example.com ──┘

  app.example.com  ──┐
                     ├── Cross-site → Cookie NOT sent
  attacker.com     ──┘
```

A compromised subdomain (`blog.example.com`) can forge requests to `app.example.com` with cookies attached.

### Bypass 3: Method Override

Some frameworks accept method overrides, converting GET to POST:

```html
<a href="https://target.com/api/action?_method=DELETE&id=123">Click</a>
```

| Framework | Override Mechanism     |
|-----------|-----------------------|
| Rails     | `_method` parameter   |
| Laravel   | `_method` hidden field|
| Spring    | HiddenHttpMethodFilter|

### Bypass 4: Open Redirect

An open redirect on the target converts cross-site into same-site:

```
1. Attacker: https://target.com/redirect?url=/admin/delete?id=5
2. Browser: top-level nav to target.com → cookie sent (Lax)
3. Redirect follows → action executes with auth
```

---

## 5. Configuration Best Practices

### Decision Guide

```
Does the cookie need cross-site access?
    /                              \
  YES                               NO
  /                                   \
SameSite=None; Secure           External login links important?
+ additional CSRF defense           /                \
(tokens, Origin check)           YES                  NO
                                 /                      \
                           SameSite=Lax             SameSite=Strict
                           + CSRF tokens
```

### Platform Configuration

```python
# Django settings.py
SESSION_COOKIE_SAMESITE = 'Lax'
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SAMESITE = 'Lax'
```

```javascript
// Express.js
app.use(session({
    cookie: { sameSite: 'lax', secure: true, httpOnly: true }
}));
```

```nginx
# Nginx
proxy_cookie_flags ~ SameSite=Lax Secure HttpOnly;
```

```apache
# Apache
Header always edit Set-Cookie (.*) "$1; SameSite=Lax; Secure"
```

---

## Summary Table

| Concept                 | Key Takeaway                                                 |
|-------------------------|--------------------------------------------------------------|
| **SameSite=Strict**     | No cross-site cookies at all; strongest but impacts UX       |
| **SameSite=Lax**        | Only top-level GET; good balance of security and usability   |
| **SameSite=None**       | All cross-site; requires Secure; no CSRF help                |
| **Browser Defaults**    | Chrome/Edge/Firefox default to Lax since 2020-2022           |
| **Lax+POST Window**     | Chrome allows cross-site POST for 2 min on fresh cookies     |
| **Subdomain Bypass**    | Subdomains are same-site; compromised subdomain bypasses it  |
| **Defense in Depth**    | SameSite alone insufficient; combine with tokens and headers |

---

## Revision Questions

1. Explain the difference between "same-site" and "same-origin." Why does this matter for SameSite security?
2. An app sets `SameSite=Lax` but has `GET /account/delete`. Why is it still vulnerable?
3. Describe Chrome's Lax+POST 2-minute exception and a realistic attack exploiting it.
4. XSS exists on `blog.example.com`. Explain how CSRF against `shop.example.com` works despite `SameSite=Lax`.
5. Why does `SameSite=None` require `Secure`? What happens without it in modern browsers?
6. Design a defense-in-depth strategy for an SSO provider that must support cross-site cookies.

---

*Previous: [03-token-based-protection.md](03-token-based-protection.md) | Next: [05-testing-methodology.md](05-testing-methodology.md)*

---

*[Back to README](../README.md)*
