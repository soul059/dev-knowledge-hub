# Chapter 1.5: Ansible Overview and Architecture

[← Previous: CM Tools Comparison](04-cm-tools-comparison.md) | [Next: Unit 2 - Installation →](../02-Ansible-Basics/01-installation.md)

---

## Overview

Ansible is an **open-source IT automation engine** created by Michael DeHaan in 2012, acquired by Red Hat in 2015. It automates configuration management, application deployment, orchestration, and many other IT processes. This chapter provides a comprehensive overview of Ansible's architecture and core components.

---

## Ansible Architecture

```
  ANSIBLE COMPLETE ARCHITECTURE
  ══════════════════════════════════════════════════════════════════

  ┌──────────────────────────────────────────────────────────────┐
  │                     CONTROL NODE                              │
  │                                                               │
  │  ┌───────────────────────────────────────────────────────┐   │
  │  │                    USER INPUT                          │   │
  │  │  $ ansible-playbook -i inventory site.yml             │   │
  │  └───────────────────────┬───────────────────────────────┘   │
  │                          │                                    │
  │                          ▼                                    │
  │  ┌──────────┐    ┌──────────────┐    ┌──────────────────┐   │
  │  │Inventory │───►│   ANSIBLE    │◄───│   Playbooks      │   │
  │  │          │    │   ENGINE     │    │   (YAML)         │   │
  │  │- hosts   │    │              │    │   - plays        │   │
  │  │- groups  │    │  ┌────────┐  │    │   - tasks        │   │
  │  │- vars    │    │  │ Plugin │  │    │   - handlers     │   │
  │  └──────────┘    │  │ System │  │    │   - roles        │   │
  │                  │  └────────┘  │    └──────────────────┘   │
  │  ┌──────────┐    │  ┌────────┐  │    ┌──────────────────┐   │
  │  │ansible.  │───►│  │Modules │  │◄───│   Roles          │   │
  │  │cfg       │    │  │Library │  │    │   - tasks        │   │
  │  │          │    │  └────────┘  │    │   - handlers     │   │
  │  │- forks   │    │              │    │   - templates    │   │
  │  │- timeout │    └──────┬───────┘    │   - vars         │   │
  │  │- SSH opts│           │            └──────────────────┘   │
  │  └──────────┘           │                                    │
  │                         │  Connection Plugins                │
  │              ┌──────────┼──────────┐                         │
  │              │    SSH    │  WinRM   │                         │
  │              │   Local   │  Docker  │                         │
  │              └──────────┼──────────┘                         │
  └─────────────────────────│────────────────────────────────────┘
                            │
              ┌─────────────┼─────────────┐
              │             │             │
              ▼             ▼             ▼
  ┌───────────────┐ ┌──────────────┐ ┌──────────────┐
  │  MANAGED NODE │ │ MANAGED NODE │ │ MANAGED NODE │
  │  (Linux)      │ │ (Windows)    │ │ (Network)    │
  │               │ │              │ │              │
  │  ┌─────────┐  │ │ ┌──────────┐ │ │ ┌──────────┐│
  │  │ Python  │  │ │ │PowerShell│ │ │ │  API/CLI ││
  │  │ (2.7/3) │  │ │ │  + .NET  │ │ │ │          ││
  │  └─────────┘  │ │ └──────────┘ │ │ └──────────┘│
  │               │ │              │ │              │
  │  No agent!    │ │  No agent!   │ │  No agent!  │
  └───────────────┘ └──────────────┘ └──────────────┘
```

---

## Core Components

### 1. Control Node
The machine where Ansible is installed and from where automation is run.

```
  CONTROL NODE REQUIREMENTS
  ═══════════════════════════════════════

  ┌──────────────────────────────┐
  │        Control Node          │
  │                              │
  │  ✅ Linux/macOS/WSL          │
  │  ✅ Python 3.8+              │
  │  ✅ Ansible installed        │
  │  ✅ SSH client               │
  │  ✅ Network access to nodes  │
  │                              │
  │  ❌ Windows (not supported   │
  │     as control node natively │
  │     Use WSL instead)         │
  └──────────────────────────────┘
```

