# Chapter 3.1: Static Inventory

[← Previous: SSH and Authentication](../02-Ansible-Basics/06-ssh-and-authentication.md) | [Next: Dynamic Inventory →](02-dynamic-inventory.md)

---

## Overview

Static inventory is the simplest way to define your managed nodes. It's a manually maintained file (INI or YAML) listing all hosts, groups, and their variables. Ideal for stable environments where infrastructure doesn't change frequently.

---

## INI Format (Detailed)

```ini
# inventory/hosts.ini

# === UNGROUPED HOSTS ===
mail.example.com
jump.example.com ansible_host=10.0.0.1

# === WEB SERVERS ===
[webservers]
web1.example.com ansible_host=192.168.1.10
web2.example.com ansible_host=192.168.1.11
web3.example.com ansible_host=192.168.1.12

# === DATABASE SERVERS ===
[dbservers]
db-primary.example.com ansible_host=192.168.2.10 db_role=primary
db-replica.example.com ansible_host=192.168.2.11 db_role=replica

# === APPLICATION SERVERS ===
[appservers]
app[01:05].example.com          # Range: app01 through app05

# === LOAD BALANCERS ===
[loadbalancers]
lb-[a:c].example.com            # Range: lb-a through lb-c

# === META GROUPS (Groups of Groups) ===
[production:children]
webservers
dbservers
appservers
loadbalancers

[staging:children]
staging_web
staging_db

# === STAGING GROUPS ===
[staging_web]
staging-web1.example.com

[staging_db]
staging-db1.example.com

# === GROUP VARIABLES ===
[webservers:vars]
ansible_user=deploy
http_port=80
max_connections=1000

[dbservers:vars]
ansible_user=dbadmin
ansible_port=2222
db_port=5432

[production:vars]
env=production
monitoring=enabled

[all:vars]
ansible_python_interpreter=/usr/bin/python3
ntp_server=time.example.com
```

---

## YAML Format (Detailed)

```yaml
# inventory/hosts.yml
all:
  vars:
    ansible_python_interpreter: /usr/bin/python3
    ntp_server: time.example.com
  hosts:
    mail.example.com:
    jump.example.com:
      ansible_host: 10.0.0.1
  children:
    production:
      vars:
        env: production
        monitoring: enabled
      children:
        webservers:
          vars:
            ansible_user: deploy
            http_port: 80
            max_connections: 1000
          hosts:
            web1.example.com:
              ansible_host: 192.168.1.10
            web2.example.com:
              ansible_host: 192.168.1.11
            web3.example.com:
              ansible_host: 192.168.1.12
        dbservers:
          vars:
            ansible_user: dbadmin
            db_port: 5432
          hosts:
            db-primary.example.com:
              ansible_host: 192.168.2.10
              db_role: primary
            db-replica.example.com:
              ansible_host: 192.168.2.11
              db_role: replica
        appservers:
          hosts:
            app01.example.com:
            app02.example.com:
            app03.example.com:
    staging:
      vars:
        env: staging
      children:
        staging_web:
          hosts:
            staging-web1.example.com:
        staging_db:
          hosts:
            staging-db1.example.com:
```

---

## Inventory Directory Structure

```
  RECOMMENDED INVENTORY LAYOUT
  ══════════════════════════════════════════════════════════

  inventory/
  ├── production/
  │   ├── hosts.ini              ← Host definitions
  │   ├── group_vars/
  │   │   ├── all.yml            ← Variables for all hosts
  │   │   ├── webservers.yml     ← Variables for webservers
  │   │   └── dbservers.yml      ← Variables for dbservers
  │   └── host_vars/
  │       ├── web1.example.com.yml  ← Host-specific vars
  │       └── db-primary.example.com.yml
  │
  ├── staging/
  │   ├── hosts.ini
  │   ├── group_vars/
  │   │   └── all.yml
  │   └── host_vars/
  │
  └── development/
      ├── hosts.ini
      ├── group_vars/
      │   └── all.yml
      └── host_vars/

  Usage:
  ansible-playbook -i inventory/production site.yml
  ansible-playbook -i inventory/staging site.yml
```

