# Chapter 2.5: ansible.cfg Configuration

[← Previous: Modules Overview](04-modules-overview.md) | [Next: SSH and Authentication →](06-ssh-and-authentication.md)

---

## Overview

The `ansible.cfg` file controls Ansible's default behavior — from SSH settings to output formatting, parallelism, and privilege escalation. Understanding this file is key to efficient Ansible workflows.

---

## Configuration File Precedence

```
  ANSIBLE.CFG LOOKUP ORDER (First Found Wins)
  ══════════════════════════════════════════════════════════

  1. ANSIBLE_CONFIG environment variable
     │  export ANSIBLE_CONFIG=/path/to/ansible.cfg
     │
  2. ./ansible.cfg  (current directory) ◄── Most common
     │
  3. ~/.ansible.cfg (home directory)
     │
  4. /etc/ansible/ansible.cfg (system-wide)

  ┌─────────────────────────────────────────────────┐
  │  ⚠️  Security Note:                             │
  │  Ansible ignores ansible.cfg in current dir     │
  │  if the directory is world-writable.            │
  │  This prevents privilege escalation attacks.    │
  └─────────────────────────────────────────────────┘
```

---

## Complete ansible.cfg Example

```ini
# ansible.cfg - Project-level configuration

[defaults]
# Inventory
inventory           = ./inventory/hosts.ini
# Remote user
remote_user         = deploy
# Disable host key checking (lab/dev only!)
host_key_checking   = False
# Number of parallel processes
forks               = 20
# SSH timeout
timeout             = 30
# Retry files (disable for cleanliness)
retry_files_enabled = False
# Callback plugin for output formatting
stdout_callback     = yaml
# Gathering facts setting
gathering           = smart
# Fact caching
fact_caching        = jsonfile
fact_caching_connection = /tmp/ansible_facts_cache
fact_caching_timeout    = 86400
# Roles path
roles_path          = ./roles:~/.ansible/roles
# Module search path
library             = ./library
# Log file
log_path            = ./ansible.log
# Vault password file
vault_password_file = ~/.vault_pass
# Interpreter discovery
interpreter_python  = auto_silent
# Display skipped hosts
display_skipped_hosts = False
# Disable deprecation warnings
deprecation_warnings  = False
# Collections path
collections_path    = ./collections:~/.ansible/collections

[privilege_escalation]
become              = True
become_method       = sudo
become_user         = root
become_ask_pass     = False

[ssh_connection]
# SSH arguments for performance
ssh_args            = -C -o ControlMaster=auto -o ControlPersist=60s
# Pipelining (faster, requires requiretty disabled)
pipelining          = True
# SCP or SFTP
transfer_method     = smart
# Control path for SSH multiplexing
control_path        = %(directory)s/%%h-%%r-%%p

[paramiko_connection]
record_host_keys    = False

[colors]
changed             = yellow
ok                  = green
error               = red
skip                = cyan
```

---

## Key Configuration Sections

### [defaults] Section

```
  [defaults] KEY SETTINGS
  ══════════════════════════════════════════════════════════

  ┌────────────────────┬──────────────────────────────────┐
  │ Setting            │ Impact                           │
  ├────────────────────┼──────────────────────────────────┤
  │ inventory          │ Where to find hosts              │
  │ remote_user        │ SSH user for connections         │
  │ forks              │ Parallelism (default: 5)         │
  │ timeout            │ SSH connection timeout           │
  │ host_key_checking  │ SSH host key verification        │
  │ gathering          │ Fact collection behavior         │
  │ roles_path         │ Where to find roles              │
  │ stdout_callback    │ Output format                    │
  │ log_path           │ Write logs to file               │
  │ vault_password_file│ Auto-decrypt vault files         │
  └────────────────────┴──────────────────────────────────┘
```

### Gathering Modes

```yaml
# gathering options:
# implicit  - gather facts for every play (default)
# explicit  - only gather when 'gather_facts: true'
# smart     - gather once and cache, skip if cached

gathering = smart    # Best for performance
```

### Forks Setting

```
  FORKS = NUMBER OF PARALLEL CONNECTIONS
  ══════════════════════════════════════════════

  forks = 5 (default):
  ┌──────┐  Tasks    ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐
  │Ansible│─────────►│ H1 │ │ H2 │ │ H3 │ │ H4 │ │ H5 │ → wait →
  └──────┘  batch1   └────┘ └────┘ └────┘ └────┘ └────┘
                     ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐
            ────────►│ H6 │ │ H7 │ │ H8 │ │ H9 │ │H10│
             batch2  └────┘ └────┘ └────┘ └────┘ └────┘

  forks = 20:
  ┌──────┐  Tasks    ┌───┐┌───┐┌───┐┌───┐...┌────┐
  │Ansible│─────────►│H1 ││H2 ││H3 ││H4 │  │H20│ → done!
  └──────┘  batch1   └───┘└───┘└───┘└───┘...└────┘

  Higher forks = faster for large inventories
  But uses more memory/CPU on control node
```

### [ssh_connection] Performance

