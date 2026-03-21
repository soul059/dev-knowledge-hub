# Unit 7: Cross-Site Request Forgery (CSRF) - Topic 1: CSRF Mechanism

## Overview

Cross-Site Request Forgery (CSRF) forces authenticated users to execute unwanted actions on a web application. Unlike XSS, which exploits user trust in a site, CSRF exploits the trust a site has in the user's browser. This topic covers the mechanics, conditions, and forms of CSRF attacks.

---

## 1. How CSRF Attacks Work

```
+----------------+       +----------------+       +----------------+
| Victim Browser |       | evil-site.com  |       |   bank.com     |
+----------------+       +----------------+       +----------------+
       |                        |                        |
       | 1. Login to bank.com   |                        |
       |------------------------------------------------>|
       |                        |    Session cookie set   |
       |<------------------------------------------------|
       | 2. Visit evil-site.com |                        |
       |----------------------->|                        |
       | 3. Page with hidden    |                        |
       |    form returned       |                        |
       |<-----------------------|                        |
       | 4. Auto-submit to bank.com (cookie attached)    |
       |------------------------------------------------>|
       |                        | 5. Transfer processed  |
       |<------------------------------------------------|
```

1. Victim authenticates with `bank.com` and receives a session cookie
2. Victim visits `evil-site.com` without logging out
3. Malicious page contains a hidden form targeting `bank.com`
4. Browser automatically attaches the session cookie to the forged request
5. `bank.com` processes the request as legitimate

---

## 2. Cookie-Based Authentication Exploitation

Browsers automatically include cookies with every request to the matching domain, regardless of origin.

| Aspect                  | Description                                              |
|-------------------------|----------------------------------------------------------|
| **Automatic Inclusion** | Cookies attach to all requests matching the domain       |
| **No Origin Check**     | Server cannot distinguish user-initiated vs forged       |
| **Session Persistence** | Cookies remain valid until expiry or logout               |
| **Cross-Origin Sends**  | Cookies sent from third-party origins by default         |

```html
<!-- Attacker page — victim's session cookie is sent automatically -->
<img src="https://bank.com/transfer?to=attacker&amount=5000" />
```

---

## 3. GET vs POST CSRF Attacks

**GET-Based** — simplest form, uses elements that trigger GET requests:

```html
<img src="https://vulnerable.com/api/delete-account?confirm=yes" width="0" height="0" />
```

**POST-Based** — requires an auto-submitting form:

```html
<form action="https://bank.com/transfer" method="POST" id="csrf">
  <input type="hidden" name="to_account" value="ATTACKER" />
  <input type="hidden" name="amount" value="10000" />
</form>
<script>document.getElementById('csrf').submit();</script>
```

| Feature         | GET-Based CSRF               | POST-Based CSRF                 |
|-----------------|------------------------------|---------------------------------|
| **Delivery**    | Image, iframe, link          | Auto-submitting hidden form     |
| **Visibility**  | Completely invisible         | May cause page flash/redirect   |
| **Complexity**  | Very simple                  | Slightly more complex           |
| **Targets**     | Poorly designed APIs, logout | Forms (transfer, settings)      |

---

## 4. Conditions Required for CSRF

All conditions must be met for a successful attack:

```
+--------------------------------------------------------------+
|  [1] A relevant state-changing action exists                  |
|  [2] Application relies solely on cookies for sessions        |
|  [3] All request parameters can be guessed/determined         |
|  [4] Victim has an active authenticated session               |
|  [5] Victim visits the attacker-controlled page               |
+--------------------------------------------------------------+
```

```bash
# Quick CSRF check — does the endpoint accept requests without a token?
curl -X POST https://target.com/api/change-password \
  -H "Cookie: session=VICTIM_SESSION" \
  -d "new_password=hacked123"

# Check for SameSite cookie attribute
curl -v https://target.com/login 2>&1 | grep -i "set-cookie"
```

---

## 5. Real-World Attack Scenarios

| Scenario               | Target Action           | Impact                          |
|------------------------|-------------------------|---------------------------------|
| **Banking Transfer**   | POST /transfer          | Unauthorized money transfer     |
| **Email Change**       | POST /account/email     | Account takeover via reset chain|
| **Admin User Creation**| POST /admin/create-user | Privilege escalation            |
| **Router DNS Change**  | GET /admin/dns?server=X | DNS hijacking on local network  |
| **Password Change**    | POST /change-password   | Full account takeover           |

---

## Summary Table

| Concept                | Key Takeaway                                                |
|------------------------|-------------------------------------------------------------|
| **CSRF Definition**    | Forces authenticated users to perform unwanted actions      |
| **Root Cause**         | Browsers auto-attach cookies regardless of request origin   |
| **GET CSRF**           | Simplest; uses image tags, links, or redirects              |
| **POST CSRF**          | Requires hidden auto-submitting forms                       |
| **Key Prerequisite**   | Victim must be authenticated and visit attacker's page      |
| **Critical Condition** | No unpredictable tokens are required by the server          |

---

## Revision Questions

1. Explain the trust relationship CSRF exploits. How does this differ from XSS?
2. Why do browsers attach cookies to cross-origin requests, and how does this enable CSRF?
3. Construct a CSRF payload targeting `GET /api/delete-account?confirm=yes`.
4. List all five conditions for a successful CSRF attack. Which single condition, if eliminated, provides the strongest defense?
5. An application uses POST for all state changes but has no CSRF tokens. Demonstrate a proof-of-concept payload.
6. Describe how CSRF against a home router could lead to network compromise.

---

*Next: [02-csrf-vs-xss.md](02-csrf-vs-xss.md)*

---

*[Back to README](../README.md)*
