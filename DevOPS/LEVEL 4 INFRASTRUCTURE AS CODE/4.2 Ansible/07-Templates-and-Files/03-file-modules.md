# Chapter 7.3: File Modules

[← Previous: Template Module](02-template-module.md) | [Next: lineinfile and blockinfile →](04-lineinfile-and-blockinfile.md)

---

## Overview

Ansible provides several modules for managing files on remote hosts: **copy**, **file**, **fetch**, **stat**, **synchronize**, and **assemble**. These modules handle transferring files, setting permissions, creating symlinks, and gathering file information.

---

## copy Module

Copies files from the control node to managed hosts, or places inline content.

```yaml
# Copy a file
- name: Copy config file
  copy:
    src: app.conf
    dest: /etc/myapp/app.conf
    owner: root
    group: root
    mode: '0644'

# Copy with inline content
- name: Create file with content
  copy:
    content: |
      # Application configuration
      debug = false
      port = 8080
    dest: /etc/myapp/settings.conf
    mode: '0644'

# Copy a directory recursively
- name: Copy web files
  copy:
    src: website/          # Trailing / copies contents only
    dest: /var/www/html/
    owner: www-data
    mode: '0755'

# Copy with backup
- name: Copy with backup
  copy:
    src: updated.conf
    dest: /etc/app.conf
    backup: yes

# Remote-to-remote copy
- name: Copy file on remote host
  copy:
    src: /tmp/source.txt
    dest: /opt/destination.txt
    remote_src: yes
```

```
  COPY: src TRAILING SLASH MATTERS
  ══════════════════════════════════════════════════════════

  src: files/website       →  /var/www/html/website/...
  src: files/website/      →  /var/www/html/...
                      ^
                  Trailing / = copy CONTENTS, not the dir
```

### Key Parameters

| Parameter | Description |
|-----------|-------------|
| `src` | Source file/dir on control node |
| `dest` | Destination on remote host |
| `content` | Inline content (instead of src) |
| `mode` | File permissions |
| `owner`/`group` | Ownership |
| `backup` | Backup existing file |
| `remote_src` | Source is on the remote host |
| `force` | Replace if different (default: yes) |
| `validate` | Validate before placing |

---

## file Module

Manages file properties: create, delete, set permissions, create links.

```yaml
# Create a directory
- name: Create app directory
  file:
    path: /opt/myapp
    state: directory
    owner: appuser
    group: appuser
    mode: '0755'

# Create nested directories
- name: Create nested dirs
  file:
    path: /opt/myapp/logs/archive
    state: directory
    recurse: yes

# Create a symlink
- name: Create symlink
  file:
    src: /opt/myapp/current/config.conf
    dest: /etc/myapp.conf
    state: link

# Create a hard link
- name: Create hard link
  file:
    src: /var/log/app.log
    dest: /opt/logs/app.log
    state: hard

# Set permissions
- name: Set permissions
  file:
    path: /opt/myapp/run.sh
    mode: '0755'

# Set permissions recursively
- name: Fix directory permissions
  file:
    path: /var/www/html
    owner: www-data
    group: www-data
    recurse: yes

# Delete a file
- name: Remove old config
  file:
    path: /etc/old-app.conf
    state: absent

# Delete a directory
- name: Remove temp directory
  file:
    path: /tmp/build
    state: absent       # Removes directory and all contents

# Touch a file (create if not exists, update timestamp)
- name: Touch file
  file:
    path: /tmp/marker
    state: touch
    mode: '0644'
```

### State Options

| State | Description |
|-------|-------------|
| `file` | File must exist; set attributes only |
| `directory` | Create directory |
| `link` | Create symbolic link |
| `hard` | Create hard link |
| `touch` | Create empty file or update timestamp |
| `absent` | Delete file or directory |

---

## fetch Module

Copies files **from** managed hosts **to** the control node (opposite of copy).

```yaml
- name: Fetch log files
  fetch:
    src: /var/log/syslog
    dest: /tmp/fetched/
    flat: no      # Default: creates host-specific subdirectory

# Result: /tmp/fetched/web01/var/log/syslog
```

```yaml
- name: Fetch with flat path
  fetch:
    src: /etc/hostname
    dest: "/tmp/hostnames/{{ inventory_hostname }}-hostname"
    flat: yes     # No subdirectory structure

# Result: /tmp/hostnames/web01-hostname
```

```
  FETCH DIRECTORY STRUCTURE
  ══════════════════════════════════════════════════════════

  flat: no (default)
  /tmp/fetched/
  ├── web01/
  │   └── var/log/syslog
  └── web02/
      └── var/log/syslog

  flat: yes
  /tmp/fetched/
  ├── web01-syslog
  └── web02-syslog
```

---

## stat Module

Gathers information about a file without modifying it.

```yaml
- name: Check if config exists
  stat:
    path: /etc/myapp.conf
  register: config_stat

- name: Deploy if missing
  template:
    src: myapp.conf.j2
    dest: /etc/myapp.conf
  when: not config_stat.stat.exists

- name: Show file info
  debug:
    msg: |
      Exists: {{ config_stat.stat.exists }}
      Size: {{ config_stat.stat.size }}
      UID: {{ config_stat.stat.uid }}
      Mode: {{ config_stat.stat.mode }}
      Is Dir: {{ config_stat.stat.isdir }}
      Is Link: {{ config_stat.stat.islnk }}
      Checksum: {{ config_stat.stat.checksum }}
```

