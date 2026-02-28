# Chapter 7.1: Jinja2 Templates

[← Previous: Role Testing](../06-Roles/06-role-testing.md) | [Next: Template Module →](02-template-module.md)

---

## Overview

**Jinja2** is the templating engine used by Ansible to generate dynamic configuration files, build strings, and evaluate expressions. Every Ansible template, variable reference, and conditional expression uses Jinja2 syntax.

---

## Jinja2 Syntax Basics

```
  JINJA2 DELIMITERS
  ══════════════════════════════════════════════════════════

  {{ expression }}       ◄── Output / print a value
  {% statement %}        ◄── Logic (if, for, set, etc.)
  {# comment #}         ◄── Comment (not rendered)
```

---

## Expressions (Output)

```jinja2
Hello, {{ username }}!
Server: {{ ansible_hostname }}
Port: {{ http_port | default(80) }}
IP: {{ ansible_default_ipv4.address }}
```

---

## Variables and Data Types

```jinja2
{# String #}
{{ "hello" }}

{# Integer #}
{{ 42 }}

{# Boolean #}
{{ true }}

{# List access #}
{{ my_list[0] }}
{{ my_list | first }}

{# Dictionary access #}
{{ my_dict.key }}
{{ my_dict['key'] }}

{# Nested #}
{{ servers.web.ip }}
{{ servers['web']['ip'] }}
```

---

## Control Structures

### if / elif / else

```jinja2
{% if enable_ssl %}
listen 443 ssl;
ssl_certificate /etc/ssl/certs/{{ domain }}.crt;
ssl_certificate_key /etc/ssl/private/{{ domain }}.key;
{% elif redirect_to_ssl %}
return 301 https://$host$request_uri;
{% else %}
listen 80;
{% endif %}
```

### for Loops

```jinja2
{% for server in upstream_servers %}
server {{ server.host }}:{{ server.port }} weight={{ server.weight | default(1) }};
{% endfor %}
```

Output:
```
server app1.example.com:8080 weight=2;
server app2.example.com:8080 weight=1;
server app3.example.com:8080 weight=1;
```

### Loop Variables

```jinja2
{% for user in users %}
{{ loop.index }}. {{ user.name }}    {# 1-based index #}
{{ loop.index0 }}                    {# 0-based index #}
{{ loop.first }}                     {# True on first iteration #}
{{ loop.last }}                      {# True on last iteration #}
{{ loop.length }}                    {# Total items #}
{{ loop.revindex }}                  {# Reverse count #}
{% endfor %}
```

### Loop with Conditional

```jinja2
{% for host in groups['webservers'] %}
{% if hostvars[host].ansible_default_ipv4 is defined %}
{{ hostvars[host].ansible_default_ipv4.address }}  {{ host }}
{% endif %}
{% endfor %}
```

---

## Whitespace Control

```jinja2
{# Problem: extra blank lines #}
{% for item in list %}
{{ item }}
{% endfor %}

{# Solution: use - to strip whitespace #}
{% for item in list -%}
{{ item }}
{%- endfor %}

{# - before %} strips whitespace before the tag #}
{# - after {%  strips whitespace after the tag #}
```

```
  WHITESPACE CONTROL
  ══════════════════════════════════════════════════════════

  {%- ... %}     Strip whitespace BEFORE the tag
  {% ... -%}     Strip whitespace AFTER the tag
  {%- ... -%}    Strip whitespace on BOTH sides
  
  Same applies to {{ }} and {# #}:
  {{- value -}}
  {#- comment -#}
```

---

## Filters

```jinja2
{# String filters #}
{{ name | upper }}                        JOHN
{{ name | lower }}                        john
{{ name | capitalize }}                   John
{{ name | title }}                        John Doe
{{ name | replace("old", "new") }}
{{ name | truncate(20) }}

{# Default values #}
{{ variable | default("fallback") }}
{{ variable | default(omit) }}           {# Skip param if undefined #}

{# Type conversion #}
{{ "42" | int }}
{{ 42 | string }}
{{ "true" | bool }}
{{ value | float }}

{# List filters #}
{{ list | join(", ") }}
{{ list | sort }}
{{ list | unique }}
{{ list | flatten }}
{{ list | length }}
{{ list | first }}
{{ list | last }}
{{ list | random }}
{{ list | difference(other_list) }}
{{ list | intersect(other_list) }}
{{ list | union(other_list) }}

{# Math #}
{{ value | abs }}
{{ list | max }}
{{ list | min }}
{{ list | sum }}
{{ value | round(2) }}

{# Data format #}
{{ data | to_json }}
{{ data | to_yaml }}
{{ data | to_nice_json }}
{{ data | to_nice_yaml }}
{{ data | from_json }}
{{ data | from_yaml }}

{# Hashing #}
{{ password | password_hash('sha512') }}
{{ string | hash('md5') }}
```

