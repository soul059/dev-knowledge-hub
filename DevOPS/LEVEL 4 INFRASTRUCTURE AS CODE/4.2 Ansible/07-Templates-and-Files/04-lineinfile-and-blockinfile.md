# Chapter 7.4: lineinfile and blockinfile

[← Previous: File Modules](03-file-modules.md) | [Next: Managing Directories →](05-managing-directories.md)

---

## Overview

**lineinfile** manages individual lines in a file, while **blockinfile** manages multi-line blocks. Both are ideal for making surgical edits to existing configuration files without managing the entire file via a template.

---

## When to Use Which

```
  DECISION FLOW
  ══════════════════════════════════════════════════════════

  "Do I manage the WHOLE file?"
         │
    YES ─┤─── Use template module
         │
    NO ──┤─── "Single line change?"
              │
         YES ┤─── lineinfile
              │
         NO ──┤─── "Multi-line block?"
               │
          YES ─┘─── blockinfile
```

---

## lineinfile Module

### Ensure a Line Exists

```yaml
- name: Enable IP forwarding
  lineinfile:
    path: /etc/sysctl.conf
    line: "net.ipv4.ip_forward = 1"
    state: present
```

### Find and Replace a Line

```yaml
- name: Set SSH port
  lineinfile:
    path: /etc/ssh/sshd_config
    regexp: '^#?Port\s+'
    line: "Port 2222"
  notify: restart sshd

- name: Disable root login
  lineinfile:
    path: /etc/ssh/sshd_config
    regexp: '^#?PermitRootLogin\s+'
    line: "PermitRootLogin no"
  notify: restart sshd
```

### Insert After / Before a Pattern

```yaml
# Insert after a matching line
- name: Add nameserver after search line
  lineinfile:
    path: /etc/resolv.conf
    insertafter: '^search'
    line: "nameserver 8.8.8.8"

# Insert before a matching line
- name: Add comment before block
  lineinfile:
    path: /etc/fstab
    insertbefore: '^/dev/sdb'
    line: "# Data partition below"
```

### Special Positions

```yaml
# Insert at end of file (BOF for beginning)
- name: Add line at end
  lineinfile:
    path: /etc/environment
    insertafter: EOF
    line: 'JAVA_HOME=/usr/lib/jvm/java-17'

# Insert at beginning
- name: Add shebang
  lineinfile:
    path: /opt/script.sh
    insertbefore: BOF
    line: '#!/bin/bash'
```

### Remove a Line

```yaml
- name: Remove deprecated setting
  lineinfile:
    path: /etc/myapp.conf
    regexp: '^old_setting='
    state: absent
```

### Create File if Missing

```yaml
- name: Ensure line in file (create if needed)
  lineinfile:
    path: /etc/myapp/custom.conf
    line: "option = value"
    create: yes
    owner: root
    mode: '0644'
```

### Validate Before Saving

```yaml
- name: Edit sudoers safely
  lineinfile:
    path: /etc/sudoers
    regexp: '^%wheel'
    line: '%wheel ALL=(ALL) NOPASSWD: ALL'
    validate: 'visudo -cf %s'
```

---

## lineinfile Key Parameters

| Parameter | Description |
|-----------|-------------|
| `path` | File to modify |
| `line` | The line content to ensure |
| `regexp` | Regex to find existing line to replace |
| `state` | `present` (default) or `absent` |
| `insertafter` | Add after matching regex (or `EOF`) |
| `insertbefore` | Add before matching regex (or `BOF`) |
| `create` | Create file if it doesn't exist |
| `backup` | Create backup before modifying |
| `validate` | Validate command (`%s` = temp file) |
| `firstmatch` | Replace only first match |

---

## lineinfile Behaviour Matrix

```
  HOW lineinfile DECIDES WHAT TO DO
  ══════════════════════════════════════════════════════════

  regexp provided?   line matches?   Action
  ────────────────   ─────────────   ──────────────────────
  Yes                Found           Replace matched line
  Yes                Not found       Add line (insertafter)
  No                 Found           No change (idempotent)
  No                 Not found       Append line to file
```

---

## blockinfile Module

### Add a Block

```yaml
- name: Add proxy settings
  blockinfile:
    path: /etc/environment
    block: |
      http_proxy=http://proxy.example.com:3128
      https_proxy=http://proxy.example.com:3128
      no_proxy=localhost,127.0.0.1,.example.com
```

Output in file:

```
# BEGIN ANSIBLE MANAGED BLOCK
http_proxy=http://proxy.example.com:3128
https_proxy=http://proxy.example.com:3128
no_proxy=localhost,127.0.0.1,.example.com
# END ANSIBLE MANAGED BLOCK
```

### Custom Markers

```yaml
- name: Add iptables rules
  blockinfile:
    path: /etc/iptables/rules.v4
    marker: "# {mark} ANSIBLE MANAGED - web rules"
    block: |
      -A INPUT -p tcp --dport 80 -j ACCEPT
      -A INPUT -p tcp --dport 443 -j ACCEPT
```

