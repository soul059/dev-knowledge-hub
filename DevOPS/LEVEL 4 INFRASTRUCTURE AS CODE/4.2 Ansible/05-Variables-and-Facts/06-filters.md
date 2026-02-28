# Chapter 5.6: Filters

[← Previous: Lookups](05-lookups.md) | [Next: Fact Caching →](07-fact-caching.md)

---

## Overview

**Filters** transform variable data using the pipe (`|`) syntax from Jinja2. They're used to format strings, manipulate lists and dictionaries, handle defaults, perform math, convert types, and much more. Ansible includes both standard Jinja2 filters and many custom ones.

---

## Filter Syntax

```
  {{ variable | filter_name }}
  {{ variable | filter1 | filter2 | filter3 }}    ◄── Chaining
  {{ variable | filter(arg1, arg2) }}              ◄── With arguments
```

---

## String Filters

```yaml
vars:
  name: "hello world"
  path: "/etc/nginx/nginx.conf"

tasks:
  - debug:
      msg: |
        upper:      {{ name | upper }}                # HELLO WORLD
        lower:      {{ name | lower }}                # hello world
        title:      {{ name | title }}                # Hello World
        capitalize: {{ name | capitalize }}            # Hello world
        replace:    {{ name | replace('world','ansible') }}  # hello ansible
        trim:       {{ "  hello  " | trim }}          # hello
        length:     {{ name | length }}                # 11
        reverse:    {{ name | reverse }}               # dlrow olleh
        center:     {{ name | center(20) }}           #     hello world
        truncate:   {{ name | truncate(5, True) }}    # he...
        basename:   {{ path | basename }}              # nginx.conf
        dirname:    {{ path | dirname }}               # /etc/nginx
        splitext:   {{ path | splitext }}              # ['/etc/nginx/nginx', '.conf']
```

### Regex Filters

```yaml
tasks:
  - debug:
      msg: |
        search:  {{ 'server123' | regex_search('\\d+') }}           # 123
        replace: {{ 'foo-123' | regex_replace('\\d+', 'NUM') }}     # foo-NUM
        match:   {{ 'hello' | regex_search('^hel') is not none }}   # True
        findall: {{ 'a1b2c3' | regex_findall('\\d') }}              # ['1','2','3']
```

---

## Default Values

```yaml
tasks:
  # Provide fallback for undefined variables
  - debug:
      msg: "Port: {{ http_port | default(80) }}"

  # Default for empty string (not just undefined)
  - debug:
      msg: "Name: {{ user_name | default('anonymous', true) }}"
      # 'true' means: also use default if value is empty/false/0

  # Mandatory — fail if undefined
  - debug:
      msg: "{{ required_var | mandatory }}"
```

---

## Type Conversion Filters

```yaml
tasks:
  - debug:
      msg: |
        to int:    {{ "42" | int }}                    # 42
        to float:  {{ "3.14" | float }}                # 3.14
        to bool:   {{ "yes" | bool }}                  # True
        to string: {{ 42 | string }}                   # "42"
        to json:   {{ mydict | to_json }}              # JSON string
        to yaml:   {{ mydict | to_yaml }}              # YAML string
        from json: {{ '{"a":1}' | from_json }}         # dict
        from yaml: {{ 'a: 1' | from_yaml }}            # dict
        to nice json: {{ mydict | to_nice_json(indent=2) }}
        to nice yaml: {{ mydict | to_nice_yaml(indent=2) }}
```

---

## List Filters

```yaml
vars:
  numbers: [3, 1, 4, 1, 5, 9, 2, 6]
  names: [alice, bob, charlie]
  nested: [[1,2], [3,4], [5,6]]

tasks:
  - debug:
      msg: |
        sort:       {{ numbers | sort }}               # [1,1,2,3,4,5,6,9]
        reverse:    {{ numbers | reverse | list }}     # [6,2,9,5,1,4,1,3]
        unique:     {{ numbers | unique }}              # [3,1,4,5,9,2,6]
        min:        {{ numbers | min }}                 # 1
        max:        {{ numbers | max }}                 # 9
        sum:        {{ numbers | sum }}                 # 31
        length:     {{ numbers | length }}              # 8
        first:      {{ numbers | first }}               # 3
        last:       {{ numbers | last }}                # 6
        flatten:    {{ nested | flatten }}               # [1,2,3,4,5,6]
        join:       {{ names | join(', ') }}            # alice, bob, charlie
        random:     {{ names | random }}                 # (random element)
        shuffle:    {{ numbers | shuffle }}              # (random order)

  # Set operations
  - debug:
      msg: |
        union:      {{ [1,2,3] | union([3,4,5]) }}         # [1,2,3,4,5]
        intersect:  {{ [1,2,3] | intersect([2,3,4]) }}     # [2,3]
        difference: {{ [1,2,3] | difference([2,3,4]) }}    # [1]
        symmetric:  {{ [1,2,3] | symmetric_difference([2,3,4]) }}  # [1,4]
```