---

## Tests

```jinja2
{# Tests return true/false — use with 'is' #}
{% if variable is defined %}
{% if variable is undefined %}
{% if value is number %}
{% if value is string %}
{% if value is mapping %}         {# dict #}
{% if value is iterable %}
{% if value is none %}
{% if path is file %}
{% if path is directory %}
{% if loop.first is sameas true %}

{# Ansible-specific tests #}
{% if result is success %}
{% if result is failed %}
{% if result is changed %}
{% if result is skipped %}
{% if version is version('2.0', '>=') %}
```

---

## Macros (Reusable Blocks)

```jinja2
{% macro render_vhost(domain, port, root) %}
server {
    listen {{ port }};
    server_name {{ domain }};
    root {{ root }};
    
    location / {
        try_files $uri $uri/ =404;
    }
}
{% endmacro %}

{# Use the macro #}
{{ render_vhost("app1.example.com", 80, "/var/www/app1") }}
{{ render_vhost("app2.example.com", 8080, "/var/www/app2") }}
```

---

## Template Inheritance

### base.conf.j2

```jinja2
# Base configuration
{% block global_settings %}
worker_processes auto;
{% endblock %}

{% block http_settings %}
http {
    {% block server %}{% endblock %}
}
{% endblock %}
```

### site.conf.j2

```jinja2
{% extends "base.conf.j2" %}

{% block server %}
server {
    listen {{ http_port }};
    server_name {{ server_name }};
}
{% endblock %}
```

---

## Real-World Scenario: Dynamic Configuration

```yaml
# group_vars/webservers.yml
---
upstream_servers:
  - { host: "10.0.1.10", port: 8080, weight: 3 }
  - { host: "10.0.1.11", port: 8080, weight: 2 }
  - { host: "10.0.1.12", port: 8080, weight: 1 }

locations:
  - path: /api
    proxy_pass: "http://backend"
  - path: /static
    alias: /var/www/static
  - path: /health
    return: "200 'OK'"
```

```jinja2
# templates/nginx-site.conf.j2
# {{ ansible_managed }}
upstream backend {
{% for server in upstream_servers %}
    server {{ server.host }}:{{ server.port }} weight={{ server.weight }};
{% endfor %}
}

server {
    listen {{ http_port | default(80) }};
    server_name {{ server_name }};

{% for loc in locations %}
    location {{ loc.path }} {
{%   if loc.proxy_pass is defined %}
        proxy_pass {{ loc.proxy_pass }};
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
{%   elif loc.alias is defined %}
        alias {{ loc.alias }};
{%   elif loc.return is defined %}
        return {{ loc.return }};
{%   endif %}
    }

{% endfor %}
}
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| `UndefinedError` | Variable not defined — use `| default()` or check `is defined` |
| Extra blank lines in output | Use whitespace control: `{%- ... -%}` |
| Dict key with dot in name | Use bracket notation: `dict['key.name']` |
| String looks like YAML bool | Quote in YAML: `value: "yes"` or use `| string` |
| Jinja2 in a `when:` clause | Don't add `{{ }}` in `when:` — it's already Jinja2 context |

---

## Summary Table

| Syntax | Purpose |
|--------|---------|
| `{{ expr }}` | Output a value |
| `{% stmt %}` | Logic block |
| `{# ... #}` | Comment |
| `{% if %}` | Conditional |
| `{% for %}` | Loop |
| `{% macro %}` | Reusable template function |
| `{% extends %}` | Template inheritance |
| `| filter` | Transform value |
| `is test` | Evaluate condition |
| `{%- -%}` | Whitespace control |

---

## Quick Revision Questions

1. **What are the three Jinja2 delimiters?**
   > `{{ }}` for expressions, `{% %}` for statements, and `{# #}` for comments.

2. **How do you provide a default value for an undefined variable?**
   > Use the `default` filter: `{{ variable | default("fallback") }}`.

3. **How do you remove blank lines caused by Jinja2 blocks?**
   > Use whitespace control with the `-` character: `{%- ... -%}`.

4. **What is the `loop` variable in a `for` block?**
   > A special object providing iteration metadata: `loop.index`, `loop.first`, `loop.last`, `loop.length`.

5. **Why should you not use `{{ }}` inside a `when:` clause?**
   > The `when:` clause is already evaluated as a Jinja2 expression. Adding `{{ }}` causes double-evaluation errors.

---

[← Previous: Role Testing](../06-Roles/06-role-testing.md) | [Next: Template Module →](02-template-module.md)