```
  SSH CONNECTION OPTIMIZATION
  ══════════════════════════════════════════════

  WITHOUT optimization:
  Task 1: Connect → Auth → Execute → Disconnect
  Task 2: Connect → Auth → Execute → Disconnect
  Task 3: Connect → Auth → Execute → Disconnect
  (3 full SSH handshakes)

  WITH ControlPersist + Pipelining:
  Task 1: Connect → Auth → Execute
  Task 2: ──────────────── Execute  (reuse connection)
  Task 3: ──────────────── Execute  (reuse connection)
  (1 SSH handshake, persistent connection)

  ssh_args = -C -o ControlMaster=auto -o ControlPersist=60s
  │          │       │                       │
  │          │       │                       └── Keep alive 60s
  │          │       └── Multiplex SSH connections
  │          └── Enable compression
  └── SSH arguments

  pipelining = True
  └── Send module via pipe instead of copying temp file
      (Faster, but requires !requiretty in sudoers)
```

---

## Environment Variable Overrides

```bash
# Override any setting via environment variables
export ANSIBLE_CONFIG=/custom/path/ansible.cfg
export ANSIBLE_INVENTORY=/path/to/inventory
export ANSIBLE_REMOTE_USER=admin
export ANSIBLE_FORKS=30
export ANSIBLE_HOST_KEY_CHECKING=False
export ANSIBLE_STDOUT_CALLBACK=yaml
export ANSIBLE_VAULT_PASSWORD_FILE=~/.vault_pass
export ANSIBLE_LOG_PATH=~/ansible.log

# Naming convention: ANSIBLE_<SECTION>_<KEY> or ANSIBLE_<KEY>
```

---

## Viewing Configuration

```bash
# Show current configuration
ansible-config dump

# Show only changed settings
ansible-config dump --only-changed

# Show configuration file being used
ansible --version
# Output includes: config file = /path/to/ansible.cfg

# List all configuration options
ansible-config list

# View specific setting
ansible-config dump | grep FORKS
```

---

## Project-level ansible.cfg Templates

### Development Environment
```ini
[defaults]
inventory          = ./inventory/dev.ini
remote_user        = vagrant
host_key_checking  = False
forks              = 5
stdout_callback    = debug
deprecation_warnings = True

[privilege_escalation]
become             = True
become_method      = sudo
become_ask_pass    = False
```

### Production Environment
```ini
[defaults]
inventory          = ./inventory/prod.ini
remote_user        = deploy
host_key_checking  = True
forks              = 20
stdout_callback    = yaml
log_path           = /var/log/ansible/playbook.log
vault_password_file = /etc/ansible/.vault_pass
gathering          = smart
fact_caching       = jsonfile
fact_caching_connection = /tmp/ansible_cache

[privilege_escalation]
become             = True
become_method      = sudo
become_ask_pass    = False

[ssh_connection]
ssh_args           = -C -o ControlMaster=auto -o ControlPersist=300s
pipelining         = True
```

---

## Real-World Scenarios

### Scenario 1: Slow Playbook on 100 Hosts
```ini
# Problem: Takes too long with default forks=5
# Solution:
[defaults]
forks = 50

[ssh_connection]
ssh_args = -C -o ControlMaster=auto -o ControlPersist=120s
pipelining = True
```

### Scenario 2: Different Settings Per Project
```
project-a/
├── ansible.cfg      ← forks=5, user=vagrant
├── inventory/
└── playbooks/

project-b/
├── ansible.cfg      ← forks=30, user=deploy
├── inventory/
└── playbooks/
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Settings not being applied | Check `ansible --version` for which config file is loaded |
| Pipelining not working | Disable `requiretty` in sudoers on managed nodes |
| SSH timeout | Increase `timeout` setting, check network |
| Config ignored in directory | Check directory permissions (not world-writable) |
| Vault password prompts | Set `vault_password_file` in ansible.cfg |

---

## Summary Table

| Setting | Section | Default | Purpose |
|---------|---------|---------|---------|
| `inventory` | defaults | /etc/ansible/hosts | Inventory file path |
| `remote_user` | defaults | current user | SSH login user |
| `forks` | defaults | 5 | Parallel connections |
| `host_key_checking` | defaults | True | SSH key verification |
| `gathering` | defaults | implicit | Fact collection mode |
| `become` | privilege_escalation | False | Enable sudo |
| `pipelining` | ssh_connection | False | Faster SSH (pipe mode) |
| `ssh_args` | ssh_connection | (none) | Custom SSH arguments |

---

## Quick Revision Questions

1. **What is the order of ansible.cfg lookup precedence?**
   > `ANSIBLE_CONFIG` env var → `./ansible.cfg` → `~/.ansible.cfg` → `/etc/ansible/ansible.cfg`

2. **What does the `forks` setting control?**
   > The number of parallel SSH connections Ansible uses to execute tasks. Default is 5.

3. **How does `pipelining = True` improve performance?**
   > It sends modules via a pipe instead of copying temp files to remote nodes, reducing SSH overhead.

4. **What are the three `gathering` modes?**
   > `implicit` (always gather), `explicit` (only when specified), `smart` (gather once and cache).

5. **How can you override ansible.cfg settings without editing the file?**
   > Use environment variables like `ANSIBLE_FORKS=20` or command-line arguments.

6. **Why might ansible.cfg in the current directory be ignored?**
   > Ansible ignores it if the directory is world-writable, as a security measure against privilege escalation.

---

[← Previous: Modules Overview](04-modules-overview.md) | [Next: SSH and Authentication →](06-ssh-and-authentication.md)
