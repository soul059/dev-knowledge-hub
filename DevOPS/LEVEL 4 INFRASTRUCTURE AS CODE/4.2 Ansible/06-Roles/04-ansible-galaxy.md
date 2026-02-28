# Chapter 6.4: Ansible Galaxy

[← Previous: Role Dependencies](03-role-dependencies.md) | [Next: Collections →](05-collections.md)

---

## Overview

**Ansible Galaxy** is the community hub for sharing and downloading Ansible roles and collections. The `ansible-galaxy` CLI tool manages downloading, installing, and scaffolding roles and collections from Galaxy, Git repositories, and other sources.

---

## Galaxy Architecture

```
  ┌──────────────────────────────────────────────────────┐
  │                   galaxy.ansible.com                  │
  │                                                      │
  │   ┌────────────┐  ┌────────────┐  ┌────────────┐    │
  │   │   Roles    │  │Collections │  │   Search    │    │
  │   │   1000s+   │  │   1000s+   │  │  & Browse   │    │
  │   └────────────┘  └────────────┘  └────────────┘    │
  └──────────────────────┬───────────────────────────────┘
                         │
                    ansible-galaxy CLI
                         │
              ┌──────────┴──────────┐
              │  Local Machine      │
              │  ~/.ansible/roles/  │
              │  ~/.ansible/        │
              │    collections/     │
              └─────────────────────┘
```

---

## Installing Roles

### From Galaxy

```bash
# Install a role
ansible-galaxy install geerlingguy.docker

# Install specific version
ansible-galaxy install geerlingguy.docker,6.1.0

# Install to custom path
ansible-galaxy install geerlingguy.docker -p ./roles/

# List installed roles
ansible-galaxy list

# Remove a role
ansible-galaxy remove geerlingguy.docker
```

### From Git

```bash
# From GitHub
ansible-galaxy install git+https://github.com/user/role.git

# With version/branch
ansible-galaxy install git+https://github.com/user/role.git,v2.0.0

# With custom name
ansible-galaxy install git+https://github.com/user/role.git,main,my_role
```

---

## Requirements File

```yaml
# requirements.yml
---
roles:
  # From Galaxy
  - name: geerlingguy.docker
    version: "6.1.0"

  - name: geerlingguy.nginx
    version: "3.1.0"

  # From GitHub
  - src: https://github.com/user/ansible-role-app.git
    scm: git
    version: main
    name: app

  # From a tarball
  - src: https://example.com/roles/custom-role.tar.gz
    name: custom_role

collections:
  - name: community.general
    version: ">=6.0.0"
  - name: ansible.posix
```

```bash
# Install all requirements
ansible-galaxy install -r requirements.yml

# Install roles only
ansible-galaxy role install -r requirements.yml

# Install collections only
ansible-galaxy collection install -r requirements.yml

# Force reinstall
ansible-galaxy install -r requirements.yml --force
```

---

## Creating a Role for Galaxy

### 1. Scaffold

```bash
ansible-galaxy init my_role
```

### 2. Write Good meta/main.yml

```yaml
# roles/my_role/meta/main.yml
---
galaxy_info:
  role_name: my_role
  namespace: my_namespace
  author: Your Name
  description: Brief description of what this role does
  company: Your Company
  license: MIT
  min_ansible_version: "2.12"

  platforms:
    - name: Ubuntu
      versions:
        - focal
        - jammy
    - name: EL
      versions:
        - "8"
        - "9"

  galaxy_tags:
    - web
    - nginx
    - proxy

dependencies: []
```

### 3. Write a Good README.md

```markdown
# Role: my_role

## Description
Installs and configures Nginx.

## Requirements
None.

## Role Variables
| Variable | Default | Description |
|----------|---------|-------------|
| http_port | 80 | HTTP listen port |
| server_name | localhost | Server hostname |

## Dependencies
None.

## Example Playbook
    - hosts: servers
      roles:
        - my_role

## License
MIT
```

### 4. Publish

```bash
# Login to Galaxy
ansible-galaxy login

# Import from GitHub (linked to Galaxy account)
ansible-galaxy import github_user repo_name
```

---

## Galaxy CLI Reference

