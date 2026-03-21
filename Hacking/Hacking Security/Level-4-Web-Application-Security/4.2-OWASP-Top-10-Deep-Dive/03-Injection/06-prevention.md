# Unit 3: A03 - Injection — Topic 6: Injection Prevention

## Overview

Injection prevention requires a **defense-in-depth approach** combining secure coding practices, input validation, output encoding, and architectural controls. No single technique is sufficient — each layer addresses a different attack vector. This topic covers comprehensive prevention strategies for SQL, NoSQL, OS command, LDAP, and other injection types across all major programming languages and frameworks.

---

## 1. The Prevention Hierarchy

```
┌────────────────────────────────────────────────────────────────────┐
│              INJECTION PREVENTION HIERARCHY                        │
│              (Most effective → Least effective)                    │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  1. PARAMETERIZED QUERIES / PREPARED STATEMENTS  ← Best defense   │
│     Separates code from data at the protocol level                │
│                                                                    │
│  2. STORED PROCEDURES (with parameterized calls)                  │
│     Pre-compiled database routines                                │
│                                                                    │
│  3. ORM / QUERY BUILDERS                                          │
│     Abstraction layers that auto-parameterize                     │
│                                                                    │
│  4. INPUT VALIDATION (Whitelist/Allowlist)                         │
│     Only permit known-good characters/patterns                    │
│                                                                    │
│  5. OUTPUT ENCODING / ESCAPING                                    │
│     Context-aware encoding of special characters                  │
│                                                                    │
│  6. WAF / RASP (Runtime Protection)                               │
│     Pattern-based detection and blocking                          │
│                                                                    │
│  7. LEAST PRIVILEGE                                               │
│     Minimize damage if injection succeeds                         │
│                                                                    │
│  USE ALL LAYERS TOGETHER — Defense in Depth                       │
└────────────────────────────────────────────────────────────────────┘
```

---

## 2. Parameterized Queries — Every Major Language

### Python

```python
# ❌ VULNERABLE — String formatting
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")
cursor.execute("SELECT * FROM users WHERE name = '%s'" % name)
cursor.execute("SELECT * FROM users WHERE name = '" + name + "'")

# ✅ SECURE — Parameterized (sqlite3)
cursor.execute("SELECT * FROM users WHERE id = ?", (user_id,))

# ✅ SECURE — Parameterized (psycopg2 / PostgreSQL)
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))

# ✅ SECURE — Parameterized (mysql-connector)
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))

# ✅ SECURE — Named parameters
cursor.execute(
    "SELECT * FROM users WHERE name = :name AND role = :role",
    {"name": username, "role": user_role}
)
```

### Java

```java
// ❌ VULNERABLE — String concatenation
String query = "SELECT * FROM users WHERE id = " + userId;
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery(query);

// ✅ SECURE — PreparedStatement
String query = "SELECT * FROM users WHERE id = ? AND role = ?";
PreparedStatement pstmt = conn.prepareStatement(query);
pstmt.setInt(1, userId);
pstmt.setString(2, userRole);
ResultSet rs = pstmt.executeQuery();

// ✅ SECURE — JPA/Hibernate with named parameters
TypedQuery<User> query = entityManager.createQuery(
    "SELECT u FROM User u WHERE u.name = :name", User.class);
query.setParameter("name", username);
List<User> results = query.getResultList();
```

### PHP

```php
// ❌ VULNERABLE — Direct interpolation
$query = "SELECT * FROM users WHERE id = " . $_GET['id'];
$result = mysqli_query($conn, $query);

// ✅ SECURE — PDO with prepared statements
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = :id AND role = :role");
$stmt->execute(['id' => $userId, 'role' => $userRole]);
$results = $stmt->fetchAll();

// ✅ SECURE — MySQLi with prepared statements
$stmt = $mysqli->prepare("SELECT * FROM users WHERE id = ? AND name = ?");
$stmt->bind_param("is", $userId, $userName);  // i=integer, s=string
$stmt->execute();
$result = $stmt->get_result();
```

