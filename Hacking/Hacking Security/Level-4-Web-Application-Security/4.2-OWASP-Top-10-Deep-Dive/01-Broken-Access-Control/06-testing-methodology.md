# Unit 1: A01 - Broken Access Control — Topic 6: Testing Methodology

## Overview

Systematic access control testing requires a **structured, repeatable methodology** that covers horizontal access, vertical privilege escalation, function-level controls, and API endpoint authorization. This topic provides step-by-step checklists, Burp Suite extension workflows, testing matrices, and integration with the OWASP Testing Guide (WSTG) for comprehensive access control validation.

---

## 1. Access Control Testing Framework

```
┌─────────────────────────────────────────────────────────────────┐
│              ACCESS CONTROL TESTING PHASES                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Phase 1: RECONNAISSANCE                                       │
│  ├─ Map all roles and privilege levels                         │
│  ├─ Identify all endpoints/resources                            │
│  ├─ Understand business logic and data flow                    │
│  └─ Create test accounts for each role                         │
│                                                                 │
│  Phase 2: HORIZONTAL TESTING                                   │
│  ├─ Access User B's resources with User A's session            │
│  ├─ Test all CRUD operations per resource type                 │
│  └─ Test parameter-based references (IDOR)                     │
│                                                                 │
│  Phase 3: VERTICAL TESTING                                     │
│  ├─ Access admin functions with regular user session           │
│  ├─ Test role escalation through parameter tampering           │
│  └─ Test JWT/session token manipulation                         │
│                                                                 │
│  Phase 4: FUNCTION-LEVEL TESTING                               │
│  ├─ Test every API endpoint with each role                     │
│  ├─ Test HTTP method variations (GET/POST/PUT/DELETE)          │
│  └─ Test admin UI vs API consistency                           │
│                                                                 │
│  Phase 5: CONTEXT-BASED TESTING                                │
│  ├─ Test access from different IP addresses                    │
│  ├─ Test during different time windows                          │
│  └─ Test with expired/revoked sessions                         │
│                                                                 │
│  Phase 6: REPORTING                                            │
│  ├─ Document findings with reproduction steps                  │
│  ├─ Classify severity (CVSS scoring)                           │
│  └─ Provide remediation recommendations                        │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Role-Based Testing Matrix

### Setting Up Test Accounts

```
Test Account Matrix:
┌────────────────────────────────────────────────────────────┐
│  Role            │ Username     │ Purpose                  │
├──────────────────┼──────────────┼──────────────────────────┤
│  Super Admin     │ superadmin   │ Highest privilege        │
│  Admin           │ admin_test   │ Administrative access    │
│  Manager         │ manager_test │ Mid-level privileges     │
│  Regular User A  │ user_a       │ Standard user (attacker) │
│  Regular User B  │ user_b       │ Standard user (victim)   │
│  Read-Only User  │ readonly     │ Minimal permissions      │
│  Unauthenticated │ (none)       │ No session/token         │
└────────────────────────────────────────────────────────────┘
```

### Authorization Matrix Template

| Endpoint | Method | Super Admin | Admin | Manager | User (own) | User (other) | Unauth |
|----------|--------|:-----------:|:-----:|:-------:|:----------:|:------------:|:------:|
| `/api/users` | GET | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| `/api/users/{id}` | GET | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ |
| `/api/users/{id}` | PUT | ✅ | ✅ | ❌ | ✅ | ❌ | ❌ |
| `/api/users/{id}` | DELETE | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| `/api/admin/config` | GET | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| `/api/admin/config` | PUT | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| `/api/reports` | GET | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ |
| `/api/reports` | POST | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ |

> ✅ = Should be ALLOWED | ❌ = Should be DENIED
> Any ❌ that returns 200 OK = **VULNERABILITY**

---

## 3. Burp Suite Testing Workflow

### Autorize Extension Setup

```
Step 1: Install Autorize
  → Burp → Extender → BApp Store → Search "Autorize" → Install

Step 2: Configure Low-Privilege Cookies
  → Autorize tab → Paste low-privilege user's session cookies
  → Example: "session=abc123; token=xyz789"

