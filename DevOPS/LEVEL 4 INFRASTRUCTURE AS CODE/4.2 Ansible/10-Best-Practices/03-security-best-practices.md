# Chapter 10.3: Security Best Practices

[← Previous: Playbook Best Practices](02-playbook-best-practices.md) | [Next: Testing Strategies →](04-testing-strategies.md)

---

## Overview

Ansible manages critical infrastructure, so **security must be built in from the start**. This chapter covers secrets management with Vault, SSH hardening, privilege escalation, credential handling, and secure configuration patterns.

---

## Ansible Vault

### Encrypt Secrets

```bash
# Encrypt an entire file
ansible-vault encrypt group_vars/production/vault.yml

# Create a new encrypted file
ansible-vault create group_vars/production/vault.yml

# Edit encrypted file
ansible-vault edit group_vars/production/vault.yml

# View encrypted file
ansible-vault view group_vars/production/vault.yml

# Decrypt (remove encryption)
ansible-vault decrypt group_vars/production/vault.yml

# Re-key (change password)
ansible-vault rekey group_vars/production/vault.yml
```

### Encrypt Single Variables (Inline)

```bash
# Encrypt a single string
ansible-vault encrypt_string 'SuperSecret123!' --name 'vault_db_password'
```

```yaml
# Result — paste into your vars file
vault_db_password: !vault |
  $ANSIBLE_VAULT;1.1;AES256
  61343963626361636135623762306...
  3739653864356437653964613566...
```

### Vault Password Management

```
  VAULT PASSWORD STRATEGIES
  ══════════════════════════════════════════════════════════

  ┌─────────────────────────────────────────────────────┐
  │  1. Password File (basic)                           │
  │     ansible-playbook site.yml                       │
  │       --vault-password-file ~/.vault_pass           │
  │                                                     │
  │  2. Environment Variable                            │
  │     export ANSIBLE_VAULT_PASSWORD_FILE=~/.vault_pass│
  │                                                     │
  │  3. Script (dynamic — e.g., from secrets manager)   │
  │     ansible-playbook site.yml                       │
  │       --vault-password-file get_vault_pass.sh       │
  │                                                     │
  │  4. Multiple Vault IDs (per-environment)            │
  │     ansible-playbook site.yml                       │
  │       --vault-id prod@prompt                        │
  │       --vault-id dev@~/.dev_vault_pass              │
  └─────────────────────────────────────────────────────┘
```

### Vault ID pattern (Multi-Environment)

```bash
# Encrypt with vault ID
ansible-vault encrypt --vault-id prod@prompt vault_prod.yml
ansible-vault encrypt --vault-id dev@prompt vault_dev.yml

# Decrypt both at playbook run
ansible-playbook site.yml \
  --vault-id prod@~/.vault_prod \
  --vault-id dev@~/.vault_dev
```

---

## Secrets Naming Convention

```yaml
# group_vars/production/vault.yml — encrypted
---
vault_db_password: "s3cret_pr0d!"
vault_api_key: "ak-xxxxxxxxxxxx"
vault_ssl_key: |
  -----BEGIN RSA PRIVATE KEY-----
  ...
  -----END RSA PRIVATE KEY-----

# group_vars/production/main.yml — plain text (references vault vars)
---
db_password: "{{ vault_db_password }}"
api_key: "{{ vault_api_key }}"
ssl_key: "{{ vault_ssl_key }}"
```

```
  WHY THE vault_ PREFIX PATTERN?
  ══════════════════════════════════════════════════════════

  vault.yml (encrypted)          main.yml (plain text)
  ┌──────────────────┐           ┌────────────────────────┐
  │ vault_db_pass:   │──────────▶│ db_pass:               │
  │   "s3cret"       │   ref     │   "{{ vault_db_pass }}"│
  └──────────────────┘           └────────────────────────┘

  Benefits:
  • grep for vault_ to find all secrets
  • Plain vars file shows structure without exposing values
  • Easy to audit which values are encrypted
```

