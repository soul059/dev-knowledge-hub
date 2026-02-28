# Chapter 5.4: Vault (Encrypting Secrets)

[← Previous: Magic Variables](03-magic-variables.md) | [Next: Lookups →](05-lookups.md)

---

## Overview

**Ansible Vault** encrypts sensitive data — passwords, API keys, certificates — so they can be safely stored in version control alongside your playbooks. Vault uses AES-256 encryption and can encrypt entire files or individual variables.

---

## Vault Workflow

```
  VAULT WORKFLOW
  ══════════════════════════════════════════════════════════

  ┌──────────────┐     ansible-vault     ┌──────────────┐
  │  secrets.yml │ ──── encrypt ────────► │  $ANSIBLE_   │
  │  (plaintext) │                        │  VAULT;1.2;  │
  │              │ ◄─── decrypt ──────── │  AES256      │
  │  password:   │                        │  3836653032  │
  │    mysecret  │     ansible-vault     │  6261376135  │
  └──────────────┘                        └──────────────┘
                                               │
                                               │ stored in Git
                                               ▼
                                          ┌──────────────┐
                                          │  Repository  │
                                          │  (safe!)     │
                                          └──────────────┘
                                               │
                  ansible-playbook              │
                  --ask-vault-pass              │
                                               ▼
                                          ┌──────────────┐
                                          │  Decrypted   │
                                          │  at runtime  │
                                          │  (in memory) │
                                          └──────────────┘
```

---

## Encrypting Files

### Create New Encrypted File

```bash
# Interactive — prompts for vault password
ansible-vault create secrets.yml

# Opens your $EDITOR with empty file
# Save and close → file is encrypted
```

### Encrypt Existing File

```bash
# Encrypt a plaintext file
ansible-vault encrypt vars/secrets.yml

# Encrypt multiple files
ansible-vault encrypt vars/prod-secrets.yml vars/staging-secrets.yml
```

### View Encrypted File

```bash
# View without decrypting to disk
ansible-vault view vars/secrets.yml
```

### Edit Encrypted File

```bash
# Decrypt in memory, open in editor, re-encrypt on save
ansible-vault edit vars/secrets.yml
```

### Decrypt File

```bash
# Decrypt permanently (remove encryption)
ansible-vault decrypt vars/secrets.yml

# Decrypt to different file
ansible-vault decrypt vars/secrets.yml --output vars/secrets-plain.yml
```

### Rekey (Change Password)

```bash
# Change the vault password
ansible-vault rekey vars/secrets.yml
```

---

## Encrypted File Format

```yaml
# Before encryption
---
db_password: supersecret123
api_key: ak_live_abc123def456
ssl_private_key: |
  -----BEGIN PRIVATE KEY-----
  MIIEvgIBADANBg...
  -----END PRIVATE KEY-----
```

```
# After encryption
$ANSIBLE_VAULT;1.2;AES256
38366530326261376135653464636264613236363539623037336532623864
61363033336663616434626130653535613836343562366262646531396535
3131383964310a616334366130613838303537393137616163303935393730
...
```

---

## Encrypting Individual Variables

Encrypt a single variable instead of an entire file:

```bash
# Encrypt a string
ansible-vault encrypt_string 'supersecret123' --name 'db_password'

# Output:
# db_password: !vault |
#   $ANSIBLE_VAULT;1.2;AES256
#   38366530326261376135...
```

```yaml
# vars/main.yml — Mix of plain and encrypted variables
db_host: db.example.com
db_port: 5432
db_name: production
db_user: appuser
db_password: !vault |
  $ANSIBLE_VAULT;1.2;AES256
  38366530326261376135653464636264613236363539623037336532623864
  61363033336663616434626130653535613836343562366262646531396535
  3131383964310a616334366130613838303537393137616163303935393730
```

```bash
# Encrypt from stdin
echo -n 'my_api_key_value' | ansible-vault encrypt_string --stdin-name 'api_key'

# Encrypt with specific vault ID
ansible-vault encrypt_string 'secret' --name 'var' --vault-id prod@prompt
```

---

## Providing the Vault Password

### Interactive Prompt

```bash
ansible-playbook site.yml --ask-vault-pass
# or
ansible-playbook site.yml --ask-vault-password
```