### Node.js

```javascript
// ❌ VULNERABLE — Template literal / concatenation
const query = `SELECT * FROM users WHERE id = ${userId}`;
db.query(query, callback);

// ✅ SECURE — Parameterized (mysql2)
db.query('SELECT * FROM users WHERE id = ? AND role = ?', [userId, role], callback);

// ✅ SECURE — Parameterized (pg / PostgreSQL)
const result = await pool.query(
    'SELECT * FROM users WHERE id = $1 AND role = $2',
    [userId, role]
);

// ✅ SECURE — Knex.js query builder
const users = await knex('users')
    .where({ id: userId, role: role })
    .select('*');
```

### C# (.NET)

```csharp
// ❌ VULNERABLE — String concatenation
string query = "SELECT * FROM users WHERE id = " + userId;
SqlCommand cmd = new SqlCommand(query, conn);

// ✅ SECURE — Parameterized query
string query = "SELECT * FROM users WHERE id = @id AND role = @role";
SqlCommand cmd = new SqlCommand(query, conn);
cmd.Parameters.AddWithValue("@id", userId);
cmd.Parameters.AddWithValue("@role", userRole);
SqlDataReader reader = cmd.ExecuteReader();

// ✅ SECURE — Entity Framework (LINQ)
var users = context.Users
    .Where(u => u.Id == userId && u.Role == role)
    .ToList();
```

### Ruby

```ruby
# ❌ VULNERABLE — String interpolation
User.where("name = '#{params[:name]}'")

# ✅ SECURE — Parameterized (ActiveRecord)
User.where("name = ? AND role = ?", params[:name], params[:role])
User.where(name: params[:name], role: params[:role])

# ✅ SECURE — Arel
users = User.arel_table
User.where(users[:name].eq(params[:name]))
```

### Go

```go
// ❌ VULNERABLE — fmt.Sprintf
query := fmt.Sprintf("SELECT * FROM users WHERE id = %s", userID)
rows, err := db.Query(query)

// ✅ SECURE — Parameterized
rows, err := db.Query("SELECT * FROM users WHERE id = $1 AND role = $2", userID, role)

// ✅ SECURE — Prepared statement
stmt, err := db.Prepare("SELECT * FROM users WHERE id = $1")
defer stmt.Close()
rows, err := stmt.Query(userID)
```

---

## 3. ORM Security

| ORM | Language | Secure Usage | Dangerous Patterns |
|-----|----------|-------------|-------------------|
| **SQLAlchemy** | Python | `session.query(User).filter_by(name=n)` | `session.execute(text(f"...{input}..."))` |
| **Django ORM** | Python | `User.objects.filter(name=n)` | `User.objects.raw(f"...{input}...")` |
| **Hibernate** | Java | Named parameters in HQL | String concat in HQL |
| **ActiveRecord** | Ruby | `where(name: n)` | `where("name = '#{n}'")` |
| **Sequelize** | Node.js | `User.findAll({where: {name}})` | `sequelize.query(...)` with concat |
| **Entity Framework** | C# | LINQ queries | `FromSqlRaw` with concat |

### ORM Injection Pitfalls

```python
# Even ORMs can be vulnerable when misused!

# ❌ VULNERABLE — Raw SQL through ORM
User.objects.raw(f"SELECT * FROM users WHERE name = '{name}'")

# ❌ VULNERABLE — extra() with user input (Django)
User.objects.extra(where=[f"name = '{name}'"])

# ❌ VULNERABLE — text() with formatting (SQLAlchemy)
session.execute(text(f"SELECT * FROM users WHERE name = '{name}'"))

# ✅ SECURE — Use ORM methods properly
User.objects.filter(name=name)  # Django
session.query(User).filter(User.name == name)  # SQLAlchemy
```

---

## 4. Input Validation

### Whitelist (Allowlist) Approach

