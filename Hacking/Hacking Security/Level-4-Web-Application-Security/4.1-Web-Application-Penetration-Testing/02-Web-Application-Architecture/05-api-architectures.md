# API Architectures (REST, GraphQL)

## Unit 2: Web Application Architecture — Topic 5

## 🎯 Overview

Modern web applications heavily rely on APIs for communication between frontend and backend. REST and GraphQL are the dominant API architectures, each with distinct security implications. Understanding API design patterns helps pentesters identify endpoints, test access controls, and exploit API-specific vulnerabilities.

---

## 1. REST API Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                    REST API STRUCTURE                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Base URL: https://api.target.com/v2                        │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  Resource    │  GET        │  POST     │  DELETE     │    │
│  ├─────────────┼─────────────┼───────────┼────────────┤    │
│  │  /users     │  List all   │  Create   │  N/A       │    │
│  │  /users/123 │  Get one    │  N/A      │  Delete    │    │
│  │  /users/123 │             │           │            │    │
│  │   /orders   │  User orders│  Create   │  N/A       │    │
│  │  /orders/456│  Get order  │  N/A      │  Cancel    │    │
│  └─────────────┴─────────────┴───────────┴────────────┘    │
│                                                              │
│  Conventions:                                                │
│  • Resources as nouns (not verbs)                            │
│  • HTTP methods define actions                               │
│  • Stateless — each request contains all needed info         │
│  • JSON is the standard data format                          │
└──────────────────────────────────────────────────────────────┘
```

### REST API Testing

```bash
# Enumerate API endpoints
curl https://api.target.com/v2/users
curl https://api.target.com/v2/users/1
curl https://api.target.com/v2/users/1/orders

# Test IDOR — change user ID
curl -H "Authorization: Bearer $TOKEN" \
  https://api.target.com/v2/users/2/orders

# Test methods
curl -X DELETE https://api.target.com/v2/users/1
curl -X PUT https://api.target.com/v2/users/1 \
  -d '{"role":"admin"}'

# Version enumeration
curl https://api.target.com/v1/users  # Old version may lack security
curl https://api.target.com/v3/users  # Unreleased version
```

---

## 2. GraphQL Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                  GRAPHQL vs REST                              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  REST: Multiple endpoints          GraphQL: Single endpoint  │
│  GET /users/1                      POST /graphql             │
│  GET /users/1/posts                                          │
│  GET /users/1/friends              query {                   │
│  = 3 requests                        user(id: 1) {          │
│                                        name                  │
│                                        posts { title }       │
│                                        friends { name }      │
│                                      }                       │
│                                    }                         │
│                                    = 1 request               │
└──────────────────────────────────────────────────────────────┘
```

### GraphQL Introspection (Enumeration)

```graphql
# Full schema introspection query
{
  __schema {
    types {
      name
      fields {
        name
        type { name }
      }
    }
  }
}

# Enumerate queries and mutations
{
  __schema {
    queryType { fields { name description } }
    mutationType { fields { name description } }
  }
}

# Find specific type details
{
  __type(name: "User") {
    fields {
      name
      type { name kind }
    }
  }
}
```

```bash
# Test for introspection
curl -X POST https://target.com/graphql \
  -H "Content-Type: application/json" \
  -d '{"query":"{__schema{types{name}}}"}'

# Use GraphQL Voyager or GraphiQL for visualization
```

---

## 3. API Authentication Mechanisms

```
┌──────────────────────────────────────────────────────────────┐
│              API AUTHENTICATION METHODS                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  API Keys        │  Bearer Tokens    │  OAuth 2.0            │
│  ────────────    │  ─────────────    │  ──────────           │
│  X-API-Key:      │  Authorization:   │  Authorization:       │
│  abc123          │  Bearer eyJ...    │  Bearer <access_token>│
│                  │                   │                       │
│  • Simple        │  • JWT or opaque  │  • Delegated auth     │
│  • Often static  │  • Time-limited   │  • Scoped access      │
│  • Easy to leak  │  • Self-contained │  • Complex flow       │
│                  │   (if JWT)        │                       │
└──────────────────────────────────────────────────────────────┘
```

