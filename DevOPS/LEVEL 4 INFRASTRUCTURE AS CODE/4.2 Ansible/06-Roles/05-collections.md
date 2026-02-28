# Chapter 6.5: Collections

[← Previous: Ansible Galaxy](04-ansible-galaxy.md) | [Next: Role Testing →](06-role-testing.md)

---

## Overview

**Collections** are the modern distribution format for Ansible content. A collection bundles roles, modules, plugins, playbooks, and documentation into a single distributable package with proper namespacing.

---

## Collections vs Roles

```
  ROLE                                 COLLECTION
  ═══════════════════                  ═══════════════════
  • Single unit of automation          • Bundle of multiple content types
  • Contains tasks, handlers,          • Contains roles + modules +
    templates, vars                      plugins + playbooks
  • Namespace: flat name               • Namespace: author.name
    (geerlingguy.docker)                 (community.docker)
  • galaxy.ansible.com/roles           • galaxy.ansible.com/collections
```

---

## Collection Structure

```
  namespace/
  └── collection_name/
      ├── galaxy.yml               ◄── Collection metadata
      ├── README.md
      ├── docs/
      ├── plugins/
      │   ├── modules/             ◄── Custom modules
      │   ├── inventory/           ◄── Inventory plugins
      │   ├── lookup/              ◄── Lookup plugins
      │   ├── filter/              ◄── Filter plugins
      │   ├── callback/            ◄── Callback plugins
      │   └── connection/          ◄── Connection plugins
      ├── roles/
      │   ├── role_one/
      │   └── role_two/
      ├── playbooks/
      │   └── deploy.yml
      ├── meta/
      │   └── runtime.yml          ◄── Redirects, deprecations
      └── tests/
```

---

## Key Built-in Collections

| Collection | Content |
|------------|---------|
| `ansible.builtin` | Core modules (copy, file, template, yum, apt, etc.) |
| `ansible.posix` | POSIX modules (authorized_key, mount, seboolean) |
| `community.general` | General-purpose modules (archive, timezone, etc.) |
| `community.docker` | Docker modules and inventory |
| `community.mysql` | MySQL modules |
| `community.postgresql` | PostgreSQL modules |
| `amazon.aws` | AWS cloud modules |
| `azure.azcollection` | Azure cloud modules |
| `google.cloud` | GCP modules |

---

## Installing Collections

```bash
# From Galaxy
ansible-galaxy collection install community.docker

# Specific version
ansible-galaxy collection install community.docker:3.4.0

# From requirements file
ansible-galaxy collection install -r requirements.yml

# From a tarball
ansible-galaxy collection install community-docker-3.4.0.tar.gz

# To a custom path
ansible-galaxy collection install community.docker -p ./collections/
```

### requirements.yml

```yaml
---
collections:
  - name: community.general
    version: ">=6.0.0,<7.0.0"
  - name: community.docker
    version: "3.4.0"
  - name: amazon.aws
    version: ">=5.0.0"
  - name: ansible.posix
```

---

## Using Collection Content

### Fully Qualified Collection Name (FQCN)

```yaml
---
- name: Use FQCN for modules
  hosts: all
  tasks:
    # Fully qualified — always unambiguous
    - ansible.builtin.copy:
        src: file.txt
        dest: /tmp/file.txt

    - community.docker.docker_container:
        name: myapp
        image: nginx:latest
        state: started

    - community.general.timezone:
        name: Europe/London

    - ansible.posix.authorized_key:
        user: deploy
        key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
```

### Using `collections` Keyword (Shorthand)

```yaml
---
- name: Using collections keyword
  hosts: all
  collections:
    - community.docker
    - community.general

  tasks:
    # Can now use short names
    - docker_container:
        name: myapp
        image: nginx:latest
        state: started

    - timezone:
        name: Europe/London
```

> **Best Practice**: Always use FQCN for clarity and to avoid name collisions.

---

## Collection Roles

```yaml
# Using a role from a collection
- hosts: all
  roles:
    - role: geerlingguy.docker.docker
    #       namespace.collection.role_name

# Or with import_role
  tasks:
    - import_role:
        name: geerlingguy.docker.docker
```

---

## Creating a Collection

### 1. Scaffold

```bash
ansible-galaxy collection init my_namespace.my_collection
```