### 2. Managed Nodes
Target machines that Ansible manages.

```
  MANAGED NODE REQUIREMENTS
  ═══════════════════════════════════════

  Linux/Unix:
  ┌──────────────────────────────┐
  │  ✅ Python 2.7+ or 3.5+     │
  │  ✅ SSH server running       │
  │  ✅ User with sudo (optional)│
  │  ❌ No Ansible needed        │
  └──────────────────────────────┘

  Windows:
  ┌──────────────────────────────┐
  │  ✅ PowerShell 3.0+          │
  │  ✅ .NET Framework 4.0+      │
  │  ✅ WinRM configured         │
  │  ❌ No Ansible needed        │
  └──────────────────────────────┘

  Network Devices:
  ┌──────────────────────────────┐
  │  ✅ SSH or API access        │
  │  ❌ No Python needed         │
  │  ❌ No Ansible needed        │
  └──────────────────────────────┘
```

### 3. Inventory

The list of managed nodes, organized into groups:

```ini
# /etc/ansible/hosts (INI format)
[webservers]
web1.example.com
web2.example.com

[dbservers]
db1.example.com ansible_port=5432

[production:children]
webservers
dbservers
```

### 4. Modules

Reusable units of code that Ansible executes on managed nodes:

```
  MODULE CATEGORIES
  ═══════════════════════════════════════════════

  ┌─────────────────────────────────────────────┐
  │  System     │ user, group, service, cron    │
  │  Files      │ copy, file, template, fetch   │
  │  Packages   │ apt, yum, pip, npm            │
  │  Network    │ uri, get_url, nmcli           │
  │  Cloud      │ ec2, azure_rm, gcp_compute    │
  │  Database   │ mysql_db, postgresql_db       │
  │  Containers │ docker_container, k8s         │
  │  Commands   │ command, shell, raw, script   │
  │  Source Ctrl│ git, subversion               │
  └─────────────────────────────────────────────┘

  Total: 7,000+ modules available
```

### 5. Plugins

Extend Ansible's core functionality:

```
  PLUGIN TYPES
  ═══════════════════════════════════════════════

  ┌────────────────┬────────────────────────────┐
  │ Connection     │ SSH, WinRM, Docker, Local  │
  │ Callback       │ Output formatting, logging │
  │ Lookup         │ Read data from external    │
  │                │ sources (files, APIs)      │
  │ Filter         │ Transform data in Jinja2   │
  │ Inventory      │ Dynamic inventory sources  │
  │ Strategy       │ Execution strategies       │
  │                │ (linear, free, debug)      │
  │ Vars           │ Variable sources           │
  │ Cache          │ Fact caching               │
  └────────────────┴────────────────────────────┘
```

### 6. Playbooks

YAML files that define automation workflows:

```yaml
---
# site.yml - Main playbook
- name: Configure web servers
  hosts: webservers
  become: yes
  roles:
    - common
    - nginx
    - app-deploy

- name: Configure database servers
  hosts: dbservers
  become: yes
  roles:
    - common
    - postgresql
```

---

## Ansible Execution Flow

```
  STEP-BY-STEP EXECUTION FLOW
  ══════════════════════════════════════════════════════════════

  ┌─────────────────────────────────┐
  │ 1. PARSE                        │
  │    Read playbook YAML           │
  │    Load inventory               │
  │    Load ansible.cfg             │
  └──────────────┬──────────────────┘
                 │
                 ▼
  ┌─────────────────────────────────┐
  │ 2. GATHER FACTS (optional)      │
  │    Collect system info from     │
  │    managed nodes                │
  │    (OS, IP, memory, etc.)       │
  └──────────────┬──────────────────┘
                 │
                 ▼
  ┌─────────────────────────────────┐
  │ 3. COMPILE TASKS                │
  │    Resolve variables            │
  │    Evaluate conditionals        │
  │    Expand loops                 │
  └──────────────┬──────────────────┘
                 │
                 ▼
  ┌─────────────────────────────────┐
  │ 4. CONNECT                      │
  │    Establish SSH/WinRM          │
  │    connections to targets       │
  │    (parallel, up to 'forks')    │
  └──────────────┬──────────────────┘
                 │
                 ▼
  ┌─────────────────────────────────┐
  │ 5. EXECUTE                      │
  │    For each task:               │
  │    a. Generate module code      │
  │    b. Copy to remote temp dir   │
  │    c. Execute module            │
  │    d. Parse JSON result         │
  │    e. Clean up temp files       │
  └──────────────┬──────────────────┘
                 │
                 ▼
  ┌─────────────────────────────────┐
  │ 6. REPORT                       │
  │    Display results per host     │
  │    Run handlers if notified     │
  │    Show play recap summary      │
  └─────────────────────────────────┘
```