Step 3: Configure Detection
  → Set enforcement detector patterns:
     Enforced: "403", "401", "Forbidden", "Unauthorized"
     Not Enforced: "200", response contains user data

Step 4: Enable Interception
  → Toggle Autorize ON

Step 5: Browse as Admin
  → Navigate the application as the high-privilege user
  → Autorize replays each request with low-privilege cookies

Step 6: Analyze Results
  → Color-coded results:
     🔴 Red = Authorization BYPASSED (vulnerability!)
     🟢 Green = Authorization ENFORCED (secure)
     🟡 Yellow = Needs manual review
```

### Manual Testing with Burp Repeater

```
Workflow:
1. Capture admin request in Proxy → History
2. Send to Repeater (Ctrl+R)
3. Replace admin cookies/token with low-priv user's credentials
4. Send request
5. Compare responses:
   - Same content as admin → VULNERABLE
   - 403/401 → Protected
   - Different content → Investigate further

Pro Tips:
- Use Comparer to diff admin vs user responses
- Check response size differences (similar size = suspicious)
- Test both cookies AND Authorization headers
- Test with completely expired/invalid sessions too
```

---

## 4. Step-by-Step Testing Checklist

### OWASP WSTG Access Control Tests

| Test ID | Test Name | Description |
|---------|-----------|-------------|
| WSTG-ATHZ-01 | Directory Traversal/File Include | Test path traversal in file parameters |
| WSTG-ATHZ-02 | Bypassing Authorization Schema | Test accessing pages without proper role |
| WSTG-ATHZ-03 | Privilege Escalation | Test vertical and horizontal escalation |
| WSTG-ATHZ-04 | IDOR | Test object reference manipulation |

### Comprehensive Checklist

```
ACCESS CONTROL TESTING CHECKLIST
═══════════════════════════════════════════════════

□ 1. ENDPOINT DISCOVERY
  □ Map all URLs from sitemap/crawling
  □ Identify admin/internal endpoints
  □ Check robots.txt, sitemap.xml
  □ Review JavaScript for hidden API endpoints
  □ Fuzz for undocumented endpoints

□ 2. AUTHENTICATION BOUNDARY
  □ Access all endpoints without authentication
  □ Access with expired session tokens
  □ Access with invalid session tokens
  □ Test session fixation

□ 3. HORIZONTAL ACCESS CONTROL
  □ Access User B's profile with User A's session
  □ Access User B's orders/invoices/documents
  □ Modify User B's data (PUT/PATCH)
  □ Delete User B's resources
  □ Test across all resource types

□ 4. VERTICAL ACCESS CONTROL
  □ Access admin panel with regular user
  □ Call admin API endpoints with regular token
  □ Modify role/privilege parameters
  □ Test mass assignment on user objects
  □ Attempt admin operations (user create/delete)

□ 5. FUNCTION-LEVEL ACCESS
  □ Test each HTTP method per endpoint
  □ Test method override headers
  □ Compare UI-available vs API-available actions
  □ Test bulk operations
  □ Test import/export functions

□ 6. DATA-LEVEL ACCESS
  □ Test query parameter manipulation
  □ Test path parameter manipulation
  □ Test POST body parameter manipulation
  □ Test filter/search bypass
  □ Test pagination to access restricted records

□ 7. TOKEN/SESSION MANIPULATION
  □ JWT algorithm confusion
  □ JWT claim manipulation
  □ Session token prediction
  □ Cookie manipulation
  □ Token replay after logout

□ 8. CONTEXT-BASED CONTROLS
  □ IP-based restrictions
  □ Time-based restrictions
  □ Geographic restrictions
  □ Device-based restrictions
  □ Rate limiting effectiveness
```

---

## 5. Automated Testing Scripts

### Access Control Matrix Tester

```python
#!/usr/bin/env python3
"""
Automated Access Control Matrix Tester
Tests all endpoints against all roles to build authorization matrix
"""
import requests
import json
from tabulate import tabulate