---

## Range Patterns

```ini
# Numeric ranges
[web_cluster]
web[01:50].example.com          # web01 through web50
web[001:100].example.com        # web001 through web100 (zero-padded)

# Alphabetic ranges
[nodes]
node-[a:f].example.com          # node-a through node-f

# Stepped ranges
[db_shards]
db[0:8:2].example.com           # db0, db2, db4, db6, db8

# IP address ranges
[lab_servers]
192.168.1.[1:254]               # 192.168.1.1 through 192.168.1.254

# Combining patterns
[cluster]
rack[1:3]-node[01:10].dc1.example.com
# rack1-node01 through rack3-node10
```

---

## Verifying Inventory

```bash
# List all hosts
ansible all --list-hosts -i inventory/hosts.ini

# List hosts in a group
ansible webservers --list-hosts

# Show inventory graph
ansible-inventory --graph
# @all:
#   |--@production:
#   |  |--@webservers:
#   |  |  |--web1.example.com
#   |  |  |--web2.example.com
#   |  |--@dbservers:
#   |  |  |--db-primary.example.com

# Export inventory as JSON
ansible-inventory --list -y    # YAML output
ansible-inventory --list       # JSON output

# Show specific host variables
ansible-inventory --host web1.example.com
```

---

## Real-World Scenarios

### Scenario 1: Multi-datacenter Setup
```ini
[dc1_web]
dc1-web[01:10].example.com

[dc2_web]
dc2-web[01:10].example.com

[webservers:children]
dc1_web
dc2_web

[dc1:children]
dc1_web
dc1_db

[dc2:children]
dc2_web
dc2_db
```

### Scenario 2: Different Credentials Per Environment
```ini
# inventory/production/hosts.ini
[webservers]
prod-web1.example.com
prod-web2.example.com

[webservers:vars]
ansible_user=ansible-svc
ansible_ssh_private_key_file=/etc/ansible/keys/prod_key
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Hosts not appearing | Check group names, indentation (YAML), and syntax |
| Range not expanding | Verify range format `[start:end]` with no spaces |
| Variables not applying | Check group_vars file naming matches group name |
| Duplicate hosts in output | Host may be in multiple groups (this is OK) |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| Static inventory | Manually maintained file listing hosts and groups |
| INI format | Simple text format with `[group]` sections |
| YAML format | Structured format with `all.children` hierarchy |
| Ranges | Expand to multiple hosts: `web[01:50]` |
| Meta groups | Groups of groups using `[group:children]` |
| Group vars | Variables applied to all hosts in a group |
| Host vars | Variables specific to one host |
| Inventory directory | Folder-based inventory with group_vars/host_vars |

---

## Quick Revision Questions

1. **What are the two formats for static inventory files?**
   > INI format and YAML format. INI is simpler; YAML supports deeper nesting.

2. **How do you create a group of groups in INI format?**
   > Use `[parent_group:children]` followed by child group names.

3. **What is the difference between group_vars and host_vars?**
   > Group vars apply to all hosts in a group; host vars are specific to a single host and take higher precedence.

4. **How do you specify a range of 100 hosts?**
   > Use range patterns: `web[001:100].example.com` for zero-padded numbering.

5. **Why use separate inventory directories per environment?**
   > To prevent accidentally running production playbooks against staging and to maintain clear separation of variables.

6. **How do you verify your inventory is correctly parsed?**
   > Use `ansible-inventory --graph` for visual tree or `ansible-inventory --list` for full JSON output.

---

[← Previous: SSH and Authentication](../02-Ansible-Basics/06-ssh-and-authentication.md) | [Next: Dynamic Inventory →](02-dynamic-inventory.md)