```
  my_namespace/my_collection/
  ├── docs/
  ├── galaxy.yml
  ├── meta/
  │   └── runtime.yml
  ├── plugins/
  │   └── README.md
  ├── README.md
  └── roles/
```

### 2. galaxy.yml

```yaml
# galaxy.yml
---
namespace: my_namespace
name: my_collection
version: 1.0.0
readme: README.md
authors:
  - Your Name <you@example.com>
description: My custom Ansible collection
license:
  - MIT
repository: https://github.com/user/my_collection
dependencies:
  community.general: ">=6.0.0"
  ansible.posix: "*"
tags:
  - infrastructure
  - linux
```

### 3. Build and Publish

```bash
# Build tarball
ansible-galaxy collection build

# Publish to Galaxy
ansible-galaxy collection publish my_namespace-my_collection-1.0.0.tar.gz --api-key=<key>
```

---

## Collection Search Path

```ini
# ansible.cfg
[defaults]
collections_path = ./collections:~/.ansible/collections:/usr/share/ansible/collections
```

```
  COLLECTION SEARCH ORDER
  ══════════════════════════════════════════════════════════

  1. ./collections/ansible_collections/
  2. ~/.ansible/collections/ansible_collections/
  3. /usr/share/ansible/collections/ansible_collections/
```

---

## runtime.yml — Redirects and Deprecations

```yaml
# meta/runtime.yml
---
requires_ansible: ">=2.12.0"
plugin_routing:
  modules:
    old_module_name:
      redirect: my_namespace.my_collection.new_module_name
    deprecated_module:
      deprecation:
        removal_version: "3.0.0"
        warning_text: Use new_module instead
```

---

## Real-World Scenario: Multi-Cloud Project

```yaml
# requirements.yml
---
collections:
  - name: amazon.aws
    version: ">=5.0.0"
  - name: azure.azcollection
    version: ">=1.14.0"
  - name: google.cloud
    version: ">=1.1.0"
  - name: community.general
  - name: ansible.posix
```

```yaml
# deploy.yml
---
- name: Provision AWS infrastructure
  hosts: localhost
  connection: local
  tasks:
    - amazon.aws.ec2_instance:
        name: web-server
        instance_type: t3.micro
        image_id: ami-0abcdef1234567890
        state: running

- name: Provision Azure resources
  hosts: localhost
  connection: local
  tasks:
    - azure.azcollection.azure_rm_virtualmachine:
        name: web-vm
        resource_group: myResourceGroup
        vm_size: Standard_B1s
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| `couldn't resolve module/action` | Install the collection: `ansible-galaxy collection install <name>` |
| Version conflict | Pin exact versions in `requirements.yml` |
| FQCN too verbose | Use `collections:` keyword in play, but prefer FQCN |
| Custom collection not found | Check `collections_path` in `ansible.cfg` |
| Collection dependency missing | Check `galaxy.yml` dependencies, install them |

---

## Summary Table

| Feature | Description |
|---------|-------------|
| FQCN | `namespace.collection.module` — unambiguous module reference |
| `galaxy.yml` | Collection metadata (version, deps, authors) |
| `requirements.yml` | Declare collection dependencies |
| `collections_path` | Search path for installed collections |
| `runtime.yml` | Module redirects and deprecation notices |
| `ansible-galaxy collection build` | Create distributable tarball |
| `ansible-galaxy collection install` | Install from Galaxy or file |

---

## Quick Revision Questions

1. **What is a Fully Qualified Collection Name (FQCN)?**
   > The complete reference format: `namespace.collection.module_name` (e.g., `community.docker.docker_container`).

2. **What is the difference between a collection and a role?**
   > A role is a single automation unit (tasks + handlers + templates). A collection bundles multiple roles, modules, plugins, and playbooks under a namespace.

3. **Where does Ansible look for installed collections?**
   > In paths listed in `collections_path`: `./collections/`, `~/.ansible/collections/`, `/usr/share/ansible/collections/`.

4. **How do you specify collection dependencies for a project?**
   > List them under `collections:` in a `requirements.yml` file and install with `ansible-galaxy collection install -r requirements.yml`.

5. **Why is FQCN preferred over short module names?**
   > FQCN avoids name collisions between modules in different collections and makes it clear which collection provides the module.

---

[← Previous: Ansible Galaxy](04-ansible-galaxy.md) | [Next: Role Testing →](06-role-testing.md)
