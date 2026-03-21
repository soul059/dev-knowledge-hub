# Prevention Techniques

## Unit 5: SQL Injection — Topic 7

## 🎯 Overview

Preventing SQL injection requires a defense-in-depth approach combining secure coding practices, input validation, and architectural controls. As a penetration tester, understanding prevention helps you identify when protections are properly implemented and find gaps in defenses. This topic covers the primary prevention methods and how to verify their effectiveness.

---

## 1. Parameterized Queries (Prepared Statements)

```
┌──────────────────────────────────────────────────────────────┐
│      THE #1 DEFENSE: PARAMETERIZED QUERIES                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ❌ VULNERABLE (String concatenation):                       │
│  query = "SELECT * FROM users WHERE id = '" + userInput + "'"│
│                                                              │
│  ✅ SECURE (Parameterized):                                  │
│  query = "SELECT * FROM users WHERE id = ?"                  │
│  stmt.execute(query, [userInput])                            │
│                                                              │
│  Why it works:                                               │
│  • SQL structure is pre-compiled                             │
│  • User input is ALWAYS treated as data, never code          │
│  • Database engine handles escaping                          │
│  • Injection is structurally impossible                      │
└──────────────────────────────────────────────────────────────┘
```

### Language-Specific Examples

```python
# Python (psycopg2)
# ❌ Vulnerable
cursor.execute(f"SELECT * FROM users WHERE id = '{user_id}'")
# ✅ Secure
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))

# Python (SQLAlchemy)
# ✅ ORM is safe by default
User.query.filter_by(id=user_id).first()
```

```javascript
// Node.js (MySQL)
// ❌ Vulnerable
connection.query(`SELECT * FROM users WHERE id = '${userId}'`);
// ✅ Secure
connection.query("SELECT * FROM users WHERE id = ?", [userId]);
```

```php
// PHP (PDO)
// ❌ Vulnerable
$stmt = $pdo->query("SELECT * FROM users WHERE id = '$id'");
// ✅ Secure
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([$id]);
```

```java
// Java (PreparedStatement)
// ❌ Vulnerable
Statement stmt = conn.createStatement();
stmt.executeQuery("SELECT * FROM users WHERE id = '" + id + "'");
// ✅ Secure
PreparedStatement stmt = conn.prepareStatement(
    "SELECT * FROM users WHERE id = ?");
stmt.setString(1, id);
```

---

## 2. Input Validation

```
┌──────────────────────────────────────────────────────────────┐
│              INPUT VALIDATION STRATEGIES                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Whitelist (Allowlist):                                      │
│  • Only allow expected characters/patterns                   │
│  • Numeric ID: ^[0-9]+$                                      │
│  • Username: ^[a-zA-Z0-9_]{3,20}$                            │
│  • Email: standard email regex                               │
│                                                              │
│  Type Checking:                                              │
│  • Cast to expected type before use                          │
│  • int(user_id) — fails if not numeric                       │
│  • Use type-safe parameters                                  │
│                                                              │
│  Length Limits:                                               │
│  • Maximum input length                                      │
│  • Prevents very long injection payloads                     │
│                                                              │
│  ⚠ Input validation is SECONDARY defense                    │
│  ⚠ NEVER rely on validation alone                           │
│  ⚠ Always use parameterized queries as primary defense      │
└──────────────────────────────────────────────────────────────┘
```

---

## 3. Stored Procedures

```sql
-- Stored procedures can be safe IF properly implemented

-- ✅ Safe stored procedure
CREATE PROCEDURE GetUser(@UserId INT)
AS
  SELECT * FROM users WHERE id = @UserId
GO
-- Parameter is typed (INT) and cannot contain SQL

-- ❌ Unsafe stored procedure (dynamic SQL inside)
CREATE PROCEDURE SearchUsers(@SearchTerm NVARCHAR(100))
AS
  EXEC('SELECT * FROM users WHERE name = ''' + @SearchTerm + '''')
GO
-- String concatenation inside stored procedure = still vulnerable!

-- ✅ Safe dynamic SQL in stored procedure
CREATE PROCEDURE SearchUsers(@SearchTerm NVARCHAR(100))
AS
  EXEC sp_executesql 
    N'SELECT * FROM users WHERE name = @term',
    N'@term NVARCHAR(100)',
    @term = @SearchTerm
GO
```

---

## 4. Web Application Firewall (WAF)

