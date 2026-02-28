# Chapter 8.4: User and Group Modules

[← Previous: Command Modules](03-command-modules.md) | [Next: Cloud Modules →](05-cloud-modules.md)

---

## Overview

Managing users and groups is a core task in system administration. Ansible provides the **user**, **group**, and **authorized_key** modules to create accounts, manage group membership, set passwords, and deploy SSH keys.

---

## group Module

```yaml
# Create a group
- name: Create application group
  group:
    name: appgroup
    state: present

# Create with specific GID
- name: Create group with GID
  group:
    name: appgroup
    gid: 1500
    state: present

# System group
- name: Create system group
  group:
    name: myservice
    system: yes

# Remove a group
- name: Remove old group
  group:
    name: oldgroup
    state: absent
```

### group Parameters

| Parameter | Description |
|-----------|-------------|
| `name` | Group name |
| `gid` | Group ID |
| `state` | `present` or `absent` |
| `system` | Create as system group |

---

## user Module

### Create Users

```yaml
# Create a basic user
- name: Create application user
  user:
    name: appuser
    comment: "Application Service Account"
    state: present

# Create with full details
- name: Create deploy user
  user:
    name: deploy
    comment: "Deployment User"
    uid: 1500
    group: deploy
    groups:
      - sudo
      - docker
    shell: /bin/bash
    home: /home/deploy
    create_home: yes

# System user (no home, no login)
- name: Create service account
  user:
    name: myservice
    system: yes
    shell: /usr/sbin/nologin
    create_home: no
    comment: "My Service Account"

# Set password
- name: Create user with password
  user:
    name: admin
    password: "{{ 'SecurePass123!' | password_hash('sha512') }}"
    update_password: on_create    # Only set on first creation

# Generate SSH key
- name: Create user with SSH key
  user:
    name: deploy
    generate_ssh_key: yes
    ssh_key_bits: 4096
    ssh_key_comment: "deploy@{{ ansible_hostname }}"
    ssh_key_file: .ssh/id_rsa
```

### Modify Users

```yaml
# Add user to additional groups
- name: Add user to docker group
  user:
    name: deploy
    groups: docker
    append: yes            # IMPORTANT: append, don't replace

# Change shell
- name: Set bash shell
  user:
    name: admin
    shell: /bin/bash

# Lock user account
- name: Lock user
  user:
    name: oldemployee
    password_lock: yes

# Set password expiry
- name: Set password expiry
  user:
    name: contractor
    expires: 1735689600    # Unix timestamp
```

### Remove Users

```yaml
# Remove user
- name: Remove user
  user:
    name: olduser
    state: absent

# Remove user and home directory
- name: Remove user and files
  user:
    name: olduser
    state: absent
    remove: yes            # Removes home directory
    force: yes             # Remove even if logged in
```

### user Key Parameters

| Parameter | Description |
|-----------|-------------|
| `name` | Username |
| `uid` | User ID |
| `group` | Primary group |
| `groups` | Supplementary groups |
| `append` | Append to groups vs replace |
| `shell` | Login shell |
| `home` | Home directory path |
| `create_home` | Create home directory |
| `password` | Hashed password |
| `update_password` | `always` or `on_create` |
| `system` | System account |
| `state` | `present` or `absent` |
| `remove` | Remove home when absent |
| `generate_ssh_key` | Generate SSH keypair |
| `password_lock` | Lock the account |
| `expires` | Account expiry (unix timestamp) |

---

## authorized_key Module

```yaml
# Deploy SSH public key
- name: Add SSH key for deploy user
  authorized_key:
    user: deploy
    key: "ssh-rsa AAAAB3NzaC1yc2EAAAA... deploy@workstation"
    state: present

# From a file
- name: Add SSH key from file
  authorized_key:
    user: deploy
    key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"

# Multiple keys
- name: Add multiple SSH keys
  authorized_key:
    user: deploy
    key: "{{ item }}"
  loop:
    - "{{ lookup('file', 'keys/admin1.pub') }}"
    - "{{ lookup('file', 'keys/admin2.pub') }}"

# Exclusive — remove all other keys
- name: Set exact keys (remove unlisted)
  authorized_key:
    user: deploy
    key: "{{ lookup('file', 'keys/authorized_keys') }}"
    exclusive: yes

# With key options
- name: Add restricted key
  authorized_key:
    user: backup
    key: "ssh-rsa AAAAB3Nza... backup@server"
    key_options: 'command="/usr/local/bin/backup.sh",no-port-forwarding,no-X11-forwarding'

# Remove a key
- name: Remove old SSH key
  authorized_key:
    user: deploy
    key: "ssh-rsa AAAAB3NzaOLD... old@workstation"
    state: absent
```

