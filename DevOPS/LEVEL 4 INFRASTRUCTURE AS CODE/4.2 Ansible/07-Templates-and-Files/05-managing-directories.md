# Chapter 7.5: Managing Directories

[← Previous: lineinfile and blockinfile](04-lineinfile-and-blockinfile.md) | [Next: Package Modules →](../08-Ansible-Modules/01-package-modules.md)

---

## Overview

Managing directories is fundamental to system automation — creating application directories, setting permissions, cleaning up old files, and synchronising directory trees. This chapter covers directory operations using the `file`, `find`, `synchronize`, and `archive`/`unarchive` modules.

---

## Creating Directories

### Single Directory

```yaml
- name: Create application directory
  file:
    path: /opt/myapp
    state: directory
    owner: appuser
    group: appuser
    mode: '0755'
```

### Nested Directories

```yaml
# Creates all parent directories automatically
- name: Create nested directory structure
  file:
    path: /opt/myapp/releases/v2.1.0/config
    state: directory
    owner: appuser
    mode: '0755'
```

### Multiple Directories with loop

```yaml
- name: Create application directory structure
  file:
    path: "{{ item }}"
    state: directory
    owner: appuser
    group: appuser
    mode: '0755'
  loop:
    - /opt/myapp
    - /opt/myapp/config
    - /opt/myapp/logs
    - /opt/myapp/data
    - /opt/myapp/tmp
    - /var/log/myapp
```

### Directories with Different Permissions

```yaml
- name: Create dirs with specific permissions
  file:
    path: "{{ item.path }}"
    state: directory
    owner: "{{ item.owner | default('root') }}"
    group: "{{ item.group | default('root') }}"
    mode: "{{ item.mode }}"
  loop:
    - { path: /opt/myapp,        mode: '0755' }
    - { path: /opt/myapp/config, mode: '0750' }
    - { path: /opt/myapp/logs,   mode: '0755' }
    - { path: /opt/myapp/data,   mode: '0700', owner: appuser }
    - { path: /opt/myapp/ssl,    mode: '0700', owner: root }
```

---

## Setting Recursive Permissions

```yaml
- name: Fix ownership recursively
  file:
    path: /var/www/html
    owner: www-data
    group: www-data
    recurse: yes

# For more precise control, combine find + file
- name: Find directories
  find:
    paths: /var/www/html
    file_type: directory
  register: found_dirs

- name: Set directory permissions
  file:
    path: "{{ item.path }}"
    mode: '0755'
  loop: "{{ found_dirs.files }}"

- name: Find files
  find:
    paths: /var/www/html
    file_type: file
  register: found_files

- name: Set file permissions
  file:
    path: "{{ item.path }}"
    mode: '0644'
  loop: "{{ found_files.files }}"
```

---

## find Module

Searches for files matching criteria on the remote host.

```yaml
# Find log files older than 7 days
- name: Find old log files
  find:
    paths: /var/log/myapp
    patterns: '*.log'
    age: 7d
    recurse: yes
  register: old_logs

- name: Delete old log files
  file:
    path: "{{ item.path }}"
    state: absent
  loop: "{{ old_logs.files }}"

# Find large files
- name: Find files larger than 100MB
  find:
    paths: /var/log
    size: 100m
    recurse: yes
  register: large_files

# Find by content (grep-like)
- name: Find files containing pattern
  find:
    paths: /etc
    patterns: '*.conf'
    contains: 'password'
    recurse: yes
  register: files_with_passwords
```

### find Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `paths` | Directories to search | `/var/log` |
| `patterns` | Filename glob patterns | `*.log`, `*.tmp` |
| `excludes` | Patterns to exclude | `*.keep` |
| `age` | File age threshold | `7d`, `4w`, `1m` |
| `age_stamp` | Which timestamp | `mtime`, `atime`, `ctime` |
| `size` | File size threshold | `100m`, `1g` |
| `file_type` | `file`, `directory`, `link`, `any` | `file` |
| `recurse` | Search subdirectories | `yes` |
| `contains` | Content regex match | `'ERROR'` |
| `hidden` | Include hidden files | `yes` |

---

## Removing Directories

```yaml
# Remove a directory and all contents
- name: Remove old release
  file:
    path: /opt/myapp/releases/v1.0.0
    state: absent

# Conditional cleanup
- name: Check if temp exists
  stat:
    path: /tmp/build
  register: build_dir

- name: Remove temp build directory
  file:
    path: /tmp/build
    state: absent
  when: build_dir.stat.exists and build_dir.stat.isdir
```

---

## archive and unarchive Modules

### Creating Archives

```yaml
- name: Archive log directory
  archive:
    path: /var/log/myapp/
    dest: /backup/myapp-logs-{{ ansible_date_time.date }}.tar.gz
    format: gz

- name: Archive specific files
  archive:
    path:
      - /etc/nginx/
      - /etc/ssl/certs/
    dest: /backup/nginx-config.tar.gz
    format: gz
    exclude_path:
      - /etc/nginx/sites-available/default
```