### Password File

```bash
# File containing the password (single line, no newline)
echo 'MyVaultPassword123' > ~/.vault_pass
chmod 600 ~/.vault_pass

# Use password file
ansible-playbook site.yml --vault-password-file ~/.vault_pass

# In ansible.cfg (no CLI flag needed)
# [defaults]
# vault_password_file = ~/.vault_pass
```

### Password Script

```bash
#!/bin/bash
# ~/.vault_pass.sh — Retrieve from password manager
# Must be executable and output password to stdout

# Example: from environment variable
echo "${ANSIBLE_VAULT_PASSWORD}"

# Example: from AWS Secrets Manager
aws secretsmanager get-secret-value \
  --secret-id ansible-vault-password \
  --query SecretString \
  --output text

# Example: from HashiCorp Vault
vault kv get -field=password secret/ansible/vault
```

```bash
chmod +x ~/.vault_pass.sh
ansible-playbook site.yml --vault-password-file ~/.vault_pass.sh
```

### Environment Variable

```bash
export ANSIBLE_VAULT_PASSWORD_FILE=~/.vault_pass
ansible-playbook site.yml   # No flag needed
```

---

## Multiple Vault Passwords (Vault IDs)

```bash
# Encrypt with specific vault ID
ansible-vault encrypt --vault-id dev@prompt vars/dev-secrets.yml
ansible-vault encrypt --vault-id prod@~/.prod_vault_pass vars/prod-secrets.yml

# Encrypt string with vault ID
ansible-vault encrypt_string --vault-id prod@prompt 'secret' --name 'db_password'

# Run playbook with multiple vault passwords
ansible-playbook site.yml \
  --vault-id dev@prompt \
  --vault-id prod@~/.prod_vault_pass
```

```
  VAULT IDs
  ══════════════════════════════════════════════════════════

  ┌────────────────┐    Vault ID: dev
  │ dev-secrets.yml│────Password: devpass123
  └────────────────┘

  ┌────────────────┐    Vault ID: prod
  │prod-secrets.yml│────Password: (from file)
  └────────────────┘

  ansible-playbook site.yml \
    --vault-id dev@prompt \
    --vault-id prod@~/.prod_pass

  Ansible tries each vault ID until decryption succeeds
```

---

## Best Practices for Vault

### Separate Secret Files

```
  RECOMMENDED PROJECT STRUCTURE
  ══════════════════════════════════════════════════════════

  project/
  ├── group_vars/
  │   ├── all/
  │   │   ├── vars.yml          ◄── Plain variables
  │   │   └── vault.yml         ◄── Encrypted (prefix: vault_)
  │   ├── production/
  │   │   ├── vars.yml
  │   │   └── vault.yml
  │   └── staging/
  │       ├── vars.yml
  │       └── vault.yml
  └── site.yml
```

### Naming Convention for Vault Variables

```yaml
# group_vars/production/vault.yml (ENCRYPTED)
vault_db_password: supersecret123
vault_api_key: ak_live_abc123
vault_ssl_key: |
  -----BEGIN PRIVATE KEY-----
  ...

# group_vars/production/vars.yml (PLAIN TEXT)
db_password: "{{ vault_db_password }}"
api_key: "{{ vault_api_key }}"
ssl_key: "{{ vault_ssl_key }}"
```

This pattern:
1. Makes `grep` work on variable names (plain file references vault variables)
2. Clear which variables contain secrets (vault_ prefix)
3. Encrypted file only contains secrets

### .gitignore

```gitignore
# Never commit these
*.retry
.vault_pass
vault_password
*.vault_pass*

# But DO commit encrypted vault files (that's the whole point)
# !group_vars/*/vault.yml
```

---

## Using Vault in Playbooks

```yaml
---
- name: Deploy application with secrets
  hosts: appservers
  become: yes
  vars_files:
    - vars/common.yml
    - vars/secrets.yml           # Encrypted file
  tasks:
    - name: Configure database connection
      template:
        src: database.yml.j2
        dest: /etc/myapp/database.yml
        mode: '0600'                     # Restrict permissions
      no_log: true                       # Don't show in output

    - name: Set API key
      copy:
        content: "{{ api_key }}"
        dest: /etc/myapp/api_key
        mode: '0600'
      no_log: true
```