### Useful stat Attributes

| Attribute | Description |
|-----------|-------------|
| `stat.exists` | Whether file exists |
| `stat.isdir` | Is a directory |
| `stat.isreg` | Is a regular file |
| `stat.islnk` | Is a symlink |
| `stat.size` | File size in bytes |
| `stat.mode` | Permission mode |
| `stat.uid`/`stat.gid` | Owner/group IDs |
| `stat.checksum` | SHA1 checksum |
| `stat.mtime` | Modification timestamp |

---

## synchronize Module

Wrapper around **rsync** for efficient file synchronisation.

```yaml
- name: Sync application files
  synchronize:
    src: /opt/release/
    dest: /var/www/app/
    delete: yes              # Remove extra files on dest
    recursive: yes

- name: Sync with exclusions
  synchronize:
    src: /opt/app/
    dest: /var/www/app/
    rsync_opts:
      - "--exclude=*.log"
      - "--exclude=.git"
      - "--exclude=node_modules"

# Pull mode (remote → control node)
- name: Pull files from remote
  synchronize:
    src: /var/log/
    dest: /backup/logs/{{ inventory_hostname }}/
    mode: pull
```

---

## assemble Module

Concatenates multiple file fragments into a single file.

```yaml
- name: Assemble SSH config
  assemble:
    src: /etc/ssh/sshd_config.d/    # Directory of fragments
    dest: /etc/ssh/sshd_config
    owner: root
    mode: '0600'
    validate: sshd -t -f %s
  notify: restart sshd
```

```
  ASSEMBLE FLOW
  ══════════════════════════════════════════════════════════

  /etc/ssh/sshd_config.d/
  ├── 00-base.conf        ──┐
  ├── 10-security.conf       ├──▶  /etc/ssh/sshd_config
  └── 20-access.conf       ──┘    (concatenated in order)
```

---

## Real-World Scenario: Application Deployment

```yaml
---
- name: Deploy application
  hosts: app_servers
  become: yes
  tasks:
    - name: Create app directory
      file:
        path: "{{ item }}"
        state: directory
        owner: appuser
        mode: '0755'
      loop:
        - /opt/myapp
        - /opt/myapp/config
        - /opt/myapp/logs
        - /opt/myapp/data

    - name: Deploy application binary
      copy:
        src: releases/myapp-{{ version }}
        dest: /opt/myapp/myapp
        owner: appuser
        mode: '0755'
      notify: restart myapp

    - name: Deploy configuration
      template:
        src: app.conf.j2
        dest: /opt/myapp/config/app.conf
        owner: appuser
        mode: '0640'
        validate: /opt/myapp/myapp --validate-config %s
      notify: restart myapp

    - name: Create current version symlink
      file:
        src: /opt/myapp
        dest: /opt/current-app
        state: link

    - name: Fetch deployment receipt
      fetch:
        src: /opt/myapp/config/app.conf
        dest: "receipts/{{ inventory_hostname }}/"
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| `Permission denied` on copy | Use `become: yes`, check dest directory permissions |
| Symlink pointing to wrong target | Verify `src` (target) and `dest` (link name) aren't swapped |
| `stat` shows `exists: false` | Path may be a symlink to missing file — check `islnk` |
| `synchronize` fails | Install rsync on both control node and remote host |
| `copy` is slow for large files | Use `synchronize` (rsync) instead |
| Trailing slash confusion with `copy` | `src: dir/` copies contents; `src: dir` copies the directory itself |

---

## Summary Table

| Module | Direction | Purpose |
|--------|-----------|---------|
| `copy` | Control → Remote | Transfer files/content to hosts |
| `template` | Control → Remote | Render Jinja2 then transfer |
| `file` | Remote | Manage permissions, dirs, links |
| `fetch` | Remote → Control | Pull files from hosts |
| `stat` | Remote | Get file metadata |
| `synchronize` | Both | rsync-based efficient sync |
| `assemble` | Remote | Concatenate fragments → single file |

---

## Quick Revision Questions

1. **What is the difference between `copy` and `template`?**
   > `copy` transfers files as-is (or inline content). `template` renders Jinja2 variables/logic before transferring.

2. **What does a trailing slash on `copy src` mean?**
   > `src: dir/` copies the **contents** of the directory. Without the slash, it copies the directory itself.

3. **How does `fetch` differ from `copy`?**
   > `fetch` pulls files **from** managed hosts **to** the control node. `copy` pushes **from** the control node **to** managed hosts.

4. **What module would you use to check if a file exists before acting?**
   > The `stat` module — register the result and check `result.stat.exists`.

5. **How does the `assemble` module work?**
   > It reads all files from a source directory, sorts them alphabetically, and concatenates them into a single destination file.

---

[← Previous: Template Module](02-template-module.md) | [Next: lineinfile and blockinfile →](04-lineinfile-and-blockinfile.md)