### Extracting Archives

```yaml
# Extract from control node
- name: Extract application release
  unarchive:
    src: releases/myapp-2.1.0.tar.gz
    dest: /opt/myapp/releases/
    owner: appuser
    group: appuser

# Extract from URL
- name: Download and extract
  unarchive:
    src: https://example.com/releases/app-2.1.0.tar.gz
    dest: /opt/myapp/
    remote_src: yes

# Extract already on remote host
- name: Extract remote archive
  unarchive:
    src: /tmp/backup.tar.gz
    dest: /opt/restore/
    remote_src: yes
    creates: /opt/restore/data    # Skip if already extracted
```

---

## Real-World Scenario: Application Deployment Directory Structure

```yaml
---
- name: Manage application deployment
  hosts: app_servers
  become: yes
  vars:
    app_name: myapp
    app_user: appuser
    app_version: "2.1.0"
    releases_to_keep: 5

  tasks:
    - name: Create base directory structure
      file:
        path: "{{ item }}"
        state: directory
        owner: "{{ app_user }}"
        mode: '0755'
      loop:
        - "/opt/{{ app_name }}"
        - "/opt/{{ app_name }}/releases"
        - "/opt/{{ app_name }}/shared"
        - "/opt/{{ app_name }}/shared/config"
        - "/opt/{{ app_name }}/shared/log"
        - "/opt/{{ app_name }}/shared/tmp"

    - name: Extract new release
      unarchive:
        src: "releases/{{ app_name }}-{{ app_version }}.tar.gz"
        dest: "/opt/{{ app_name }}/releases/"
        owner: "{{ app_user }}"

    - name: Update current symlink
      file:
        src: "/opt/{{ app_name }}/releases/{{ app_version }}"
        dest: "/opt/{{ app_name }}/current"
        state: link
        owner: "{{ app_user }}"

    - name: Find old releases
      find:
        paths: "/opt/{{ app_name }}/releases"
        file_type: directory
      register: releases

    - name: Remove old releases (keep last N)
      file:
        path: "{{ item.path }}"
        state: absent
      loop: "{{ (releases.files | sort(attribute='mtime') | list)[:-releases_to_keep] }}"
      when: releases.files | length > releases_to_keep
```

```
  DEPLOYMENT DIRECTORY STRUCTURE
  ══════════════════════════════════════════════════════════

  /opt/myapp/
  ├── current -> releases/2.1.0/    ◄── Symlink to active
  ├── releases/
  │   ├── 1.9.0/                    ◄── Old
  │   ├── 2.0.0/                    ◄── Previous
  │   └── 2.1.0/                    ◄── Current
  └── shared/
      ├── config/                   ◄── Persistent config
      ├── log/                      ◄── Persistent logs
      └── tmp/                      ◄── Persistent temp
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| `Permission denied` creating dir | Use `become: yes` or check parent directory permissions |
| `recurse: yes` too slow | Use `find` + `file` for targeted permission changes |
| `unarchive` extracts wrong structure | Check archive structure; use `extra_opts: [--strip-components=1]` |
| `find` returns too many files | Add `patterns`, `age`, or `size` filters |
| Symlink `state: link` fails | Ensure `src` (target) exists first |
| Archive missing files | Check `path` is correct and files are accessible |

---

## Summary Table

| Module | Purpose |
|--------|---------|
| `file` (state: directory) | Create directories |
| `file` (state: absent) | Remove directories |
| `file` (recurse: yes) | Set permissions recursively |
| `find` | Search for files/dirs by criteria |
| `archive` | Create tar/zip archives |
| `unarchive` | Extract archives |
| `synchronize` | rsync-based directory sync |

---

## Quick Revision Questions

1. **How do you create a nested directory structure in Ansible?**
   > Use the `file` module with `state: directory` — Ansible creates all parent directories automatically, like `mkdir -p`.

2. **How do you find and delete files older than 30 days?**
   > Use the `find` module with `age: 30d`, register the result, then loop over it with `file` module `state: absent`.

3. **What is the difference between `recurse: yes` on the `file` module vs using `find`?**
   > `recurse: yes` applies the same permissions to everything. `find` + `file` lets you apply different permissions to files vs directories.

4. **How do you extract an archive from a URL?**
   > Use `unarchive` with `src: https://...` and `remote_src: yes`.

5. **How do you keep only the last N releases in a deployment directory?**
   > Use `find` to list directories, sort by `mtime`, and remove all but the last N using list slicing.

---

[← Previous: lineinfile and blockinfile](04-lineinfile-and-blockinfile.md) | [Next: Package Modules →](../08-Ansible-Modules/01-package-modules.md)