```
┌──────────────────────────────────────────────────────────────┐
│              WAF AS DEFENSE LAYER                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  WAF can detect and block common SQLi patterns:              │
│  • ' OR 1=1--                                                │
│  • UNION SELECT                                              │
│  • SLEEP(), BENCHMARK()                                      │
│  • xp_cmdshell, LOAD_FILE                                    │
│                                                              │
│  Limitations:                                                │
│  • Bypassable with encoding/obfuscation                     │
│  • False positives (blocks legitimate input)                 │
│  • Cannot understand application context                     │
│  • Maintenance burden (rule updates)                         │
│                                                              │
│  ⚠ WAF is an ADDITIONAL layer, NOT a replacement            │
│  ⚠ Fix the code, don't just block the attack                │
└──────────────────────────────────────────────────────────────┘
```

---

## 5. Least Privilege Database Access

```sql
-- Application database user should have MINIMUM permissions

-- ❌ Bad: Application uses root/sa account
-- Full database access, file system access, OS commands

-- ✅ Good: Create restricted application user
CREATE USER 'webapp'@'localhost' IDENTIFIED BY 'strong_password';
GRANT SELECT, INSERT, UPDATE ON webapp_db.* TO 'webapp'@'localhost';
-- No DELETE, DROP, CREATE, FILE, EXECUTE privileges

-- Separate users for different functions
-- Read-only user for reporting
-- Read-write user for application
-- Admin user only for maintenance (never used by app)

-- Disable dangerous features
-- MSSQL: Disable xp_cmdshell
EXEC sp_configure 'xp_cmdshell', 0;
RECONFIGURE;
-- MySQL: Disable LOAD_FILE/INTO OUTFILE
-- Set --secure-file-priv in configuration
```

---

## 6. Defense-in-Depth Strategy

```
┌──────────────────────────────────────────────────────────────┐
│              DEFENSE IN DEPTH                                 │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Layer 1: Parameterized Queries     → Primary defense        │
│  Layer 2: Input Validation          → Reject bad input       │
│  Layer 3: Escaping                  → Encode special chars   │
│  Layer 4: Least Privilege           → Limit DB permissions   │
│  Layer 5: WAF                       → Block known patterns   │
│  Layer 6: Error Handling            → Don't leak DB errors   │
│  Layer 7: Monitoring & Logging      → Detect attack attempts │
│  Layer 8: Code Review               → Catch vulns early      │
│  Layer 9: Security Testing          → Verify defenses work   │
│                                                              │
│  Each layer catches what the previous layer missed           │
└──────────────────────────────────────────────────────────────┘
```

---

## 7. Verifying Prevention Effectiveness

```bash
# As a pentester, verify protections:

# 1. Test parameterized queries
# Submit SQLi payload → should return no error, no injection
curl "https://target.com/page?id=1' OR 1=1--"
# Expected: Same page as id=1 (input treated as literal string)

# 2. Test error handling
curl "https://target.com/page?id=999999'"
# Expected: Generic error, NO database error details

# 3. Test input validation
curl "https://target.com/page?id=abc"
# Expected: Validation error (if id should be numeric)

# 4. Test WAF
curl "https://target.com/page?id=1 UNION SELECT 1,2,3--"
# Expected: WAF blocks request (403)

# 5. Test privilege
# If SQLi found, try:
sqlmap -u "URL" --privileges
# Application user should NOT be DBA/root
```

---

## 📊 Summary Table

| Defense | Effectiveness | Layer |
|---------|-------------|-------|
| Parameterized queries | ✅ Primary (prevents injection) | Code |
| Input validation | ✅ Secondary (rejects bad input) | Code |
| ORM usage | ✅ Generally safe (use correctly) | Code |
| Stored procedures | ⚠️ Must avoid dynamic SQL inside | Database |
| Least privilege | ✅ Limits damage if exploited | Database |
| WAF | ⚠️ Bypassable but helpful | Network |
| Error handling | ✅ Prevents info leakage | Code |

---

## ❓ Revision Questions

1. Why are parameterized queries the primary defense against SQL injection?
2. How can a stored procedure still be vulnerable to SQL injection?
3. What database permissions should an application user have?
4. Why is a WAF insufficient as the sole defense against SQLi?
5. What is defense-in-depth and how does it apply to SQL injection?
6. How do you verify that SQL injection protections are properly implemented?

---

*Previous: [06-automated-tools-sqlmap.md](06-automated-tools-sqlmap.md)*

---

*[Back to README](../README.md)*