---

## Password Handling

```
  PASSWORD HASHING FLOW
  ══════════════════════════════════════════════════════════

  Plain password → password_hash('sha512') → Hashed string
                                              → user module
                                                (password param)

  NEVER store plain-text passwords in playbooks!
  Use Ansible Vault to encrypt variables.
```

```yaml
# Hash with password_hash filter
- name: Create user with password
  user:
    name: admin
    password: "{{ user_password | password_hash('sha512') }}"
  no_log: true    # Don't show password in output

# Vault-encrypted variable
# vars/vault.yml (encrypted)
# admin_password: SecurePass123!

- name: Create user
  user:
    name: admin
    password: "{{ admin_password | password_hash('sha512') }}"
    update_password: on_create
  no_log: true
```

---

## Real-World Scenario: Team User Management

```yaml
---
- name: Manage system users
  hosts: all
  become: yes
  vars:
    user_groups:
      - name: developers
        gid: 2000
      - name: operators
        gid: 2001

    users:
      - name: alice
        comment: "Alice Smith"
        groups: [developers, sudo]
        ssh_key: "ssh-rsa AAAAB3... alice@laptop"
        state: present
      - name: bob
        comment: "Bob Jones"
        groups: [developers]
        ssh_key: "ssh-rsa AAAAB3... bob@laptop"
        state: present
      - name: charlie
        comment: "Charlie Brown"
        groups: [operators, sudo]
        ssh_key: "ssh-rsa AAAAB3... charlie@laptop"
        state: present
      - name: dave
        comment: "Dave Wilson (left company)"
        state: absent
        remove: yes

  tasks:
    - name: Create groups
      group:
        name: "{{ item.name }}"
        gid: "{{ item.gid }}"
        state: present
      loop: "{{ user_groups }}"

    - name: Create active users
      user:
        name: "{{ item.name }}"
        comment: "{{ item.comment }}"
        groups: "{{ item.groups | default([]) }}"
        append: yes
        shell: /bin/bash
        create_home: yes
        state: present
      loop: "{{ users | selectattr('state', 'equalto', 'present') }}"

    - name: Deploy SSH keys
      authorized_key:
        user: "{{ item.name }}"
        key: "{{ item.ssh_key }}"
        exclusive: yes
      loop: "{{ users | selectattr('state', 'equalto', 'present') }}"
      when: item.ssh_key is defined

    - name: Remove departed users
      user:
        name: "{{ item.name }}"
        state: absent
        remove: "{{ item.remove | default(false) }}"
      loop: "{{ users | selectattr('state', 'equalto', 'absent') }}"
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Password not working | Ensure using `password_hash('sha512')`; plain text won't work |
| Groups replaced instead of added | Use `append: yes` with `groups` |
| User can't log in | Check `shell` setting; `/usr/sbin/nologin` prevents login |
| SSH key not accepted | Verify key format and file permissions (`~/.ssh` = 700, `authorized_keys` = 600) |
| Password shows in logs | Add `no_log: true` to the task |
| `expires` wrong | Use Unix timestamp, not date string |

---

## Summary Table

| Module | Purpose |
|--------|---------|
| `group` | Create, remove, manage groups |
| `user` | Create, remove, manage user accounts |
| `authorized_key` | Manage SSH authorized keys |
| `password_hash` | Filter to hash passwords for user module |

| Parameter | Note |
|-----------|------|
| `append: yes` | Add to groups without removing existing |
| `update_password: on_create` | Set password only on first creation |
| `exclusive: yes` | Remove all keys not in the provided list |
| `system: yes` | Create as system account |
| `no_log: true` | Hide sensitive output |

---

## Quick Revision Questions

1. **Why do you need `password_hash('sha512')` when setting passwords?**
   > The `user` module requires pre-hashed passwords. Providing plain text results in an unusable account.

2. **What does `append: yes` do when setting groups?**
   > It adds the user to the listed groups without removing them from any existing groups.

3. **How do you ensure only specific SSH keys are authorized for a user?**
   > Use `authorized_key` with `exclusive: yes` — this removes any keys not in the provided list.

4. **What is `update_password: on_create`?**
   > It sets the password only when the user is first created. Subsequent runs won't change an existing password.

5. **How do you create a service account that cannot log in interactively?**
   > Set `system: yes`, `shell: /usr/sbin/nologin`, and `create_home: no`.

---

[← Previous: Command Modules](03-command-modules.md) | [Next: Cloud Modules →](05-cloud-modules.md)
