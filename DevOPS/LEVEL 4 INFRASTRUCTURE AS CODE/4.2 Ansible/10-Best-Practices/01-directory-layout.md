# Chapter 10.1: Directory Layout

[← Previous: Performance Tuning](../09-Advanced-Features/06-performance-tuning.md) | [Next: Playbook Best Practices →](02-playbook-best-practices.md)

---

## Overview

A well-organised directory structure is the foundation of maintainable Ansible projects. This chapter covers the **recommended project layout**, explains where each file type belongs, and shows how the structure scales from small projects to enterprise environments.

---

## Recommended Directory Layout

```
  ansible-project/
  ══════════════════════════════════════════════════════════

  ├── ansible.cfg                 ◄── Project config
  ├── requirements.yml            ◄── Galaxy dependencies
  │
  ├── inventories/
  │   ├── production/
  │   │   ├── hosts.yml           ◄── Production hosts
  │   │   ├── group_vars/
  │   │   │   ├── all.yml         ◄── Vars for all hosts
  │   │   │   ├── webservers.yml  ◄── Vars per group
  │   │   │   └── dbservers.yml
  │   │   └── host_vars/
  │   │       ├── web01.yml       ◄── Vars per host
  │   │       └── db01.yml
  │   │
  │   └── staging/
  │       ├── hosts.yml
  │       ├── group_vars/
  │       │   └── all.yml
  │       └── host_vars/
  │
  ├── playbooks/
  │   ├── site.yml                ◄── Main playbook
  │   ├── webservers.yml
  │   ├── dbservers.yml
  │   └── monitoring.yml
  │
  ├── roles/
  │   ├── common/                 ◄── Shared baseline role
  │   ├── webserver/
  │   ├── database/
  │   └── monitoring/
  │
  ├── library/                    ◄── Custom modules
  ├── module_utils/               ◄── Module shared code
  ├── callback_plugins/           ◄── Custom callbacks
  ├── filter_plugins/             ◄── Custom Jinja2 filters
  │
  ├── files/                      ◄── Static files (non-role)
  ├── templates/                  ◄── Templates (non-role)
  │
  └── group_vars/                 ◄── Cross-inventory vars
      └── all.yml                     (optional)
```

---

## Key Directory Purposes

| Directory / File | Purpose |
|-----------------|---------|
| `ansible.cfg` | Project-specific config (inventory path, roles path, etc.) |
| `inventories/` | Separate dirs per environment |
| `group_vars/` | Variables applied to inventory groups |
| `host_vars/` | Variables applied to individual hosts |
| `playbooks/` | Entry-point playbooks |
| `roles/` | Reusable role components |
| `library/` | Custom modules |
| `filter_plugins/` | Custom Jinja2 filters |
| `files/` / `templates/` | Non-role static files and templates |
| `requirements.yml` | Galaxy roles/collections to install |

---

## Environment Separation

```
  inventories/
  ══════════════════════════════════════════════════════════

  ├── development/
  │   ├── hosts.yml
  │   └── group_vars/
  │       └── all.yml          ◄── debug: true
  │                                env: development
  │
  ├── staging/
  │   ├── hosts.yml
  │   └── group_vars/
  │       └── all.yml          ◄── debug: false
  │                                env: staging
  │
  └── production/
      ├── hosts.yml
      └── group_vars/
          └── all.yml          ◄── debug: false
                                   env: production

  # Usage:
  ansible-playbook -i inventories/staging playbooks/site.yml
  ansible-playbook -i inventories/production playbooks/site.yml
```

---

## ansible.cfg for the Project

```ini
# ansible.cfg (at project root)
[defaults]
inventory       = inventories/staging/hosts.yml   # Default inventory
roles_path      = roles
library         = library
filter_plugins  = filter_plugins
callback_plugins = callback_plugins
stdout_callback = yaml
forks           = 20

[ssh_connection]
pipelining = True
ssh_args = -o ControlMaster=auto -o ControlPersist=60s

[privilege_escalation]
become = True
become_method = sudo
```

---

## Playbook Organisation

### Master Playbook (site.yml)

```yaml
# playbooks/site.yml — imports all sub-playbooks
---
- import_playbook: webservers.yml
- import_playbook: dbservers.yml
- import_playbook: monitoring.yml
```

### Sub-Playbooks

```yaml
# playbooks/webservers.yml
---
- name: Configure web servers
  hosts: webservers
  roles:
    - common
    - webserver

# playbooks/dbservers.yml
---
- name: Configure database servers
  hosts: dbservers
  roles:
    - common
    - database
```

---

## group_vars Organisation

### Single File per Group

```yaml
# inventories/production/group_vars/webservers.yml
---
http_port: 80
max_connections: 1000
ssl_enabled: true
```

### Directory per Group (for complex vars)

```
  group_vars/
  ├── webservers/
  │   ├── main.yml            ◄── General settings
  │   ├── ssl.yml             ◄── SSL settings
  │   └── vault.yml           ◄── Encrypted secrets
  │
  └── dbservers/
      ├── main.yml
      └── vault.yml
```

