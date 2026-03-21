# Unit 8: A08 - Software and Data Integrity Failures — Topic 4: Insecure Deserialization

## Overview

Insecure deserialization occurs when an application **deserializes untrusted data without validation**, allowing attackers to manipulate serialized objects to achieve remote code execution, authentication bypass, privilege escalation, or denial of service. Deserialization vulnerabilities are often **critical severity** because they frequently lead to RCE.

---

## 1. Serialization and Deserialization

```
SERIALIZATION: Convert object → byte stream (for storage/transport)
DESERIALIZATION: Convert byte stream → object (reconstruct)

Object ──serialize──→ Byte Stream ──deserialize──→ Object

Languages with native serialization:
- Java: ObjectInputStream / ObjectOutputStream
- Python: pickle / marshal
- PHP: serialize() / unserialize()
- .NET: BinaryFormatter / DataContractSerializer
- Ruby: Marshal.dump / Marshal.load
```

---

## 2. How Deserialization Attacks Work

```
┌─────────────────────────────────────────────────────────────────┐
│           DESERIALIZATION ATTACK FLOW                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Application serializes objects (session, API data, cache)  │
│  2. Serialized data is accessible to user (cookie, form, API) │
│  3. Attacker modifies serialized data or crafts malicious one  │
│  4. Application deserializes without validation                │
│  5. During deserialization, "magic methods" execute            │
│     (constructors, toString, readObject, __wakeup)            │
│  6. Attacker chains gadgets to achieve code execution          │
│                                                                 │
│  GADGET CHAIN:                                                 │
│  A → B → C → Runtime.exec("malicious command")               │
│  Each class's magic method calls the next                     │
│  Final class executes arbitrary code                          │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Language-Specific Attacks

### Python (pickle)

```python
# ❌ VULNERABLE — Deserializing untrusted pickle
import pickle
import base64

# Attacker creates malicious pickle
import os

class Exploit:
    def __reduce__(self):
        # __reduce__ defines how to reconstruct the object
        # It returns a callable and its arguments
        return (os.system, ("whoami",))

# Serialize the exploit
payload = pickle.dumps(Exploit())
encoded = base64.b64encode(payload).decode()

# When server does this with attacker's data:
data = base64.b64decode(user_input)
obj = pickle.loads(data)  # EXECUTES os.system("whoami")!

# ✅ SECURE — Use JSON instead
import json
data = json.loads(user_input)  # Safe — only data, no code execution
```

### Java

```java
// ❌ VULNERABLE — Deserializing untrusted ObjectInputStream
ObjectInputStream ois = new ObjectInputStream(request.getInputStream());
Object obj = ois.readObject();  // Executes gadget chains!

// Common Java deserialization gadget libraries:
// - Apache Commons Collections
// - Spring Framework
// - Apache Commons BeanUtils

// Tools to generate payloads:
// ysoserial — Java deserialization exploit generator
// java -jar ysoserial.jar CommonsCollections1 "whoami" > payload.bin

// ✅ SECURE — Use JSON with Jackson
ObjectMapper mapper = new ObjectMapper();
// Disable polymorphic deserialization
mapper.deactivateDefaultTyping();
UserDTO user = mapper.readValue(input, UserDTO.class);
```

### PHP

```php
// ❌ VULNERABLE — unserialize() with user input
$data = unserialize($_COOKIE['user_data']);

// PHP magic methods exploited:
// __wakeup() — Called during unserialize
// __destruct() — Called when object destroyed
// __toString() — Called on string conversion

// Attacker crafts serialized object:
// O:4:"User":2:{s:4:"role";s:5:"admin";s:4:"name";s:5:"hacker";}

// ✅ SECURE — Use json_decode
$data = json_decode($_COOKIE['user_data'], true);
```

---

## 4. Detection and Prevention

```
PREVENTION STRATEGIES:

1. DON'T DESERIALIZE UNTRUSTED DATA
   → Use JSON, XML, or Protocol Buffers instead
   → No code execution during parsing

2. IF YOU MUST DESERIALIZE:
   → Implement integrity checks (HMAC/digital signature)
   → Whitelist allowed classes
   → Use look-ahead deserialization
   → Run in sandboxed environment

3. JAVA-SPECIFIC:
   → Use ObjectInputFilter (Java 9+)
   → Use NotSoSerial or SerialKiller agents
   → Remove unnecessary gadget libraries

4. MONITORING:
   → Log deserialization events
   → Alert on deserialization exceptions
   → Monitor for known exploit patterns
```

```java
// Java — ObjectInputFilter (whitelist classes)
ObjectInputFilter filter = ObjectInputFilter.Config.createFilter(
    "com.myapp.model.*;!*"  // Allow only com.myapp.model classes
);

ObjectInputStream ois = new ObjectInputStream(inputStream);
ois.setObjectInputFilter(filter);
Object obj = ois.readObject();  // Only allowed classes can deserialize
```

---

## 5. Tools

```bash
# ysoserial — Java deserialization payload generator
java -jar ysoserial.jar CommonsCollections1 "id" | base64

# Java Deserialization Scanner (Burp extension)
# Automatically tests for deserialization vulnerabilities

# Python — fickling (pickle analysis)
pip install fickling
fickling --check suspicious.pkl
# Analyzes pickle files for malicious code
```

---

## Summary Table

| Language | Dangerous Function | Safe Alternative |
|----------|-------------------|------------------|
| Python | `pickle.loads()` | `json.loads()` |
| Java | `ObjectInputStream.readObject()` | Jackson/Gson JSON |
| PHP | `unserialize()` | `json_decode()` |
| .NET | `BinaryFormatter.Deserialize()` | `System.Text.Json` |
| Ruby | `Marshal.load()` | `JSON.parse()` |

---

## Revision Questions

1. What is deserialization and why is it dangerous with untrusted input?
2. Create a Python pickle exploit that executes a command. Then show the secure alternative.
3. What are gadget chains in Java deserialization? Name common vulnerable libraries.
4. How does ObjectInputFilter (Java 9+) prevent deserialization attacks?
5. Why is JSON deserialization safer than native object deserialization?

---

*Previous: [03-unsigned-updates.md](03-unsigned-updates.md) | Next: [05-code-signing.md](05-code-signing.md)*

---

*[Back to README](../README.md)*