---

## Ansible Ecosystem

```
  ANSIBLE ECOSYSTEM
  ══════════════════════════════════════════════════════════════

  ┌───────────────────────────────────────────────────────────┐
  │                                                           │
  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │
  │  │   Ansible    │  │   Ansible   │  │   Ansible       │  │
  │  │   Core       │  │   Galaxy    │  │   Tower/AWX     │  │
  │  │             │  │             │  │   (AAP)         │  │
  │  │  CLI tools   │  │  Community  │  │                 │  │
  │  │  Engine      │  │  roles &    │  │  Web UI         │  │
  │  │  Modules     │  │  collections│  │  REST API       │  │
  │  │  Plugins     │  │  sharing    │  │  RBAC           │  │
  │  └─────────────┘  └─────────────┘  │  Scheduling     │  │
  │                                     │  Workflows      │  │
  │  ┌─────────────┐  ┌─────────────┐  │  Audit/Logging  │  │
  │  │  Ansible     │  │  Molecule   │  └─────────────────┘  │
  │  │  Lint        │  │             │                        │
  │  │             │  │  Testing    │  ┌─────────────────┐  │
  │  │  Code style  │  │  framework  │  │  Collections    │  │
  │  │  checker     │  │  for roles  │  │                 │  │
  │  └─────────────┘  └─────────────┘  │  Namespaced     │  │
  │                                     │  module bundles │  │
  │  ┌─────────────┐  ┌─────────────┐  │  (e.g., amazon  │  │
  │  │  Ansible     │  │  Ansible    │  │   .aws)         │  │
  │  │  Navigator   │  │  Builder    │  └─────────────────┘  │
  │  │             │  │             │                        │
  │  │  TUI for    │  │  Build EE   │                        │
  │  │  running    │  │  containers │                        │
  │  │  playbooks  │  │             │                        │
  │  └─────────────┘  └─────────────┘                        │
  │                                                           │
  └───────────────────────────────────────────────────────────┘
```

### Key Ecosystem Components

| Component | Purpose |
|-----------|---------|
| **Ansible Core** | CLI tools, execution engine, built-in modules |
| **Ansible Galaxy** | Community hub for sharing roles and collections |
| **Ansible Tower/AWX** | Enterprise web UI, API, RBAC, scheduling (AWX = open-source) |
| **Ansible Automation Platform (AAP)** | Red Hat's commercial Ansible product |
| **Ansible Lint** | Linting tool for playbooks and roles |
| **Molecule** | Testing framework for Ansible roles |
| **Ansible Navigator** | TUI tool for running and inspecting playbooks |
| **Ansible Builder** | Build execution environment container images |
| **Collections** | Namespaced bundles of modules, roles, and plugins |

---

## Key Ansible Commands

```bash
# Core commands
ansible              # Run ad-hoc commands
ansible-playbook     # Run playbooks
ansible-galaxy       # Manage roles and collections
ansible-vault        # Encrypt/decrypt sensitive data
ansible-config       # View/manage configuration
ansible-inventory    # View/manage inventory
ansible-doc          # View module documentation
ansible-pull         # Pull mode execution
ansible-console      # Interactive REPL
ansible-lint         # Lint playbooks (separate install)
```

---

## How Ansible Differs from Scripts