```python
import re

# Validate expected data types
def validate_integer(value):
    try:
        return int(value)
    except (ValueError, TypeError):
        raise ValidationError("Invalid integer")

# Validate with regex patterns
PATTERNS = {
    'username': re.compile(r'^[a-zA-Z0-9_]{3,30}$'),
    'email': re.compile(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'),
    'domain': re.compile(r'^[a-zA-Z0-9][a-zA-Z0-9.-]{0,253}[a-zA-Z0-9]$'),
    'ip_address': re.compile(r'^(\d{1,3}\.){3}\d{1,3}$'),
    'filename': re.compile(r'^[a-zA-Z0-9._-]{1,255}$'),
    'uuid': re.compile(r'^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$'),
}

def validate_input(value, input_type):
    pattern = PATTERNS.get(input_type)
    if not pattern:
        raise ValueError(f"Unknown input type: {input_type}")
    if not pattern.match(value):
        raise ValidationError(f"Invalid {input_type}: {value}")
    return value
```

### Blacklist vs Whitelist

```
BLACKLIST (❌ NOT RECOMMENDED):
- Block known-bad characters: ', ", ;, --, etc.
- Problem: Attackers always find bypasses
- Problem: Different encodings (URL, Unicode, hex) evade blocks
- Problem: Incomplete — always missing some character

WHITELIST (✅ RECOMMENDED):
- Only allow known-good characters
- Define EXACTLY what is permitted
- Reject everything else by default
- Much harder to bypass
```

---

## 5. NoSQL Injection Prevention

```javascript
// ❌ VULNERABLE — MongoDB operator injection
app.post('/login', (req, res) => {
    db.collection('users').findOne({
        username: req.body.username,   // Could be {"$ne": ""}
        password: req.body.password    // Could be {"$gt": ""}
    });
});

// ✅ SECURE — Type checking + sanitization
const mongoSanitize = require('express-mongo-sanitize');
app.use(mongoSanitize());  // Strips $ and . from req.body/query/params

app.post('/login', (req, res) => {
    // Explicit type checking
    if (typeof req.body.username !== 'string' || 
        typeof req.body.password !== 'string') {
        return res.status(400).json({error: 'Invalid input'});
    }
    
    db.collection('users').findOne({
        username: String(req.body.username),
        password: String(req.body.password)
    });
});

// ✅ SECURE — Mongoose with schema validation
const userSchema = new mongoose.Schema({
    username: { type: String, required: true, match: /^[a-zA-Z0-9_]+$/ },
    password: { type: String, required: true }
});
```

---

## 6. OS Command Injection Prevention

```python
# ❌ VULNERABLE — Shell execution
import os
os.system(f"ping {user_input}")

# ✅ Strategy 1: Use language-native libraries (BEST)
import socket
try:
    result = socket.getaddrinfo(user_input, None)
except socket.gaierror:
    print("Invalid host")

# ✅ Strategy 2: Argument array without shell
import subprocess
# Validate first
import re
if not re.match(r'^[a-zA-Z0-9.-]+$', user_input):
    raise ValueError("Invalid input")
result = subprocess.run(
    ["ping", "-c", "4", user_input],
    shell=False,              # CRITICAL: shell=False
    capture_output=True,
    timeout=30                # Prevent hanging
)

# ✅ Strategy 3: Restricted API approach
ALLOWED_COMMANDS = {
    "ping": lambda host: subprocess.run(["ping", "-c", "4", host], shell=False),
    "nslookup": lambda host: subprocess.run(["nslookup", host], shell=False),
}

def safe_execute(command, argument):
    if command not in ALLOWED_COMMANDS:
        raise ValueError("Command not allowed")
    if not re.match(r'^[a-zA-Z0-9.-]+$', argument):
        raise ValueError("Invalid argument")
    return ALLOWED_COMMANDS[command](argument)
```

---

## 7. LDAP Injection Prevention