class AccessControlTester:
    def __init__(self, base_url):
        self.base_url = base_url
        self.results = []
    
    def add_role(self, name, headers):
        """Register a role with its authentication headers"""
        if not hasattr(self, 'roles'):
            self.roles = {}
        self.roles[name] = headers
    
    def add_endpoint(self, method, path, expected_access):
        """Register an endpoint with expected access per role"""
        if not hasattr(self, 'endpoints'):
            self.endpoints = []
        self.endpoints.append({
            'method': method,
            'path': path,
            'expected': expected_access  # {role: True/False}
        })
    
    def run_tests(self):
        """Execute all access control tests"""
        print(f"\n{'='*70}")
        print(f"  ACCESS CONTROL MATRIX TEST")
        print(f"  Target: {self.base_url}")
        print(f"  Roles: {', '.join(self.roles.keys())}")
        print(f"  Endpoints: {len(self.endpoints)}")
        print(f"{'='*70}\n")
        
        for endpoint in self.endpoints:
            for role_name, headers in self.roles.items():
                url = f"{self.base_url}{endpoint['path']}"
                method = endpoint['method'].lower()
                
                try:
                    resp = getattr(requests, method)(
                        url, headers=headers, verify=False, timeout=10
                    )
                    
                    # Determine if access was granted
                    access_granted = resp.status_code in (200, 201, 204)
                    expected = endpoint['expected'].get(role_name, False)
                    
                    status = "✅ PASS" if access_granted == expected else "❌ FAIL"
                    
                    if access_granted and not expected:
                        status = "🔴 VULN"  # Access should be denied but was granted
                    
                    self.results.append({
                        'endpoint': f"{endpoint['method']} {endpoint['path']}",
                        'role': role_name,
                        'expected': 'ALLOW' if expected else 'DENY',
                        'actual': resp.status_code,
                        'status': status,
                    })
                    
                except Exception as e:
                    self.results.append({
                        'endpoint': f"{endpoint['method']} {endpoint['path']}",
                        'role': role_name,
                        'expected': 'ALLOW' if endpoint['expected'].get(role_name) else 'DENY',
                        'actual': 'ERROR',
                        'status': f'⚠️ {str(e)[:30]}',
                    })
        
        return self.results
    
    def print_report(self):
        """Print results as a table"""
        headers = ['Endpoint', 'Role', 'Expected', 'Actual', 'Status']
        rows = [[r['endpoint'], r['role'], r['expected'], r['actual'], r['status']] 
                for r in self.results]
        print(tabulate(rows, headers=headers, tablefmt='grid'))
        
        vulns = [r for r in self.results if '🔴' in r['status']]
        if vulns:
            print(f"\n🔴 FOUND {len(vulns)} ACCESS CONTROL VIOLATIONS!")
            for v in vulns:
                print(f"  → {v['endpoint']} accessible by {v['role']}")

# Usage
tester = AccessControlTester('https://target.com')

# Register roles
tester.add_role('admin', {'Authorization': 'Bearer admin_token'})
tester.add_role('user', {'Authorization': 'Bearer user_token'})
tester.add_role('unauth', {})

# Register endpoints with expected access
tester.add_endpoint('GET', '/api/admin/users', {'admin': True, 'user': False, 'unauth': False})
tester.add_endpoint('DELETE', '/api/admin/users/1', {'admin': True, 'user': False, 'unauth': False})
tester.add_endpoint('GET', '/api/profile', {'admin': True, 'user': True, 'unauth': False})

tester.run_tests()
tester.print_report()
```

---

## 6. API Endpoint Authorization Testing

### REST API Testing Pattern

```bash
# For each API endpoint, test with each role:
ADMIN_TOKEN="Bearer eyJ..."
USER_TOKEN="Bearer eyJ..."
ENDPOINTS=(
    "GET /api/users"
    "POST /api/users"
    "GET /api/users/1"
    "PUT /api/users/1"
    "DELETE /api/users/1"
    "GET /api/admin/config"
    "PUT /api/admin/config"
)