---

## Dictionary Filters

```yaml
vars:
  user:
    name: alice
    uid: 1001
    shell: /bin/bash
  defaults:
    shell: /bin/sh
    home: /home

tasks:
  - debug:
      msg: |
        combine:    {{ defaults | combine(user) }}
        dict2items: {{ user | dict2items }}
        # [{key:name, value:alice}, {key:uid, value:1001}, ...]

  # items2dict (reverse of dict2items)
  - set_fact:
      new_dict: "{{ [{key:'a',value:1},{key:'b',value:2}] | items2dict }}"
      # {a: 1, b: 2}

  # Deep merge
  - set_fact:
      merged: "{{ dict1 | combine(dict2, recursive=True) }}"
```

---

## Math Filters

```yaml
tasks:
  - debug:
      msg: |
        abs:    {{ -5 | abs }}                         # 5
        round:  {{ 3.7 | round }}                      # 4.0
        ceil:   {{ 3.1 | round(0, 'ceil') | int }}     # 4
        floor:  {{ 3.9 | round(0, 'floor') | int }}    # 3
        log:    {{ 1000 | log(10) }}                    # 3.0
        pow:    {{ 2 | pow(10) }}                       # 1024.0
        root:   {{ 144 | root }}                        # 12.0

  # Calculate percentage
  - debug:
      msg: "Memory used: {{ ((ansible_memtotal_mb - ansible_memfree_mb) / ansible_memtotal_mb * 100) | round(1) }}%"
```

---

## Hash and Encoding Filters

```yaml
tasks:
  - debug:
      msg: |
        sha256:    {{ 'hello' | hash('sha256') }}
        md5:       {{ 'hello' | hash('md5') }}
        b64encode: {{ 'hello' | b64encode }}           # aGVsbG8=
        b64decode: {{ 'aGVsbG8=' | b64decode }}         # hello
        password:  {{ 'secret' | password_hash('sha512') }}
        urlencode: {{ 'hello world' | urlencode }}     # hello+world
```

---

## Selecting from Lists of Dictionaries

```yaml
vars:
  users:
    - name: alice
      role: admin
      active: true
    - name: bob
      role: developer
      active: false
    - name: charlie
      role: admin
      active: true

tasks:
  # Get all admin users
  - debug:
      msg: "{{ users | selectattr('role', 'eq', 'admin') | list }}"
      # [alice (admin, active), charlie (admin, active)]

  # Get names of active users
  - debug:
      msg: "{{ users | selectattr('active') | map(attribute='name') | list }}"
      # [alice, charlie]

  # Reject inactive users
  - debug:
      msg: "{{ users | rejectattr('active', 'eq', false) | list }}"

  # Get first match
  - debug:
      msg: "{{ users | selectattr('name', 'eq', 'bob') | first }}"
```

---

## IP Address Filters

```yaml
tasks:
  - debug:
      msg: |
        validate: {{ '192.168.1.1' | ansible.utils.ipaddr }}   # 192.168.1.1
        network:  {{ '192.168.1.0/24' | ansible.utils.ipaddr('network') }}
        netmask:  {{ '192.168.1.0/24' | ansible.utils.ipaddr('netmask') }}
        host:     {{ '192.168.1.5/24' | ansible.utils.ipaddr('address') }}
        # Requires ansible.utils collection
```

---

## Path Filters