### no_log: Hiding Sensitive Output

```yaml
tasks:
  # Without no_log — password visible in output!
  - name: Create DB user (INSECURE)
    mysql_user:
      name: appuser
      password: "{{ db_password }}"

  # With no_log — output censored
  - name: Create DB user (SECURE)
    mysql_user:
      name: appuser
      password: "{{ db_password }}"
    no_log: true                         # Output shows "censored"
```

---

## Real-World Scenario: Secure Multi-Environment Deployment

```
  project/
  ├── ansible.cfg
  ├── inventories/
  │   ├── staging/
  │   │   ├── hosts.yml
  │   │   └── group_vars/
  │   │       └── all/
  │   │           ├── vars.yml
  │   │           └── vault.yml      ◄── vault-id: staging
  │   └── production/
  │       ├── hosts.yml
  │       └── group_vars/
  │           └── all/
  │               ├── vars.yml
  │               └── vault.yml      ◄── vault-id: prod
  └── deploy.yml
```

```yaml
# inventories/production/group_vars/all/vault.yml (encrypted with prod vault-id)
vault_db_password: "prod_p@ssw0rd_$ecure!"
vault_api_secret: "ak_live_x7y8z9"
vault_tls_key: |
  -----BEGIN PRIVATE KEY-----
  MIIEvgIBADANBg...

# inventories/production/group_vars/all/vars.yml
env: production
db_host: prod-db.internal
db_password: "{{ vault_db_password }}"
api_secret: "{{ vault_api_secret }}"
tls_key: "{{ vault_tls_key }}"
```

```bash
# Deploy to staging
ansible-playbook -i inventories/staging deploy.yml --vault-id staging@prompt

# Deploy to production
ansible-playbook -i inventories/production deploy.yml --vault-id prod@~/.prod_vault_pass

# CI/CD pipeline
ANSIBLE_VAULT_PASSWORD_FILE=~/.vault_pass ansible-playbook -i inventories/production deploy.yml
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| "Decryption failed" | Wrong password; verify with `ansible-vault view file.yml` |
| "is not a vault encrypted file" | File wasn't encrypted; run `ansible-vault encrypt` |
| Secret visible in output | Add `no_log: true` to tasks handling secrets |
| "ERROR! Vault password not provided" | Use `--ask-vault-pass` or set `vault_password_file` |
| Can't `grep` for secret variable | Use vault_ prefix pattern: grep plaintext vars file |
| Multiple vault passwords conflict | Use vault IDs to separate environments |

---

## Summary Table

| Command | Description |
|---------|-------------|
| `ansible-vault create` | Create new encrypted file |
| `ansible-vault encrypt` | Encrypt existing file |
| `ansible-vault decrypt` | Remove encryption |
| `ansible-vault edit` | Edit encrypted file |
| `ansible-vault view` | View without decrypting to disk |
| `ansible-vault rekey` | Change vault password |
| `ansible-vault encrypt_string` | Encrypt single variable |
| `--ask-vault-pass` | Prompt for password |
| `--vault-password-file` | Password file or script |
| `--vault-id` | Named vault identifier |
| `no_log: true` | Hide task output |

---

## Quick Revision Questions

1. **What encryption algorithm does Ansible Vault use?**
   > AES-256 symmetric encryption.

2. **How do you encrypt a single variable instead of a whole file?**
   > Use `ansible-vault encrypt_string 'value' --name 'variable_name'` and paste the result into your vars file.

3. **What is the recommended way to organize vault files?**
   > Keep secrets in separate `vault.yml` files with `vault_` prefixed variable names. Reference them from plain `vars.yml` files.

4. **How do you prevent secrets from appearing in playbook output?**
   > Add `no_log: true` to any task that handles sensitive variables.

5. **How do you manage different vault passwords for different environments?**
   > Use vault IDs: encrypt with `--vault-id env@source` and provide multiple vault IDs at runtime.

6. **How can you provide the vault password in CI/CD pipelines?**
   > Use `ANSIBLE_VAULT_PASSWORD_FILE` environment variable pointing to a file or executable script that retrieves the password.

---

[← Previous: Magic Variables](03-magic-variables.md) | [Next: Lookups →](05-lookups.md)