```python
import ldap
from ldap.filter import escape_filter_chars

# ❌ VULNERABLE
filter_str = f"(&(uid={username})(userPassword={password}))"

# ✅ SECURE — Escape special characters
safe_user = escape_filter_chars(username)
safe_pass = escape_filter_chars(password)
filter_str = f"(&(uid={safe_user})(userPassword={safe_pass}))"

# ✅ BEST — Use LDAP bind authentication
def authenticate(username, password):
    safe_user = escape_filter_chars(username)
    # Find user DN first
    results = conn.search_s(base_dn, ldap.SCOPE_SUBTREE, f"(uid={safe_user})", ['dn'])
    if not results:
        return False
    # Bind as user to verify password
    user_dn = results[0][0]
    try:
        test_conn = ldap.initialize(ldap_url)
        test_conn.simple_bind_s(user_dn, password)
        return True
    except ldap.INVALID_CREDENTIALS:
        return False
```

---

## 8. WAF and Runtime Protection

### Web Application Firewall (WAF)

```
ModSecurity Core Rule Set (CRS):
- SQL injection detection patterns
- OS command injection patterns
- LDAP injection patterns
- XSS detection (related)

Rule example (ModSecurity):
SecRule ARGS "@detectSQLi" \
    "id:942100,\
    phase:2,\
    block,\
    t:none,t:urlDecodeUni,\
    msg:'SQL Injection Attack Detected',\
    logdata:'Matched Data: %{TX.0} found within %{MATCHED_VAR_NAME}',\
    tag:'OWASP_CRS',\
    severity:'CRITICAL'"
```

### RASP (Runtime Application Self-Protection)

```
RASP Tools:
┌─────────────────────────┬──────────────────────────────┐
│ Tool                    │ Languages                    │
├─────────────────────────┼──────────────────────────────┤
│ Sqreen (now Datadog)    │ Python, Node.js, Ruby, Go    │
│ Contrast Security       │ Java, .NET, Node.js, Ruby    │
│ Imperva RASP            │ Java, .NET                   │
│ OpenRASP (Baidu)        │ Java, PHP                    │
└─────────────────────────┴──────────────────────────────┘

How RASP works:
1. Instruments the application at runtime
2. Monitors database queries, system calls, file access
3. Detects injection attempts at the function level
4. Blocks malicious queries before execution
5. Provides more context than WAF (knows the code path)
```

---

## 9. SAST and DAST for Injection Detection

### Static Application Security Testing (SAST)

```
Finds injection vulnerabilities in source code:

Tools:
┌──────────────────┬───────────────────┬───────────────────────────┐
│ Tool             │ Languages         │ Type                      │
├──────────────────┼───────────────────┼───────────────────────────┤
│ Semgrep          │ 20+ languages     │ Open source               │
│ SonarQube        │ 25+ languages     │ Open source / Commercial  │
│ Checkmarx        │ 25+ languages     │ Commercial                │
│ Veracode         │ 20+ languages     │ Commercial                │
│ CodeQL (GitHub)  │ C/C++, C#, Java,  │ Free for open source      │
│                  │ JS, Python, Ruby  │                           │
│ Bandit           │ Python            │ Open source               │
│ Brakeman         │ Ruby on Rails     │ Open source               │
└──────────────────┴───────────────────┴───────────────────────────┘

Semgrep rule for SQL injection:
rules:
  - id: sql-injection
    patterns:
      - pattern: |
          $CURSOR.execute(f"...{$VAR}...")
    message: "Possible SQL injection via f-string"
    severity: ERROR
    languages: [python]
```

### Dynamic Application Security Testing (DAST)

```
Tests running applications for injection:

Tools:
┌──────────────────┬──────────────────────────────────────────┐
│ Tool             │ Capabilities                             │
├──────────────────┼──────────────────────────────────────────┤
│ Burp Suite Pro   │ Active scanning, SQLi, CMDi, LDAPi      │
│ OWASP ZAP        │ Free, active/passive scanning            │
│ SQLMap           │ Specialized SQL injection                │
│ Commix           │ Specialized command injection            │
│ Nikto            │ Web server scanner                       │
│ Arachni          │ Full-featured web scanner                │
└──────────────────┴──────────────────────────────────────────┘
```