```yaml
vars:
  file_path: /etc/nginx/sites-available/default.conf

tasks:
  - debug:
      msg: |
        basename:   {{ file_path | basename }}          # default.conf
        dirname:    {{ file_path | dirname }}            # /etc/nginx/sites-available
        expanduser: {{ '~/.ssh' | expanduser }}         # /home/user/.ssh
        realpath:   {{ '/etc/motd' | realpath }}        # /etc/motd (resolved symlinks)
        relpath:    {{ '/etc/nginx' | relpath('/etc') }} # nginx
        splitext:   {{ 'file.tar.gz' | splitext }}      # ['file.tar', '.gz']
```

---

## Ternary Filter

```yaml
tasks:
  # Like a ternary operator: condition ? true : false
  - debug:
      msg: "{{ (env == 'production') | ternary('PROD', 'NON-PROD') }}"

  - name: Set config based on environment
    set_fact:
      debug_mode: "{{ (env == 'development') | ternary('true', 'false') }}"
      log_level: "{{ (env == 'production') | ternary('warn', 'debug') }}"
      workers: "{{ (env == 'production') | ternary(8, 2) }}"
```

---

## Real-World Scenario: Dynamic Configuration with Filters

```yaml
---
- name: Generate dynamic configs
  hosts: appservers
  vars:
    all_services:
      - { name: web, port: 80, enabled: true, env: [staging, production] }
      - { name: api, port: 8080, enabled: true, env: [staging, production] }
      - { name: debug, port: 9090, enabled: false, env: [staging] }
      - { name: admin, port: 8443, enabled: true, env: [production] }
  tasks:
    - name: Get services for this environment
      set_fact:
        active_services: >-
          {{ all_services
             | selectattr('enabled')
             | selectattr('env', 'contains', env)
             | list }}

    - name: Generate service configs
      template:
        src: service.conf.j2
        dest: "/etc/app/{{ item.name }}.conf"
      loop: "{{ active_services }}"
      loop_control:
        label: "{{ item.name }}"

    - name: Show summary
      debug:
        msg: |
          Environment: {{ env }}
          Active services: {{ active_services | map(attribute='name') | join(', ') }}
          Ports in use: {{ active_services | map(attribute='port') | sort | join(', ') }}
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Filter not found | Check filter name; some are in collections (e.g., `ansible.utils.ipaddr`) |
| `int` filter on non-numeric string | Use `default(0)` first: `var \| default('0') \| int` |
| `selectattr` returns generator | Add `\| list` to materialize the result |
| JSON output not formatted | Use `to_nice_json(indent=2)` instead of `to_json` |
| `combine` overwrites nested keys | Use `combine(dict2, recursive=True)` for deep merge |
| Pipeline too complex | Break into multiple `set_fact` calls for readability |

---

## Summary Table

| Category | Key Filters |
|----------|-------------|
| String | `upper`, `lower`, `replace`, `trim`, `regex_replace` |
| Default | `default()`, `mandatory` |
| Type | `int`, `float`, `bool`, `string`, `to_json`, `from_json` |
| List | `sort`, `unique`, `flatten`, `join`, `first`, `last`, `min`, `max` |
| Set | `union`, `intersect`, `difference` |
| Dict | `combine`, `dict2items`, `items2dict` |
| Math | `round`, `abs`, `log`, `pow`, `root` |
| Hash | `hash('sha256')`, `b64encode`, `password_hash` |
| Selection | `selectattr`, `rejectattr`, `map(attribute=)` |
| Path | `basename`, `dirname`, `expanduser`, `realpath` |
| Logic | `ternary`, `bool` |

---

## Quick Revision Questions

1. **How do you provide a default value for an undefined variable?**
   > `{{ var | default('fallback') }}`. Use `default('val', true)` to also catch empty strings.

2. **How do you merge two dictionaries?**
   > `{{ dict1 | combine(dict2) }}`. For deep merge: `combine(dict2, recursive=True)`.

3. **How do you filter a list of dictionaries by an attribute?**
   > `{{ list | selectattr('key', 'eq', 'value') | list }}`.

4. **How do you extract a specific attribute from a list of dictionaries?**
   > `{{ list | map(attribute='name') | list }}`.

5. **What is the ternary filter?**
   > `{{ condition | ternary('if_true', 'if_false') }}` — equivalent to a ternary operator.

6. **How do you hash a string in Ansible?**
   > `{{ 'mystring' | hash('sha256') }}` or `{{ 'password' | password_hash('sha512') }}` for system passwords.

---

[← Previous: Lookups](05-lookups.md) | [Next: Fact Caching →](07-fact-caching.md)
