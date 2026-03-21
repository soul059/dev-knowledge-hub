# Unit 6: Cross-Site Scripting (XSS) — Topic 6: XSS Prevention

## Overview

Preventing XSS requires a defence-in-depth strategy that combines output encoding, input validation, context-aware escaping, and security headers. No single technique is sufficient — effective prevention demands understanding where and how user data is rendered in the application. This topic covers the core prevention mechanisms, framework-specific protections, and sanitisation libraries that form a robust anti-XSS architecture.

---

## 1. Output Encoding / Escaping

The primary defence against XSS. Output encoding converts dangerous characters into safe representations before they are rendered in the browser.

### Encoding Rules by Context

```
+--------------------+----------------------------+---------------------------+
|  Output Context    |  Encoding Method           |  Characters Encoded       |
+--------------------+----------------------------+---------------------------+
|  HTML Body         |  HTML Entity Encoding      |  & < > " '               |
|  HTML Attribute    |  Attribute Encoding        |  & < > " ' ` / = space   |
|  JavaScript        |  JavaScript Hex Encoding   |  All non-alphanumeric     |
|  URL Parameter     |  URL / Percent Encoding    |  All non-unreserved chars |
|  CSS Value         |  CSS Hex Encoding          |  All non-alphanumeric     |
+--------------------+----------------------------+---------------------------+
```

### HTML Entity Encoding

```
Character   Entity        Example
---------   ------        -------
<           &lt;          <script>  →  &lt;script&gt;
>           &gt;
"           &quot;        "onclick  →  &quot;onclick
'           &#x27;
&           &amp;         &lt;      →  &amp;lt;
```

### Encoding Flow Diagram

```
User Input:  <script>alert(1)</script>
      |
      v
+------------------+
| Output Encoder   |
| (Context-aware)  |
+--------+---------+
         |
         v
HTML Body Output:  &lt;script&gt;alert(1)&lt;/script&gt;

Browser renders:  <script>alert(1)</script>  (as visible TEXT, not code)
```

---

## 2. Input Validation

Input validation is a complementary defence — it reduces the attack surface but should never be the sole protection.

### Validation Strategies

```
+---------------------+--------------------------------------+
| Strategy            | Description                          |
+---------------------+--------------------------------------+
| Whitelist (prefer)  | Accept only known-good patterns      |
| Blacklist (avoid)   | Block known-bad patterns (bypassable)|
| Type checking       | Enforce expected data types          |
| Length limiting      | Restrict input length                |
| Format validation   | Regex for expected formats           |
+---------------------+--------------------------------------+
```

### Examples

```python
import re

# Whitelist: Only allow alphanumeric usernames
def validate_username(username):
    if re.match(r'^[a-zA-Z0-9_]{3,20}$', username):
        return True
    return False

# Type checking: Ensure ID is integer
def validate_id(user_id):
    try:
        return int(user_id)
    except ValueError:
        raise ValidationError("ID must be an integer")

# Format validation: Email
def validate_email(email):
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return bool(re.match(pattern, email))
```

### Input Validation vs Output Encoding

```
                        Input Validation          Output Encoding
                        ----------------          ---------------
When applied:           On data entry             On data output
Purpose:                Reject bad data           Neutralise bad data
Bypassable:             Partially (if blacklist)  No (if context-correct)
Sufficient alone:       NO                        YES (for that context)
Recommended:            As secondary defence      As primary defence
```

---

## 3. Context-Aware Encoding

The most critical concept: the same data requires different encoding depending on where it is placed in the output.

### Dangerous: Wrong Context Encoding

```html
<!-- Data HTML-encoded but placed in JavaScript context -->
<script>
  var name = "&lt;script&gt;";  // HTML encoding in JS context
  // Browser does NOT decode HTML entities inside <script> tags!
  // This may cause errors but doesn't prevent all attacks
</script>

<!-- Data JS-encoded but placed in HTML context -->
<p>\x3cscript\x3ealert(1)\x3c/script\x3e</p>
<!-- Browser doesn't interpret JS escapes in HTML — renders literally -->
```

### Correct: Context-Matched Encoding

```html
<!-- HTML body → HTML encode -->
<p>Hello, &lt;script&gt;alert(1)&lt;/script&gt;</p>

<!-- HTML attribute → Attribute encode -->
<input value="&quot; onfocus=&quot;alert(1)">

<!-- JavaScript → JS encode -->
<script>var name = "\x3cscript\x3ealert\x281\x29\x3c\x2fscript\x3e";</script>

<!-- URL parameter → URL encode -->
<a href="/search?q=%3Cscript%3Ealert(1)%3C%2Fscript%3E">Link</a>
```

### Context Decision Matrix

```
Where is user data rendered?
  |
  |-- HTML body text        → htmlEncode(data)
  |-- HTML attribute value  → attrEncode(data)
  |-- JavaScript string     → jsEncode(data)
  |-- URL parameter         → urlEncode(data)
  |-- CSS value             → cssEncode(data)
  |-- Inside <script> tag   → AVOID if possible; use JSON.stringify()
  |-- Inside HTML comment   → AVOID — strip or reject
