# Unit 3: A03 - Injection — Topic 1: Injection Types

## Overview

Injection flaws occur when an application sends **untrusted data to an interpreter** as part of a command or query. The attacker's hostile data tricks the interpreter into executing unintended commands or accessing data without authorization. Injection was the #1 OWASP Top 10 category for over a decade (2010–2017) and remains at #3 in 2021, with **33 CWEs mapped** to this category. Understanding the full spectrum of injection types is essential for comprehensive security testing.

---

## 1. How Injection Works

```
┌─────────────────────────────────────────────────────────────────┐
│                   INJECTION FUNDAMENTALS                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Normal Flow:                                                   │
│  User Input: "john"                                            │
│  Query: SELECT * FROM users WHERE name = 'john'                │
│  Result: Returns john's record ✅                               │
│                                                                 │
│  Injection Attack:                                              │
│  User Input: "' OR '1'='1"                                     │
│  Query: SELECT * FROM users WHERE name = '' OR '1'='1'         │
│  Result: Returns ALL records! ❌                                │
│                                                                 │
│  Root Cause:                                                    │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐     │
│  │  User Input  │───▶│ Concatenated │───▶│ Interpreter  │     │
│  │  (untrusted) │    │ into command │    │ executes     │     │
│  └──────────────┘    └──────────────┘    │ BOTH code    │     │
│                                          │ AND data     │     │
│                                          └──────────────┘     │
│                                                                 │
│  The interpreter CANNOT distinguish between                    │
│  intended code and attacker-injected data                      │
└─────────────────────────────────────────────────────────────────┘
```

### Tainted Data Flow

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  SOURCE  │───▶│  FLOW    │───▶│  SINK    │───▶│  IMPACT  │
│          │    │          │    │          │    │          │
│ User     │    │ Variable │    │ Database │    │ Data     │
│ input,   │    │ passing, │    │ query,   │    │ breach,  │
│ HTTP     │    │ string   │    │ OS cmd,  │    │ RCE,     │
│ headers, │    │ concat,  │    │ LDAP     │    │ auth     │
│ cookies  │    │ format   │    │ filter,  │    │ bypass   │
│          │    │ strings  │    │ template │    │          │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
   (Entry)       (Propagation)   (Execution)     (Result)
```

---

## 2. Complete Injection Type Catalog

| Injection Type | Interpreter | Severity | Prevalence |
|---------------|------------|----------|------------|
| **SQL Injection** | SQL database | Critical | Very High |
| **NoSQL Injection** | MongoDB, CouchDB | High | High |
| **OS Command Injection** | System shell | Critical | Medium |
| **LDAP Injection** | LDAP directory | High | Medium |
| **XPath Injection** | XML parser | Medium | Low |
| **Template Injection (SSTI)** | Template engine | Critical | Medium |
| **Header Injection** | HTTP/SMTP | Medium | Medium |
| **ORM Injection** | ORM framework | High | Medium |
| **Expression Language (EL)** | Java EL, Spring EL | Critical | Low |
| **Code Injection** | Language runtime | Critical | Low |
| **Log Injection** | Logging framework | Medium | High |
| **CSV/Formula Injection** | Spreadsheet | Medium | Medium |

---

## 3. Injection Type Details

### SQL Injection

```python
# VULNERABLE
query = f"SELECT * FROM users WHERE id = {user_input}"
# Input: 1 OR 1=1 → Returns all users

# SECURE
cursor.execute("SELECT * FROM users WHERE id = %s", (user_input,))
```

### NoSQL Injection (MongoDB)

```javascript
// VULNERABLE
db.users.find({ username: req.body.username, password: req.body.password });
// Input: {"username": {"$ne": ""}, "password": {"$ne": ""}} → Bypasses auth

// SECURE
db.users.find({ username: String(req.body.username), password: String(req.body.password) });
```

### OS Command Injection

```python
# VULNERABLE
os.system(f"ping {user_input}")
# Input: 127.0.0.1; cat /etc/passwd → Executes both commands

# SECURE
subprocess.run(["ping", "-c", "4", user_input], shell=False)
```

### LDAP Injection

```
# VULNERABLE filter
(&(uid={user_input})(userPassword={password}))
# Input uid: *)(|(uid=*) → Returns all users

# SECURE: Escape special characters (*, (, ), \, NUL)
```

### Server-Side Template Injection (SSTI)

```python
# VULNERABLE (Jinja2)
template = Template(f"Hello {user_input}")
# Input: {{7*7}} → Renders "Hello 49"
# Input: {{config.items()}} → Leaks configuration
# Input: {{''.__class__.__mro__[1].__subclasses__()}} → RCE possible

# SECURE
template = Template("Hello {{ name }}")
template.render(name=user_input)
```

### Header Injection (CRLF)

```
# VULNERABLE
response.set_header("Location", user_input)
# Input: http://evil.com\r\nSet-Cookie: admin=true
# → Injects arbitrary headers

# SECURE: Strip \r\n from header values
```

### Log Injection

```python
# VULNERABLE
logger.info(f"Login attempt for user: {username}")
# Input: admin\n2024-01-15 INFO Login successful for admin
# → Forged log entry

# SECURE
logger.info("Login attempt for user: %s", username.replace('\n', '').replace('\r', ''))
```

---

## 4. CWE Mappings

| CWE | Name | Injection Type |
|-----|------|---------------|
| CWE-79 | Cross-Site Scripting | XSS (HTML/JS injection) |
| CWE-89 | SQL Injection | SQL |
| CWE-78 | OS Command Injection | Command |
| CWE-90 | LDAP Injection | LDAP |
| CWE-91 | XML Injection | XML/XPath |
| CWE-94 | Code Injection | Language runtime |
| CWE-917 | Expression Language Injection | EL/OGNL |
| CWE-1336 | Template Injection | SSTI |
| CWE-943 | NoSQL Injection | NoSQL |
| CWE-113 | HTTP Response Splitting | Header |

---

## 5. Severity Comparison

```
INJECTION SEVERITY RANKING
═══════════════════════════

CRITICAL ████████████████████ OS Command, SSTI, Code, EL
  → Remote Code Execution possible
  → Full server compromise

HIGH     ██████████████████   SQL, NoSQL, LDAP, ORM
  → Data breach, auth bypass
  → Database compromise

MEDIUM   ████████████████     XPath, Header, Log, CSV
  → Data manipulation
  → Limited impact
```

---

## Summary Table

| Aspect | Description |
|--------|-------------|
| **Root Cause** | Untrusted data mixed with commands/queries |
| **Key Concept** | Interpreter can't distinguish code from data |
| **Prevention** | Parameterization, input validation, escaping |
| **Testing** | Manual payload testing, SAST, DAST, sqlmap |
| **CWE Count** | 33 CWEs mapped to A03 |
| **Impact Range** | Data breach to full RCE |

---

## Revision Questions

1. Explain the fundamental mechanism that makes injection attacks possible. What is the root cause?
2. List at least 8 different injection types and the interpreter each targets.
3. What is tainted data flow? Draw a diagram showing source, flow, sink, and impact.
4. Compare SQL injection, NoSQL injection, and OS command injection in terms of severity and impact.
5. What is Server-Side Template Injection? How can it escalate from information disclosure to RCE?
6. How does log injection work and why is it considered a security risk?

---

*Previous: None (First topic) | Next: [02-sql-injection-deep-dive.md](02-sql-injection-deep-dive.md)*

---

*[Back to README](../README.md)*
