# Chapter 5.5: Lookups

[← Previous: Vault (Encrypting Secrets)](04-vault.md) | [Next: Filters →](06-filters.md)

---

## Overview

**Lookups** retrieve data from external sources — files, environment variables, databases, APIs, password managers — and make it available in playbooks. They run on the **control node** (not on managed hosts) and are evaluated during template rendering.

---

## How Lookups Work

```
  LOOKUP EXECUTION
  ══════════════════════════════════════════════════════════

  ┌─────────────────────────────────────────────────┐
  │              CONTROL NODE                        │
  │                                                  │
  │  Playbook                     External Source    │
  │  ┌──────────────────┐       ┌────────────────┐  │
  │  │ {{ lookup(       │       │ Files          │  │
  │  │   'file',        │──────►│ Env vars       │  │
  │  │   '/etc/key')    │◄──────│ DNS            │  │
  │  │ }}               │       │ CSV            │  │
  │  └──────────────────┘       │ Passwords      │  │
  │         │                   │ APIs           │  │
  │         ▼                   └────────────────┘  │
  │   Returns string                                │
  │   (or list)                                      │
  └─────────────────────────────────────────────────┘
```

---

## Lookup Syntax

Two ways to call lookups:

```yaml
# 1. lookup() function — returns string
password: "{{ lookup('env', 'DB_PASSWORD') }}"
content: "{{ lookup('file', '/etc/ssl/cert.pem') }}"

# 2. query() function — returns list
hosts: "{{ query('lines', 'cat /etc/hosts') }}"

# 3. Shorthand with pipe lookup
message: "{{ 'Hello World' | upper }}"   # This is a filter, not lookup

# Default behavior:
# lookup() returns comma-separated string
# query() returns a list
# lookup(..., wantlist=True) also returns a list
```

---

## Common Lookups

### file — Read File Content

```yaml
tasks:
  - name: Deploy SSH key
    authorized_key:
      user: deploy
      key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"

  - name: Load config
    set_fact:
      ssl_cert: "{{ lookup('file', 'files/cert.pem') }}"
```

### env — Environment Variables

```yaml
tasks:
  - name: Use environment variable
    debug:
      msg: "Home: {{ lookup('env', 'HOME') }}"

  - name: Configure with env vars
    template:
      src: app.conf.j2
      dest: /etc/myapp/config.yml
    vars:
      db_password: "{{ lookup('env', 'DB_PASSWORD') }}"
      api_key: "{{ lookup('env', 'API_KEY') }}"
```

### template — Render Template

```yaml
tasks:
  - name: Generate config from template
    set_fact:
      rendered_config: "{{ lookup('template', 'config.j2') }}"
```

### pipe — Command Output

```yaml
tasks:
  - name: Get git commit hash
    debug:
      msg: "Commit: {{ lookup('pipe', 'git rev-parse HEAD') }}"

  - name: Get current date
    set_fact:
      deploy_date: "{{ lookup('pipe', 'date +%Y-%m-%d') }}"
```

### password — Generate/Retrieve Passwords

```yaml
tasks:
  # Generate random password and store in file
  - name: Create database password
    set_fact:
      db_password: "{{ lookup('password', '/tmp/db_password length=20 chars=ascii_letters,digits') }}"

  # Generate without storing
  - name: Random password (no file)
    set_fact:
      temp_password: "{{ lookup('password', '/dev/null length=16 chars=ascii_letters,digits,punctuation') }}"
```

### lines — Read Lines

```yaml
tasks:
  - name: Process each line
    debug:
      msg: "Line: {{ item }}"
    loop: "{{ query('lines', 'cat /etc/resolv.conf') }}"
```

### csvfile — Read CSV Data

```csv
# users.csv
username,uid,shell,groups
alice,1001,/bin/bash,admin
bob,1002,/bin/zsh,developers
charlie,1003,/bin/bash,developers
```

```yaml
tasks:
  - name: Get Alice's UID from CSV
    debug:
      msg: "UID: {{ lookup('csvfile', 'alice file=users.csv delimiter=, col=1') }}"
```

### ini — Read INI Files

```yaml
tasks:
  - name: Read from INI file
    debug:
      msg: "{{ lookup('ini', 'db_host section=database file=app.conf') }}"
```

### fileglob — Find Files by Pattern

```yaml
tasks:
  - name: Copy all config files
    copy:
      src: "{{ item }}"
      dest: /etc/myapp/conf.d/
    loop: "{{ query('fileglob', 'files/conf.d/*.conf') }}"
```

### sequence — Generate Number Sequence

```yaml
tasks:
  - name: Create numbered resources
    debug:
      msg: "Resource {{ item }}"
    loop: "{{ query('sequence', 'start=1 end=5 format=web%02d') }}"
    # Produces: web01, web02, web03, web04, web05
```

### url — Fetch from URL

```yaml
tasks:
  - name: Get data from API
    set_fact:
      api_response: "{{ lookup('url', 'https://api.example.com/config', headers={'Authorization': 'Bearer ' + api_token}) }}"

  - name: Parse JSON response
    set_fact:
      config: "{{ lookup('url', 'https://api.example.com/config') | from_json }}"
```

---

## Lookups Quick Reference