### API Key Testing

```bash
# Test for exposed API keys
# Check JavaScript files, mobile apps, git repos
# Try key in different endpoints
curl -H "X-API-Key: found_key" https://api.target.com/admin/users

# Test key permissions
curl -H "X-API-Key: found_key" https://api.target.com/v2/users     # Read
curl -X POST -H "X-API-Key: found_key" https://api.target.com/v2/users  # Write
curl -X DELETE -H "X-API-Key: found_key" https://api.target.com/v2/users/1  # Delete
```

---

## 4. API Security Testing Methodology

```
┌──────────────────────────────────────────────────────────────┐
│                API TESTING WORKFLOW                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Discovery                                                │
│     └→ Find endpoints, documentation, schemas                │
│                                                              │
│  2. Authentication                                           │
│     └→ Test auth bypass, token manipulation                  │
│                                                              │
│  3. Authorization                                            │
│     └→ BOLA/IDOR, function-level access control              │
│                                                              │
│  4. Input Validation                                         │
│     └→ Injection, parameter tampering, type confusion        │
│                                                              │
│  5. Rate Limiting                                            │
│     └→ Brute force, resource exhaustion                      │
│                                                              │
│  6. Business Logic                                           │
│     └→ Workflow bypass, race conditions                      │
└──────────────────────────────────────────────────────────────┘
```

### OWASP API Security Top 10

| # | Vulnerability | Example |
|---|--------------|---------|
| API1 | Broken Object Level Authorization | IDOR in `/api/users/{id}` |
| API2 | Broken Authentication | Weak tokens, no rate limiting |
| API3 | Broken Object Property Level Authorization | Mass assignment |
| API4 | Unrestricted Resource Consumption | No rate limiting |
| API5 | Broken Function Level Authorization | User accesses admin endpoint |
| API6 | Unrestricted Access to Sensitive Business Flows | Automated abuse |
| API7 | Server Side Request Forgery | SSRF via API parameters |
| API8 | Security Misconfiguration | Verbose errors, default creds |
| API9 | Improper Inventory Management | Old API versions exposed |
| API10 | Unsafe Consumption of APIs | Trusting third-party API data |

---

## 5. GraphQL-Specific Attacks

```graphql
# Batch query attack (bypass rate limiting)
[
  {"query": "mutation { login(user:\"admin\", pass:\"pass1\") { token } }"},
  {"query": "mutation { login(user:\"admin\", pass:\"pass2\") { token } }"},
  {"query": "mutation { login(user:\"admin\", pass:\"pass3\") { token } }"}
]

# Nested query DoS (query depth attack)
{
  user(id: 1) {
    friends {
      friends {
        friends {
          friends {
            name  # Exponential database queries
          }
        }
      }
    }
  }
}

# Field suggestion exploitation
# Sending invalid field names may trigger suggestions
{ user { passwor } }
# Error: "Did you mean 'password'?"
```

---

## 📊 Summary Table

| Aspect | REST | GraphQL |
|--------|------|---------|
| Endpoints | Multiple URLs | Single `/graphql` |
| Methods | GET, POST, PUT, DELETE | POST (typically) |
| Data fetching | Fixed response shape | Client defines shape |
| Over-fetching | Common | Avoided |
| Enumeration | Directory brute force | Introspection queries |
| Rate limiting | Per endpoint | Complex (query cost) |
| IDOR testing | Change IDs in URL | Change IDs in query |
| DoS risk | Standard | Nested query attacks |

---

## ❓ Revision Questions

1. How does REST API endpoint structure help in IDOR testing?
2. What information does GraphQL introspection reveal?
3. Why might old API versions (v1, v2) have more vulnerabilities?
4. How can GraphQL batch queries bypass rate limiting?
5. What is the OWASP API Security Top 10 and how does it differ from the web Top 10?
6. How do you discover undocumented API endpoints?

---

*Previous: [04-database-interactions.md](04-database-interactions.md) | Next: [06-microservices.md](06-microservices.md)*

---

*[Back to README](../README.md)*