| Command | Description |
|---------|-------------|
| `ansible-galaxy init <name>` | Create role skeleton |
| `ansible-galaxy install <role>` | Install a role |
| `ansible-galaxy install -r requirements.yml` | Install from requirements |
| `ansible-galaxy list` | List installed roles |
| `ansible-galaxy remove <role>` | Uninstall a role |
| `ansible-galaxy search <keyword>` | Search Galaxy |
| `ansible-galaxy info <role>` | Show role details |
| `ansible-galaxy import <user> <repo>` | Publish role to Galaxy |
| `ansible-galaxy collection init` | Create collection skeleton |
| `ansible-galaxy collection build` | Build collection tarball |
| `ansible-galaxy collection install` | Install a collection |
| `ansible-galaxy collection publish` | Publish collection |

---

## Using Galaxy Roles in Playbooks

```yaml
---
- name: Setup Docker hosts
  hosts: docker_hosts
  become: yes
  roles:
    - geerlingguy.docker
    - geerlingguy.pip

  vars:
    docker_users:
      - deploy
    pip_install_packages:
      - name: docker
```

---

## Private Galaxy (Automation Hub)

```ini
# ansible.cfg
[galaxy]
server_list = automation_hub, galaxy

[galaxy_server.automation_hub]
url = https://hub.internal.company.com/api/galaxy/
token = my_private_token

[galaxy_server.galaxy]
url = https://galaxy.ansible.com/
```

```
  GALAXY SERVER RESOLUTION
  ══════════════════════════════════════════════════════════

  1. Check automation_hub first (private)
  2. Fall back to galaxy.ansible.com (public)
```

---

## Real-World Scenario: Project Setup with Galaxy

```yaml
# requirements.yml
---
roles:
  - name: geerlingguy.docker
    version: "6.1.0"
  - name: geerlingguy.nginx
    version: "3.1.0"
  - name: geerlingguy.certbot
    version: "5.1.0"
  - name: geerlingguy.postgresql
    version: "3.4.0"

collections:
  - name: community.general
    version: ">=6.0.0"
  - name: community.docker
    version: ">=3.0.0"
```

```bash
# Initial project setup
mkdir -p project/{inventory,group_vars,roles}
cd project
ansible-galaxy install -r requirements.yml -p roles/
ansible-galaxy collection install -r requirements.yml
```

```
  project/
  ├── ansible.cfg
  ├── requirements.yml
  ├── inventory/
  ├── group_vars/
  ├── site.yml
  └── roles/
      ├── geerlingguy.docker/
      ├── geerlingguy.nginx/
      ├── geerlingguy.certbot/
      ├── geerlingguy.postgresql/
      └── my_custom_role/        ◄── Your own roles alongside Galaxy
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| `ERROR! the role 'x' was not found` | Run `ansible-galaxy install -r requirements.yml` |
| Version conflict | Pin exact versions in `requirements.yml` |
| SSL certificate error | Use `--ignore-certs` or configure CA bundle |
| Download timeout | Set `ANSIBLE_GALAXY_TIMEOUT=120` environment variable |
| Role already exists | Use `--force` to overwrite |
| Private hub auth failure | Verify token in `ansible.cfg` galaxy_server config |

---

## Summary Table

| Feature | Description |
|---------|-------------|
| Galaxy | Community hub for roles and collections |
| `requirements.yml` | Declare all role/collection dependencies |
| `ansible-galaxy install` | Download and install roles |
| `ansible-galaxy init` | Scaffold new role directory |
| Role versioning | Pin with `version:` in requirements |
| Private hub | Configure in `ansible.cfg` under `[galaxy_server]` |
| `.gitignore` roles | Add `roles/geerlingguy.*` — install via CI |

---

## Quick Revision Questions

1. **What is Ansible Galaxy?**
   > A community hub (galaxy.ansible.com) for sharing Ansible roles and collections, plus the CLI tool to manage them.

2. **How do you install all project role dependencies at once?**
   > Define them in `requirements.yml` and run `ansible-galaxy install -r requirements.yml`.

3. **How do you pin a Galaxy role to a specific version?**
   > Specify `version: "3.1.0"` in the `requirements.yml` entry for that role.

4. **What command creates a new role directory structure?**
   > `ansible-galaxy init <role_name>`.

5. **How do you configure a private Galaxy server?**
   > Add `[galaxy_server.<name>]` sections in `ansible.cfg` with `url` and `token`.

---

[← Previous: Role Dependencies](03-role-dependencies.md) | [Next: Collections →](05-collections.md)