```
  ┌──────────────────┬─────────────────────────────────────┐
  │ Lookup           │ Description                         │
  ├──────────────────┼─────────────────────────────────────┤
  │ file             │ Read file contents                  │
  │ env              │ Read environment variable           │
  │ template         │ Render a Jinja2 template            │
  │ pipe             │ Run command, return stdout           │
  │ lines            │ Run command, return as list          │
  │ password         │ Generate/retrieve passwords         │
  │ csvfile          │ Read from CSV file                  │
  │ ini              │ Read from INI file                  │
  │ fileglob         │ Find files matching pattern         │
  │ first_found      │ Return first existing file          │
  │ sequence         │ Generate number sequence            │
  │ url              │ Fetch URL content                   │
  │ dict             │ Convert dict to key/value list      │
  │ subelements      │ Cross-product of list & subelement  │
  │ together         │ Zip multiple lists                  │
  │ random_choice    │ Pick random item from list          │
  │ dig              │ DNS lookup                          │
  │ aws_ssm          │ AWS Systems Manager Parameter Store │
  │ hashi_vault      │ HashiCorp Vault secrets             │
  └──────────────────┴─────────────────────────────────────┘
```

---

## lookup() vs query()

```yaml
# lookup() returns a comma-separated STRING
result: "{{ lookup('lines', 'echo -e \"a\nb\nc\"') }}"
# result = "a,b,c"

# query() returns a LIST
result: "{{ query('lines', 'echo -e \"a\nb\nc\"') }}"
# result = ["a", "b", "c"]

# lookup() with wantlist=True = same as query()
result: "{{ lookup('lines', 'echo -e \"a\nb\nc\"', wantlist=True) }}"
# result = ["a", "b", "c"]
```

Use `query()` (or `q()` shorthand) when you need a list, especially with `loop:`:

```yaml
- name: Process files
  copy:
    src: "{{ item }}"
    dest: /etc/app/conf.d/
  loop: "{{ q('fileglob', 'files/*.conf') }}"
```

---

## first_found — Fallback File Selection

```yaml
tasks:
  # Use the first file that exists
  - name: Load OS-specific vars
    include_vars:
      file: "{{ lookup('first_found', params) }}"
    vars:
      params:
        files:
          - "{{ ansible_distribution }}-{{ ansible_distribution_major_version }}.yml"
          - "{{ ansible_distribution }}.yml"
          - "{{ ansible_os_family }}.yml"
          - default.yml
        paths:
          - vars/
```

---

## Real-World Scenario: Secrets from External Source

```yaml
---
- name: Deploy with external secrets
  hosts: appservers
  become: yes
  vars:
    # From environment (CI/CD pipeline)
    deploy_token: "{{ lookup('env', 'DEPLOY_TOKEN') }}"
    
    # From file
    ssl_cert: "{{ lookup('file', 'certs/{{ env }}/cert.pem') }}"
    
    # Generated password (stored and reused)
    db_password: "{{ lookup('password', 'credentials/{{ env }}/db_pass length=32 chars=ascii_letters,digits') }}"
    
    # Git info for deployment tracking
    git_commit: "{{ lookup('pipe', 'git rev-parse --short HEAD') }}"
    git_branch: "{{ lookup('pipe', 'git rev-parse --abbrev-ref HEAD') }}"

  tasks:
    - name: Deploy application
      template:
        src: app-config.j2
        dest: /etc/myapp/config.yml
        mode: '0600'
      no_log: true

    - name: Tag deployment
      copy:
        content: |
          version: {{ app_version }}
          commit: {{ git_commit }}
          branch: {{ git_branch }}
          deployed: {{ lookup('pipe', 'date -u +%Y-%m-%dT%H:%M:%SZ') }}
          host: {{ inventory_hostname }}
        dest: /opt/myapp/DEPLOYMENT_INFO
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| "lookup plugin not found" | Check plugin name; install collection if needed |
| File lookup returns empty | Verify path is relative to playbook or use absolute path |
| Env lookup returns empty | Variable may not be set on control node; check with `echo $VAR` |
| Pipe command fails | Command runs on control node, not managed host; check dependencies |
| Lookup returns string, need list | Use `query()` instead of `lookup()` |
| Password lookup creates file | Use `/dev/null` as path to avoid file creation |

---

## Summary Table

| Feature | Description |
|---------|-------------|
| `lookup()` | Returns string (comma-separated) |
| `query()` / `q()` | Returns list |
| `wantlist=True` | Force lookup to return list |
| Runs on | Control node (not managed hosts) |
| `file` | Read local file content |
| `env` | Read environment variable |
| `pipe` | Execute command, return stdout |
| `password` | Generate random passwords |
| `first_found` | First existing file from list |
| `url` | HTTP GET request |
| `fileglob` | Find files by glob pattern |

---

## Quick Revision Questions

1. **Where do lookups execute — on the control node or managed hosts?**
   > On the control node. They fetch data locally before sending it to managed hosts.

2. **What is the difference between `lookup()` and `query()`?**
   > `lookup()` returns a comma-separated string. `query()` returns a list. Use `query()` when iterating.

3. **How do you read a file's content into a variable?**
   > `{{ lookup('file', '/path/to/file') }}` — reads the file on the control node.

4. **How do you generate a random password?**
   > `{{ lookup('password', '/dev/null length=20 chars=ascii_letters,digits') }}`

5. **How do you use the first file that exists from a list of candidates?**
   > Use the `first_found` lookup with a list of files and paths.

6. **How do you get the output of a shell command in a variable?**
   > `{{ lookup('pipe', 'command here') }}` — runs on the control node and returns stdout.

---

[← Previous: Vault (Encrypting Secrets)](04-vault.md) | [Next: Filters →](06-filters.md)