for endpoint in "${ENDPOINTS[@]}"; do
    METHOD=$(echo "$endpoint" | cut -d' ' -f1)
    PATH=$(echo "$endpoint" | cut -d' ' -f2)
    
    echo "=== Testing $METHOD $PATH ==="
    
    # Test as admin
    admin_code=$(curl -s -o /dev/null -w "%{http_code}" \
        -X "$METHOD" "https://target.com$PATH" \
        -H "Authorization: $ADMIN_TOKEN")
    
    # Test as user
    user_code=$(curl -s -o /dev/null -w "%{http_code}" \
        -X "$METHOD" "https://target.com$PATH" \
        -H "Authorization: $USER_TOKEN")
    
    # Test unauthenticated
    unauth_code=$(curl -s -o /dev/null -w "%{http_code}" \
        -X "$METHOD" "https://target.com$PATH")
    
    echo "  Admin: $admin_code | User: $user_code | Unauth: $unauth_code"
    
    # Flag issues
    [ "$user_code" = "200" ] && [ "$admin_code" = "200" ] && \
        echo "  ⚠️  User has admin-level access!"
    [ "$unauth_code" = "200" ] && \
        echo "  ⚠️  Endpoint accessible without authentication!"
done
```

### GraphQL Authorization Testing

```bash
# Test GraphQL queries with different auth levels
# Admin query
curl -s -X POST https://target.com/graphql \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $ADMIN_TOKEN" \
    -d '{"query": "{ users { id email role } }"}'

# Same query as regular user
curl -s -X POST https://target.com/graphql \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $USER_TOKEN" \
    -d '{"query": "{ users { id email role } }"}'

# Introspection query (often leaks schema)
curl -s -X POST https://target.com/graphql \
    -H "Content-Type: application/json" \
    -d '{"query": "{ __schema { types { name fields { name } } } }"}'
```

---

## 7. Automated vs Manual Testing

| Aspect | Automated | Manual |
|--------|-----------|--------|
| **Speed** | Fast — tests hundreds of endpoints | Slow — one test at a time |
| **Coverage** | Broad — systematic matrix testing | Deep — logic flaws |
| **False Positives** | Higher — needs tuning | Lower — human judgment |
| **Business Logic** | Cannot test complex workflows | Excels at logic testing |
| **Reproducibility** | Fully reproducible | Depends on documentation |
| **Best For** | Regression, known patterns | New apps, complex flows |
| **Tools** | Autorize, custom scripts, Nuclei | Burp Repeater, browser |

### Recommended Approach

```
1. AUTOMATED FIRST (breadth):
   → Run Autorize against all endpoints
   → Run custom access control matrix tester
   → Scan with Nuclei access control templates

2. MANUAL FOLLOW-UP (depth):
   → Investigate automated findings
   → Test business logic flows
   → Test multi-step operations
   → Test edge cases and race conditions

3. CONTINUOUS (regression):
   → Add access control tests to CI/CD
   → Re-run matrix after each release
   → Monitor for new endpoints
```

---

## Summary Table

| Testing Phase | Focus | Key Tools |
|--------------|-------|-----------|
| **Recon** | Map roles, endpoints, data flow | Burp Spider, Swagger docs |
| **Horizontal** | Cross-user data access | Autorize, custom scripts |
| **Vertical** | Role escalation | Burp Repeater, jwt_tool |
| **Function-level** | HTTP method/endpoint coverage | Access control matrix tester |
| **Token** | JWT/session manipulation | jwt_tool, hashcat |
| **Automated** | Regression and scale | Nuclei, CI/CD integration |

---

## Revision Questions

1. Describe the complete access control testing methodology from reconnaissance to reporting.
2. What is an authorization matrix? Create one for a blog application with admin, editor, and reader roles.
3. How does the Burp Suite Autorize extension work? What are its setup steps and how does it detect authorization bypass?
4. Write a Python script that tests all endpoints in an API against multiple roles and generates a pass/fail report.
5. Compare automated vs manual access control testing. When should each approach be used?
6. List the OWASP WSTG test cases for authorization testing and explain each.

---

*Previous: [05-jwt-vulnerabilities.md](05-jwt-vulnerabilities.md) | Next: [07-prevention-and-remediation.md](07-prevention-and-remediation.md)*

---

*[Back to README](../README.md)*