---

## no_log — Prevent Credential Leaks

```yaml
- name: Set database password
  user:
    name: dbadmin
    password: "{{ db_password | password_hash('sha512') }}"
  no_log: true        # Hides task output in logs

- name: Call API with token
  uri:
    url: "https://api.example.com/deploy"
    headers:
      Authorization: "Bearer {{ api_token }}"
  no_log: true
```

```
  WITHOUT no_log                  WITH no_log
  ════════════════════            ═══════════════════
  TASK [Set password]            TASK [Set password]
  changed: [db01] => {           changed: [db01] => {
    "password": "s3cret"           "censored":
  }                                "no_log is enabled"
                                 }
```

---

## SSH Security

### Key-Based Authentication

```ini
# ansible.cfg
[defaults]
remote_user = ansible
private_key_file = ~/.ssh/ansible_key

[ssh_connection]
ssh_args = -o PasswordAuthentication=no
```

### SSH Hardening on Managed Nodes

```yaml
- name: Harden SSH configuration
  lineinfile:
    path: /etc/ssh/sshd_config
    regexp: "{{ item.regexp }}"
    line: "{{ item.line }}"
    validate: 'sshd -t -f %s'
  loop:
    - { regexp: '^#?PermitRootLogin', line: 'PermitRootLogin no' }
    - { regexp: '^#?PasswordAuthentication', line: 'PasswordAuthentication no' }
    - { regexp: '^#?X11Forwarding', line: 'X11Forwarding no' }
    - { regexp: '^#?MaxAuthTries', line: 'MaxAuthTries 3' }
    - { regexp: '^#?AllowAgentForwarding', line: 'AllowAgentForwarding no' }
  notify: Restart SSH

- name: Limit SSH to ansible group
  lineinfile:
    path: /etc/ssh/sshd_config
    line: "AllowGroups ansible sudo"
    validate: 'sshd -t -f %s'
  notify: Restart SSH
```

---

## Privilege Escalation Security

```yaml
# Limit sudo access
- name: Configure sudoers for Ansible user
  copy:
    content: |
      # Ansible automation user
      ansible ALL=(ALL) NOPASSWD: /usr/bin/apt-get, /usr/bin/systemctl
      ansible ALL=(ALL) NOPASSWD: /usr/sbin/service
    dest: /etc/sudoers.d/ansible
    validate: 'visudo -cf %s'
    mode: '0440'
```

```
  PRINCIPLE OF LEAST PRIVILEGE
  ══════════════════════════════════════════════════════════

  BAD:   ansible ALL=(ALL) NOPASSWD: ALL

  GOOD:  ansible ALL=(ALL) NOPASSWD: /usr/bin/apt-get
         ansible ALL=(ALL) NOPASSWD: /usr/bin/systemctl
         ansible ALL=(ALL) NOPASSWD: /usr/sbin/service
```

---

## File Permissions

```yaml
# Always set explicit permissions
- name: Deploy SSL certificate
  copy:
    src: server.crt
    dest: /etc/ssl/certs/server.crt
    owner: root
    group: root
    mode: '0644'

- name: Deploy SSL private key
  copy:
    content: "{{ vault_ssl_private_key }}"
    dest: /etc/ssl/private/server.key
    owner: root
    group: ssl-cert
    mode: '0640'        # Restrictive for private keys
```

### Common Permission Values

| File Type | Mode | Meaning |
|-----------|------|---------|
| Public config | `0644` | Owner rw, group/other read |
| Private key | `0600` | Owner rw only |
| Script | `0755` | Owner rwx, group/other rx |
| Secrets dir | `0700` | Owner only |
| sudoers file | `0440` | Owner/group read only |

---

## Securing Ansible Controller