```
  SHELL SCRIPT vs ANSIBLE PLAYBOOK
  ══════════════════════════════════════════════════════════

  Shell Script:                      Ansible Playbook:
  ┌─────────────────────────┐       ┌─────────────────────────┐
  │ #!/bin/bash              │       │ ---                     │
  │                          │       │ - hosts: webservers     │
  │ apt-get update           │       │   become: yes           │
  │ apt-get install -y nginx │       │   tasks:                │
  │ systemctl start nginx    │       │   - apt:                │
  │ systemctl enable nginx   │       │       name: nginx       │
  │                          │       │       state: present    │
  │ # No error handling      │       │       update_cache: yes │
  │ # No idempotency         │       │   - service:            │
  │ # Manual SSH to each     │       │       name: nginx       │
  │ # No state tracking      │       │       state: started    │
  │ # No parallelism         │       │       enabled: yes      │
  └─────────────────────────┘       └─────────────────────────┘

  ❌ Script Problems:                ✅ Ansible Advantages:
  • Runs on one host at a time      • Runs on many hosts parallel
  • Not idempotent                   • Idempotent by design
  • No state awareness               • Checks before acting
  • Manual error handling            • Built-in error handling
  • Hard to maintain at scale        • Modular and reusable
  • OS-specific commands             • Cross-platform modules
```

---

## Real-World Scenarios

### Scenario 1: Application Deployment Pipeline
```
  Developer ──► Git Push ──► CI/CD ──► Ansible ──► Production
                                         │
                              ┌──────────┼──────────┐
                              │          │          │
                              ▼          ▼          ▼
                          Web Server  App Server  DB Server
                          (deploy     (deploy     (run
                           static)     app)        migrations)
```

### Scenario 2: Multi-Tier Application Setup
```yaml
# site.yml
---
- import_playbook: webservers.yml
- import_playbook: appservers.yml
- import_playbook: dbservers.yml
- import_playbook: monitoring.yml
```

### Scenario 3: Disaster Recovery
When a server fails, run the same playbook on a new instance:
```bash
# Replace failed web3 with new server
ansible-playbook site.yml --limit web3.example.com
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| "No module named ansible" | Ensure Python and pip are installed, reinstall Ansible |
| "Unreachable" error | Check SSH connectivity, keys, and network |
| "Permission denied" | Verify `become: yes` and sudo privileges |
| Slow playbook execution | Increase forks, use async, disable gather_facts if not needed |
| Module not found | Check Ansible version, install required collection |

---

## Summary Table

| Component | Description |
|-----------|-------------|
| Control Node | Machine where Ansible is installed and run from |
| Managed Node | Target machine being configured |
| Inventory | List of managed nodes and their groupings |
| Playbook | YAML file defining automation tasks |
| Module | Reusable code unit for specific actions |
| Plugin | Extensions for connections, callbacks, lookups, etc. |
| Role | Reusable collection of tasks, files, templates, and variables |
| Collection | Namespaced bundle of modules, roles, and plugins |
| Galaxy | Community hub for sharing roles and collections |
| Tower/AWX | Enterprise GUI, API, RBAC, and scheduling layer |

---

## Quick Revision Questions

1. **What are the minimum requirements for an Ansible control node?**
   > Linux/macOS (or WSL on Windows), Python 3.8+, Ansible installed, SSH client, and network access to managed nodes.

2. **What is needed on a Linux managed node for Ansible to work?**
   > Only Python (2.7+ or 3.5+) and an SSH server. No Ansible agent is needed.

3. **Describe the Ansible execution flow in 4 steps.**
   > Parse playbook/inventory → Gather facts from nodes → Execute modules via SSH → Report results.

4. **What is the difference between Ansible Core and Ansible Tower?**
   > Ansible Core is the open-source CLI tool and engine. Tower (AWX) adds a web UI, REST API, RBAC, scheduling, and audit logging.

5. **Name five types of Ansible plugins.**
   > Connection, callback, lookup, filter, inventory, strategy, vars, and cache (any five).

6. **How does Ansible execute a module on a remote node?**
   > It copies the module code to a temp directory on the remote node via SSH, executes it with arguments, collects the JSON result, and cleans up the temp files.

---

[← Previous: CM Tools Comparison](04-cm-tools-comparison.md) | [Next: Unit 2 - Installation →](../02-Ansible-Basics/01-installation.md)