```yaml
# group_vars/webservers/vault.yml (encrypted)
---
vault_ssl_private_key: |
  -----BEGIN RSA PRIVATE KEY-----
  ...
  -----END RSA PRIVATE KEY-----
```

---

## Scaling Patterns

### Small Project (1-10 hosts)

```
  project/
  ├── ansible.cfg
  ├── inventory.yml
  ├── group_vars/
  │   └── all.yml
  ├── site.yml
  └── roles/
      └── webserver/
```

### Medium Project (10-100 hosts)

```
  project/
  ├── ansible.cfg
  ├── inventories/
  │   ├── staging/
  │   └── production/
  ├── playbooks/
  │   ├── site.yml
  │   └── webservers.yml
  ├── roles/
  │   ├── common/
  │   └── webserver/
  └── requirements.yml
```

### Large / Enterprise (100+ hosts)

```
  project/
  ├── ansible.cfg
  ├── inventories/
  │   ├── development/
  │   ├── staging/
  │   ├── production/
  │   │   ├── hosts.yml
  │   │   ├── group_vars/
  │   │   │   ├── all/
  │   │   │   │   ├── main.yml
  │   │   │   │   └── vault.yml
  │   │   │   ├── webservers/
  │   │   │   └── dbservers/
  │   │   └── host_vars/
  │   └── dr/                     ◄── Disaster recovery
  ├── playbooks/
  │   ├── site.yml
  │   ├── webservers.yml
  │   ├── dbservers.yml
  │   ├── deploy.yml
  │   └── rollback.yml
  ├── roles/
  │   ├── requirements.yml        ◄── External roles
  │   ├── common/
  │   ├── hardening/
  │   ├── webserver/
  │   ├── database/
  │   ├── monitoring/
  │   └── backup/
  ├── collections/
  │   └── requirements.yml        ◄── External collections
  ├── library/
  ├── module_utils/
  ├── filter_plugins/
  ├── callback_plugins/
  ├── docs/                       ◄── Project documentation
  │   └── runbooks/
  ├── tests/                      ◄── Integration tests
  │   ├── Vagrantfile
  │   └── test_playbooks/
  ├── .gitignore
  ├── .ansible-lint
  └── Makefile                    ◄── Common commands
```

---

## .gitignore for Ansible Projects

```gitignore
# .gitignore
*.retry
*.pyc
__pycache__/
.vagrant/
*.log
/tmp/
facts_cache/
*.tar.gz
.molecule/
```

---

## Real-World Scenario: Multi-Team Project

```
  REPOSITORY STRATEGY
  ══════════════════════════════════════════════════════════

  Option A: Monorepo                Option B: Multi-repo
  ────────────────────              ────────────────────
  ansible-platform/                 repo: ansible-infra
  ├── inventories/                  ├── inventories/
  ├── roles/                        └── roles/common
  │   ├── common/
  │   ├── team-a-app/               repo: ansible-app-a
  │   └── team-b-app/               ├── requirements.yml
  └── playbooks/                    └── roles/app-a/

  Pro: Single source of truth       Pro: Team autonomy
  Con: Merge conflicts              Con: Coordination overhead
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Variables not loading | Check that `group_vars/` is inside the correct inventory directory |
| Role not found | Set `roles_path` in ansible.cfg or use full path |
| Wrong inventory used | Check `ansible.cfg` default inventory and `-i` flag |
| Vault file not decrypted | Ensure vault password file is configured |
| Custom module not found | Place in `library/` or set `library` path in ansible.cfg |

---

## Summary Table

| Component | Location | Purpose |
|-----------|----------|---------|
| Config | `ansible.cfg` | Project settings |
| Inventories | `inventories/<env>/` | Per-environment hosts and vars |
| Group vars | `group_vars/<group>.yml` | Variables by group |
| Host vars | `host_vars/<host>.yml` | Variables by host |
| Playbooks | `playbooks/` | Entry-point plays |
| Roles | `roles/` | Reusable automation units |
| Custom modules | `library/` | Project-specific modules |
| Plugins | `*_plugins/` | Custom filters, callbacks |
| Dependencies | `requirements.yml` | Galaxy roles/collections |

---

## Quick Revision Questions

1. **Why should inventories be separated by environment?**
   > To keep environment-specific variables (hostnames, credentials, settings) isolated, preventing accidental cross-environment operations.

2. **What is the purpose of `site.yml`?**
   > It serves as the master playbook that imports all sub-playbooks, allowing you to configure the entire infrastructure with a single command.

3. **How can you store both plain and encrypted variables for a group?**
   > Use a directory instead of a file for the group (e.g., `group_vars/webservers/main.yml` and `group_vars/webservers/vault.yml`).

4. **Where should custom modules be placed?**
   > In a `library/` directory at the project root, or in a path configured by the `library` setting in ansible.cfg.

5. **How do you specify which inventory to use at runtime?**
   > Use the `-i` flag: `ansible-playbook -i inventories/production playbooks/site.yml`

---

[← Previous: Performance Tuning](../09-Advanced-Features/06-performance-tuning.md) | [Next: Playbook Best Practices →](02-playbook-best-practices.md)