---

## 10. Secure Architecture Patterns

```
┌─────────────────────────────────────────────────────────────────┐
│              SECURE ARCHITECTURE LAYERS                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  [Client] → [WAF] → [API Gateway] → [Application] → [Database]│
│                                                                 │
│  Layer 1: WAF                                                  │
│    - Block known injection patterns                            │
│    - Rate limiting                                             │
│    - Request size limits                                       │
│                                                                 │
│  Layer 2: API Gateway                                          │
│    - Schema validation (JSON Schema, GraphQL)                  │
│    - Input type enforcement                                    │
│    - Request sanitization                                      │
│                                                                 │
│  Layer 3: Application                                          │
│    - Input validation (whitelist)                              │
│    - Parameterized queries                                     │
│    - ORM with safe methods                                     │
│    - RASP monitoring                                           │
│                                                                 │
│  Layer 4: Database                                             │
│    - Least privilege (read-only where possible)                │
│    - No direct DDL from application accounts                  │
│    - Stored procedures for complex operations                  │
│    - Query logging and alerting                                │
│                                                                 │
│  Layer 5: Network                                              │
│    - Database not publicly accessible                          │
│    - Segmented from web tier                                   │
│    - Encrypted connections (TLS)                               │
└─────────────────────────────────────────────────────────────────┘
```

### Database Least Privilege

```sql
-- Create application-specific database user
CREATE USER 'webapp'@'localhost' IDENTIFIED BY 'strong_password';

-- Grant ONLY necessary permissions
GRANT SELECT, INSERT, UPDATE ON myapp.users TO 'webapp'@'localhost';
GRANT SELECT ON myapp.products TO 'webapp'@'localhost';

-- NEVER grant these to application accounts:
-- GRANT ALL PRIVILEGES
-- GRANT FILE                    -- File read/write
-- GRANT PROCESS                 -- Process listing  
-- GRANT SUPER                   -- Administrative
-- GRANT CREATE, DROP, ALTER     -- Schema modification
```

---

## 11. Code Review Checklist for Injection

```
□ Are ALL database queries parameterized?
□ Are ORM raw query methods avoided (or properly parameterized)?
□ Is user input validated against a whitelist before processing?
□ Are OS commands avoided? If not, is shell=False used?
□ Are LDAP queries using escape functions?
□ Is input type-checked (string vs object for NoSQL)?
□ Are stored procedures using parameterized internal queries?
□ Is the database account using least privilege?
□ Are error messages sanitized (no SQL errors leaked)?
□ Is a WAF deployed and configured?
□ Are SAST tools integrated into CI/CD pipeline?
□ Are dependencies scanned for known injection vulnerabilities?
```

---

## Summary Table

| Injection Type | Primary Prevention | Secondary Prevention |
|---------------|-------------------|---------------------|
| **SQL Injection** | Parameterized queries | ORM, input validation, WAF |
| **NoSQL Injection** | Type checking, sanitize `$` | Schema validation, express-mongo-sanitize |
| **OS Command** | Avoid shell; use native libs | Argument arrays, whitelist commands |
| **LDAP** | Escape special chars, bind auth | Input validation, least privilege |
| **XPath** | Parameterized XPath | Input validation |
| **Template** | Sandboxed engines, no user templates | Input validation, WAF |

---

## Revision Questions

1. Why are parameterized queries the most effective defense against SQL injection? How do they work at the protocol level?
2. Compare whitelist vs blacklist input validation. Why is whitelist preferred?
3. Show parameterized query examples in Python, Java, PHP, Node.js, and Go.
4. How can ORMs still be vulnerable to injection? Give examples in Django and SQLAlchemy.
5. Explain the defense-in-depth approach to injection prevention with all layers.
6. What is the difference between SAST and DAST? Name tools for each and their strengths.

---

*Previous: [05-ldap-injection.md](05-ldap-injection.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*