Output:

```
# BEGIN ANSIBLE MANAGED - web rules
-A INPUT -p tcp --dport 80 -j ACCEPT
-A INPUT -p tcp --dport 443 -j ACCEPT
# END ANSIBLE MANAGED - web rules
```

### Multiple Independent Blocks

```yaml
# Using custom markers lets you manage multiple blocks in one file
- name: Add database rules
  blockinfile:
    path: /etc/iptables/rules.v4
    marker: "# {mark} ANSIBLE MANAGED - db rules"
    block: |
      -A INPUT -p tcp --dport 5432 -s 10.0.1.0/24 -j ACCEPT

- name: Add monitoring rules
  blockinfile:
    path: /etc/iptables/rules.v4
    marker: "# {mark} ANSIBLE MANAGED - monitoring rules"
    block: |
      -A INPUT -p tcp --dport 9090 -s 10.0.2.0/24 -j ACCEPT
```

### Insert Position

```yaml
- name: Add block after a pattern
  blockinfile:
    path: /etc/nginx/nginx.conf
    insertafter: "http {"
    marker: "# {mark} ANSIBLE MANAGED - custom headers"
    block: |
      add_header X-Frame-Options SAMEORIGIN;
      add_header X-Content-Type-Options nosniff;
      add_header X-XSS-Protection "1; mode=block";
```

### Remove a Block

```yaml
- name: Remove managed block
  blockinfile:
    path: /etc/environment
    marker: "# {mark} ANSIBLE MANAGED BLOCK"
    block: ""
    state: present
    # Or simply:
    # state: absent
```

---

## blockinfile Key Parameters

| Parameter | Description |
|-----------|-------------|
| `path` | File to modify |
| `block` | Multi-line content to insert |
| `marker` | Marker template (`{mark}` → BEGIN/END) |
| `insertafter` | Insert after matching regex |
| `insertbefore` | Insert before matching regex |
| `create` | Create file if missing |
| `backup` | Create backup before modifying |
| `state` | `present` or `absent` |
| `marker_begin` | Custom BEGIN text (default: `BEGIN`) |
| `marker_end` | Custom END text (default: `END`) |

---

## Real-World Scenario: SSH Hardening

```yaml
---
- name: Harden SSH configuration
  hosts: all
  become: yes
  tasks:
    - name: Set SSH port
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?Port\s+'
        line: "Port {{ ssh_port | default(22) }}"
      notify: restart sshd

    - name: Disable root login
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?PermitRootLogin'
        line: "PermitRootLogin no"
      notify: restart sshd

    - name: Disable password auth
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?PasswordAuthentication'
        line: "PasswordAuthentication no"
      notify: restart sshd

    - name: Add AllowUsers block
      blockinfile:
        path: /etc/ssh/sshd_config
        marker: "# {mark} ANSIBLE MANAGED - allowed users"
        block: |
          AllowUsers {{ ssh_allowed_users | join(' ') }}
          MaxAuthTries 3
          LoginGraceTime 30
        validate: sshd -t -f %s
      notify: restart sshd

  handlers:
    - name: restart sshd
      service:
        name: sshd
        state: restarted
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Line added multiple times | Use `regexp` to match existing line before replacing |
| `blockinfile` overwrites previous block | Use unique custom `marker` for each block |
| Line inserted in wrong position | Add `insertafter`/`insertbefore` with correct regex |
| `regexp` not matching | Test regex separately; remember `^` and `$` anchors |
| File not created | Set `create: yes` |
| Changes break service config | Use `validate` to test before applying |

---

## Summary Table

| Feature | `lineinfile` | `blockinfile` |
|---------|:------------:|:-------------:|
| Single line | ✓ | — |
| Multi-line block | — | ✓ |
| Regex matching | ✓ | — |
| Markers | — | ✓ |
| Insert position | ✓ | ✓ |
| Validate | ✓ | — |
| Backup | ✓ | ✓ |
| Create file | ✓ | ✓ |

---

## Quick Revision Questions

1. **When should you use `lineinfile` vs `template`?**
   > Use `lineinfile` for small, surgical edits to existing files. Use `template` when you manage the entire file content.

2. **What does the `regexp` parameter do in `lineinfile`?**
   > It searches for an existing line matching the regex. If found, that line is replaced with the `line` value. If not found, the line is added.

3. **How does `blockinfile` identify its managed content?**
   > Using marker comments (default: `# BEGIN ANSIBLE MANAGED BLOCK` and `# END ANSIBLE MANAGED BLOCK`).

4. **How do you manage multiple independent blocks in the same file?**
   > Use a unique `marker` for each `blockinfile` task, e.g., `marker: "# {mark} ANSIBLE - web rules"`.

5. **What does `{mark}` in the marker parameter get replaced with?**
   > `{mark}` is replaced with `BEGIN` for the start marker and `END` for the end marker.

---

[← Previous: File Modules](03-file-modules.md) | [Next: Managing Directories →](05-managing-directories.md)