```
  CONTROLLER SECURITY CHECKLIST
  ══════════════════════════════════════════════════════════

  [✓] Dedicated service account (not root)
  [✓] SSH keys with passphrase (use ssh-agent)
  [✓] Vault password file permissions: 0600
  [✓] ansible.cfg not world-readable
  [✓] Log files secured (0600)
  [✓] Git repo access controlled
  [✓] No secrets in version control (unencrypted)
  [✓] .gitignore excludes vault passwords, logs, retries
```

```gitignore
# .gitignore — security essentials
*.retry
*.log
.vault_pass*
vault_password*
*.pem
*.key
id_rsa*
```

---

## External Secrets Managers

```yaml
# HashiCorp Vault lookup
- name: Get secret from HashiCorp Vault
  set_fact:
    db_password: "{{ lookup('hashi_vault', 'secret/data/db:password') }}"

# AWS Secrets Manager
- name: Get secret from AWS
  set_fact:
    api_key: "{{ lookup('amazon.aws.aws_secret', 'myapp/api_key') }}"

# Azure Key Vault
- name: Get secret from Azure
  set_fact:
    cert: "{{ lookup('azure.azcollection.azure_keyvault_secret',
              'my-cert', vault_url='https://myvault.vault.azure.net') }}"
```

```
  EXTERNAL SECRETS FLOW
  ══════════════════════════════════════════════════════════

  ┌──────────────┐    Lookup     ┌─────────────────────┐
  │  Playbook    │──────────────▶│  Secrets Manager    │
  │              │               │  (Vault/AWS/Azure)  │
  │  {{ lookup  │  secret value │                     │
  │     (...) }}│◀──────────────│  Authenticated via  │
  │              │               │  token / IAM role   │
  └──────────────┘               └─────────────────────┘

  Secrets never stored in Ansible files!
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Vault password prompt every run | Use `--vault-password-file` or `ANSIBLE_VAULT_PASSWORD_FILE` |
| Secret visible in output | Add `no_log: true` to the task |
| Forgot vault password | No recovery — re-encrypt with new password |
| Vault file shows as binary in git | Normal — encrypted files are binary; use `ansible-vault view` |
| Permission denied on managed node | Check SSH key, user exists, sudoers config |
| `!vault` tag not recognised | Ensure Ansible >= 2.4 and file is valid YAML |

---

## Summary Table

| Practice | Implementation |
|----------|---------------|
| Encrypt secrets | `ansible-vault encrypt` files or inline strings |
| Vault naming | `vault_` prefix in encrypted file, reference in plain file |
| Hide output | `no_log: true` on tasks with credentials |
| SSH security | Key-based auth, disable root login, limit groups |
| File permissions | Explicit `mode:` on every file with sensitive content |
| Least privilege | Restrict sudo to specific commands |
| .gitignore | Exclude vault passwords, keys, logs, retries |
| External secrets | Lookup plugins for HashiCorp Vault, AWS, Azure |
| Controller security | Dedicated user, secured vault pass, log protection |

---

## Quick Revision Questions

1. **How do you encrypt a single variable instead of an entire file?**
   > Use `ansible-vault encrypt_string 'value' --name 'var_name'` and paste the result into your YAML file.

2. **What is the `vault_` prefix convention?**
   > Store secrets in encrypted files with `vault_` prefixed names, then reference them from plain-text files (e.g., `db_pass: "{{ vault_db_pass }}"`) for easy auditing.

3. **What does `no_log: true` do?**
   > It prevents Ansible from displaying the task's input parameters and return values in the console output and logs, protecting credentials.

4. **Why should you restrict sudo commands for the Ansible user?**
   > Following the principle of least privilege minimises the blast radius if the Ansible service account is compromised.

5. **How can you use an external secrets manager with Ansible?**
   > Use lookup plugins (e.g., `hashi_vault`, `aws_secret`, `azure_keyvault_secret`) to fetch secrets at runtime without storing them in Ansible files.

---

[← Previous: Playbook Best Practices](02-playbook-best-practices.md) | [Next: Testing Strategies →](04-testing-strategies.md)
