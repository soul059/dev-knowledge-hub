# Chapter 2.2: Inventory Basics

[← Previous: Installation](01-installation.md) | [Next: Ad-hoc Commands →](03-ad-hoc-commands.md)

---

## Overview

The **inventory** is how Ansible knows which machines to manage. It defines hosts, groups, and variables that determine how Ansible connects to and configures managed nodes.

---

## Default Inventory Location

```
  INVENTORY LOOKUP ORDER
  ══════════════════════════════════════════════

  1. Command line:  -i /path/to/inventory
  2. ansible.cfg:   inventory = ./inventory
  3. Environment:   ANSIBLE_INVENTORY
  4. Default:       /etc/ansible/hosts
```

---

## INI Format (Simple)

```ini
# inventory.ini

# Ungrouped hosts
server1.example.com
192.168.1.100

# Group of web servers
[webservers]
web1.example.com
web2.example.com
web3.example.com

# Group of database servers
[dbservers]
db1.example.com
db2.example.com

# Group with connection details
[loadbalancers]
lb1.example.com ansible_host=10.0.0.50 ansible_port=2222
lb2.example.com ansible_host=10.0.0.51

# Range patterns
[app_servers]
app[01:05].example.com    # app01 through app05
```

## YAML Format

```yaml
# inventory.yml
all:
  hosts:
    server1.example.com:
    192.168.1.100:
  children:
    webservers:
      hosts:
        web1.example.com:
        web2.example.com:
        web3.example.com:
    dbservers:
      hosts:
        db1.example.com:
        db2.example.com:
    loadbalancers:
      hosts:
        lb1.example.com:
          ansible_host: 10.0.0.50
          ansible_port: 2222
        lb2.example.com:
          ansible_host: 10.0.0.51
```

---

## Inventory Structure

```
  INVENTORY HIERARCHY
  ══════════════════════════════════════════════════════════

  ┌──────────────────────────────────────────────┐
  │                    all                        │
  │            (implicit top group)               │
  │                                               │
  │    ┌────────────┐    ┌────────────┐           │
  │    │ webservers │    │  dbservers │           │
  │    │            │    │            │           │
  │    │ web1       │    │ db1        │           │
  │    │ web2       │    │ db2        │           │
  │    │ web3       │    │            │           │
  │    └────────────┘    └────────────┘           │
  │                                               │
  │    ┌──────────────────────────────┐           │
  │    │   production (meta group)    │           │
  │    │   children: webservers,      │           │
  │    │             dbservers        │           │
  │    └──────────────────────────────┘           │
  │                                               │
  │    ungrouped: server1, 192.168.1.100          │
  └──────────────────────────────────────────────┘

  Special Groups (always exist):
  • all       - contains every host
  • ungrouped - hosts not in any named group
```

---

## Common Inventory Commands

```bash
# List all hosts in inventory
ansible all --list-hosts -i inventory.ini

# List hosts in a specific group
ansible webservers --list-hosts

# Graph view of inventory
ansible-inventory --graph

# Output:
# @all:
#   |--@ungrouped:
#   |  |--server1.example.com
#   |--@webservers:
#   |  |--web1.example.com
#   |  |--web2.example.com
#   |--@dbservers:
#   |  |--db1.example.com

# Detailed inventory info (JSON)
ansible-inventory --list

# Show info for a specific host
ansible-inventory --host web1.example.com
```

---

## Host Connection Parameters

```ini
# Common connection parameters
[servers]
# Linux host via SSH
linux1  ansible_host=192.168.1.10  ansible_user=deploy  ansible_port=22

# Windows host via WinRM
win1    ansible_host=192.168.1.20  ansible_user=admin  ansible_connection=winrm  ansible_port=5986

# Local execution
local   ansible_connection=local

# Host with SSH key
secure1 ansible_host=10.0.0.5  ansible_ssh_private_key_file=~/.ssh/deploy_key

# Host with password (not recommended)
legacy1 ansible_host=10.0.0.6  ansible_user=root  ansible_password=secret123

# Docker container
container1 ansible_connection=docker
```

### Connection Parameters Reference

| Parameter | Description | Default |
|-----------|-------------|---------|
| `ansible_host` | IP or hostname to connect to | inventory hostname |
| `ansible_port` | SSH port | 22 |
| `ansible_user` | SSH user | current user |
| `ansible_password` | SSH password | (use keys instead) |
| `ansible_ssh_private_key_file` | Path to SSH key | ~/.ssh/id_rsa |
| `ansible_connection` | Connection type | ssh |
| `ansible_become` | Enable privilege escalation | false |
| `ansible_become_user` | User to become | root |
| `ansible_python_interpreter` | Path to Python on remote | auto-detected |

---

## Real-World Scenarios

### Scenario 1: Small Lab
```ini
# lab-inventory.ini
[control]
localhost ansible_connection=local

[web]
192.168.56.101
192.168.56.102

[db]
192.168.56.103

[lab:children]
web
db
```

### Scenario 2: Multi-Environment
```ini
# Separate files for each environment
# inventories/dev.ini
[webservers]
dev-web1.internal

[dbservers]
dev-db1.internal

# inventories/prod.ini
[webservers]
prod-web[1:3].example.com

[dbservers]
prod-db[1:2].example.com
```

```bash
# Use specific environment
ansible-playbook -i inventories/dev.ini site.yml
ansible-playbook -i inventories/prod.ini site.yml
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| "No hosts matched" | Check group names, host patterns, and inventory path |
| Can't find inventory file | Verify `ansible.cfg` inventory setting or use `-i` flag |
| Host unreachable | Check `ansible_host`, port, and network connectivity |
| Wrong Python on remote | Set `ansible_python_interpreter=/usr/bin/python3` |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| Inventory | File listing managed hosts and their groupings |
| INI format | Simple key-value format for inventory |
| YAML format | Structured format with hierarchy support |
| `all` group | Implicit group containing every host |
| `ungrouped` | Hosts not assigned to any named group |
| `ansible_host` | Actual IP/hostname for connection |
| `ansible_connection` | Connection type (ssh, winrm, local, docker) |
| `--list-hosts` | Show which hosts match a pattern |

---

## Quick Revision Questions

1. **What is the default location of the Ansible inventory file?**
   > `/etc/ansible/hosts`, but it can be overridden via `ansible.cfg` or the `-i` flag.

2. **What are the two special groups that always exist in inventory?**
   > `all` (every host) and `ungrouped` (hosts not in any named group).

3. **How do you specify a range of hosts in inventory?**
   > Use range patterns: `web[01:05].example.com` for web01 through web05.

4. **What parameter controls how Ansible connects to a managed node?**
   > `ansible_connection` (values: ssh, winrm, local, docker, etc.)

5. **How do you manage multiple environments (dev, staging, prod)?**
   > Use separate inventory files per environment and specify with `-i` flag.

6. **What command shows a visual graph of inventory groups?**
   > `ansible-inventory --graph`

---

[← Previous: Installation](01-installation.md) | [Next: Ad-hoc Commands →](03-ad-hoc-commands.md)