```

---

## 4. Framework-Specific Protections

Modern frameworks provide automatic XSS protection — but developers must understand the boundaries.

### Framework Auto-Encoding

| Framework          | Auto-Encoding          | Unsafe Bypass                    |
|--------------------|------------------------|----------------------------------|
| **React**          | JSX auto-escapes       | `dangerouslySetInnerHTML`        |
| **Angular**        | Template auto-escapes  | `bypassSecurityTrustHtml()`      |
| **Vue.js**         | `{{ }}` auto-escapes   | `v-html` directive               |
| **Django**         | Template auto-escapes  | `|safe` filter, `{% autoescape off %}` |
| **Rails (ERB)**    | `<%= %>` auto-escapes  | `raw()`, `html_safe`             |
| **ASP.NET Razor**  | `@` auto-encodes       | `@Html.Raw()`                    |
| **Jinja2**         | Auto-escapes (if on)   | `|safe` filter                   |

### React Example

```jsx
// SAFE: React auto-escapes this
function Greeting({ name }) {
  return <p>Hello, {name}</p>;
}
// Input: <script>alert(1)</script>
// Output: <p>Hello, &lt;script&gt;alert(1)&lt;/script&gt;</p>

// UNSAFE: Bypasses React's protection
function Greeting({ name }) {
  return <p dangerouslySetInnerHTML={{__html: name}} />;
}
// Input: <script>alert(1)</script>
// Output: <p><script>alert(1)</script></p>  ← XSS!
```

### Django Example

```django
{# SAFE: Django auto-escapes template variables #}
<p>{{ user_input }}</p>
{# Output: &lt;script&gt;alert(1)&lt;/script&gt; #}

{# UNSAFE: |safe disables auto-escaping #}
<p>{{ user_input|safe }}</p>
{# Output: <script>alert(1)</script>  ← XSS! #}
```

---

## 5. Sanitisation Libraries

When you must accept HTML input (rich text editors, Markdown), use a proven sanitisation library — never write your own parser.

### DOMPurify (JavaScript)

```javascript
// Installation
// npm install dompurify

import DOMPurify from 'dompurify';

// Basic sanitisation
var clean = DOMPurify.sanitize('<img src=x onerror=alert(1)>');
// Result: <img src="x">  (onerror removed)

// Allow specific tags only
var clean = DOMPurify.sanitize(dirty, {
  ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p', 'br'],
  ALLOWED_ATTR: ['href', 'title']
});

// Strip all HTML
var clean = DOMPurify.sanitize(dirty, { ALLOWED_TAGS: [] });
```

### Bleach (Python)

```python
import bleach

# Allow only safe tags
clean = bleach.clean(
    user_input,
    tags=['b', 'i', 'a', 'p', 'br', 'ul', 'li'],
    attributes={'a': ['href', 'title']},
    protocols=['http', 'https']
)
```

### Sanitisation Library Comparison

| Library         | Language   | Approach          | Maintained | Notes                    |
|-----------------|------------|-------------------|------------|--------------------------|
| DOMPurify       | JavaScript | DOM-based parsing | Yes        | Most popular for JS      |
| Bleach          | Python     | Whitelist-based   | Yes        | Django compatible        |
| sanitize-html   | Node.js    | Whitelist-based   | Yes        | Server-side Node         |
| HtmlSanitizer   | .NET       | Whitelist-based   | Yes        | ASP.NET applications     |
| Loofah          | Ruby       | Multiple modes    | Yes        | Rails compatible         |

---

## 6. Additional Defence Layers

### HTTP Security Headers

```http
Content-Security-Policy: default-src 'self'; script-src 'self'
X-Content-Type-Options: nosniff
X-XSS-Protection: 0
```

> **Note:** `X-XSS-Protection` is deprecated. Modern browsers have removed their XSS auditors. Use CSP instead.

### Cookie Security Flags

```http
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Strict; Path=/
```

| Flag         | Protection                                     |
|--------------|-------------------------------------------------|
| `HttpOnly`   | Prevents JavaScript from reading the cookie     |
| `Secure`     | Cookie only sent over HTTPS                     |
| `SameSite`   | Prevents cross-site request cookie attachment    |

### Subresource Integrity (SRI)

```html
<script src="https://cdn.example.com/lib.js"
        integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/uxy9rx7HNQlGYl1kPzQho1wx4JwY8w"
        crossorigin="anonymous"></script>
```

---

## Summary Table

| Prevention Method         | Layer         | Effectiveness | Notes                             |
|---------------------------|---------------|---------------|-----------------------------------|
| Output encoding           | Application   | High          | Primary defence; context-aware    |
| Input validation          | Application   | Medium        | Secondary; use whitelists         |
| Framework auto-escaping   | Framework     | High          | Don't bypass with unsafe methods  |
| DOMPurify / sanitisation  | Application   | High          | For rich HTML input only          |
| Content Security Policy   | HTTP Header   | High          | Defence-in-depth; see next topic  |
| HttpOnly cookies          | HTTP Header   | Medium        | Prevents cookie theft only        |
| SRI                       | HTML          | Medium        | Protects against CDN compromise   |

---

## Revision Questions

1. **Why is output encoding considered the primary defence against XSS while input validation is secondary?**

2. **Explain why the same user data requires different encoding in an HTML body versus inside a JavaScript string literal.**

3. **A React developer uses `dangerouslySetInnerHTML` to render user-submitted blog content. What is the risk, and what alternative should they use?**

4. **Compare DOMPurify and Bleach. When would you use each, and what configuration is essential for both?**

5. **What is the danger of using a blacklist-based input filter (e.g., stripping `<script>`) as the sole XSS defence?**

6. **Describe a defence-in-depth strategy that combines at least four different XSS prevention techniques for a web application that accepts rich text input.**

---

*Previous: [05-impact-and-exploitation.md](05-impact-and-exploitation.md) | Next: [07-content-security-policy.md](07-content-security-policy.md)*

---

*[Back to README](../README.md)*
