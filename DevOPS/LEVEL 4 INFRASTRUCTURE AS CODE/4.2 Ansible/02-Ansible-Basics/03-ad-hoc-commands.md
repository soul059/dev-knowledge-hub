# Chapter 2.3: Ad-hoc Commands

[← Previous: Inventory Basics](02-inventory-basics.md) | [Next: Modules Overview →](04-modules-overview.md)

---

## Overview

Ad-hoc commands are **one-liner Ansible commands** run from the terminal. They're perfect for quick tasks that don't warrant a full playbook — like checking server status, restarting a service, or copying a file.

---

## Ad-hoc Command Syntax

```
  COMMAND STRUCTURE
  ══════════════════════════════════════════════════════════

  ansible  <host-pattern>  -m <module>  -a "<arguments>"  [options]
     │          │              │              │                │
     │          │              │              │                └─ Extra options
     │          │              │              └─ Module arguments
     │          │              └─ Module to use
     │          └─ Which hosts to target
     └─ The ansible command

  Example:
  ansible  webservers  -m  apt  -a "name=nginx state=present"  -b
     │        │          │   │         │                         │
     │        │          │   │         └─ Module arguments       └─ become (sudo)
     │        │          │   └─ Module name
     │        │          └─ Module flag
     │        └─ Target group
     └─ Command
```

---

## Common Ad-hoc Commands

### System Information

```bash
# Ping all hosts (connectivity test)
ansible all -m ping

# Get system facts
ansible webservers -m setup

# Get specific facts
ansible webservers -m setup -a "filter=ansible_os_family"

# Check uptime
ansible all -m command -a "uptime"

# Check disk space
ansible all -m command -a "df -h"

# Check memory
ansible all -m command -a "free -m"
```

### Package Management

```bash
# Install a package (Debian/Ubuntu)
ansible webservers -m apt -a "name=nginx state=present" -b

# Install a package (RHEL/CentOS)
ansible webservers -m yum -a "name=httpd state=present" -b

# Remove a package
ansible webservers -m apt -a "name=nginx state=absent" -b

# Update all packages
ansible all -m apt -a "upgrade=yes update_cache=yes" -b
```

### Service Management

```bash
# Start a service
ansible webservers -m service -a "name=nginx state=started" -b

# Stop a service
ansible webservers -m service -a "name=nginx state=stopped" -b

# Restart a service
ansible webservers -m service -a "name=nginx state=restarted" -b

# Enable service at boot
ansible webservers -m service -a "name=nginx enabled=yes" -b
```

### File Operations

```bash
# Copy a file to remote hosts
ansible webservers -m copy -a "src=/tmp/config.conf dest=/etc/app/config.conf" -b

# Create a directory
ansible all -m file -a "path=/opt/myapp state=directory mode=0755" -b

# Change file permissions
ansible all -m file -a "path=/etc/app/config.conf mode=0644 owner=root" -b

# Fetch a file from remote
ansible web1 -m fetch -a "src=/var/log/syslog dest=/tmp/logs/"
```

### User Management

```bash
# Create a user
ansible all -m user -a "name=deploy state=present shell=/bin/bash" -b

# Add user to groups
ansible all -m user -a "name=deploy groups=sudo append=yes" -b

# Remove a user
ansible all -m user -a "name=olduser state=absent remove=yes" -b

# Add SSH key for a user
ansible all -m authorized_key -a "user=deploy key='ssh-rsa AAAA...'" -b
```

### Running Commands

```bash
# command module (no shell features)
ansible all -m command -a "whoami"

# shell module (supports pipes, redirects)
ansible all -m shell -a "cat /etc/passwd | grep deploy"

# raw module (no Python required on target)
ansible all -m raw -a "apt-get install -y python3"

# script module (run a local script on remote)
ansible all -m script -a "/local/path/setup.sh"
```

---

## Ad-hoc Command Options

```
  COMMON OPTIONS
  ══════════════════════════════════════════════════════════

  ┌──────────────────────┬─────────────────────────────────┐
  │ Option               │ Description                     │
  ├──────────────────────┼─────────────────────────────────┤
  │ -i inventory_file    │ Specify inventory file          │
  │ -m module_name       │ Module to run                   │
  │ -a "arguments"       │ Module arguments                │
  │ -b (--become)        │ Run with sudo/root              │
  │ -K (--ask-become-pass│ Prompt for sudo password        │
  │ -u username          │ SSH as this user                │
  │ -k (--ask-pass)      │ Prompt for SSH password         │
  │ --key-file path      │ SSH private key                 │
  │ -f 10 (--forks)      │ Number of parallel workers      │
  │ -v / -vvv / -vvvv    │ Verbosity (more v = more detail)│
  │ --check              │ Dry run (no changes)            │
  │ --diff               │ Show file differences           │
  │ -o (--one-line)      │ Condense output                 │
  │ --limit host         │ Limit to specific host          │
  └──────────────────────┴─────────────────────────────────┘
```

---

## Understanding Output

```bash
$ ansible webservers -m ping

# SUCCESS output:
web1.example.com | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,      # ◄── No changes were made
    "ping": "pong"         # ◄── Module response
}

# FAILURE output:
web3.example.com | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh",
    "unreachable": true
}

# CHANGED output:
web1.example.com | CHANGED => {
    "changed": true,       # ◄── Something was modified
    "msg": "..."
}
```

```
  OUTPUT COLORS
  ═════════════════════════

  GREEN   = ok (no change)
  YELLOW  = changed
  RED     = failed/unreachable
  BLUE    = skipped
```

---

## Practical Examples

### Quick Server Audit
```bash
# Check which servers are up
ansible all -m ping -o

# Check OS version on all servers
ansible all -m shell -a "cat /etc/os-release | head -2"

# Check running services
ansible webservers -m shell -a "systemctl list-units --type=service --state=running | head -20"

# Check disk usage
ansible all -m shell -a "df -h / | tail -1" -o
```

### Emergency Actions
```bash
# Restart a crashed service across all web servers
ansible webservers -m service -a "name=nginx state=restarted" -b

# Kill a runaway process
ansible app_servers -m shell -a "pkill -f 'memory_leak_process'" -b

# Apply security patch immediately
ansible all -m apt -a "name=openssl state=latest" -b
```

---

## Real-World Scenarios

### Scenario 1: Checking Compliance
```bash
# Verify NTP is installed on all servers
ansible all -m shell -a "dpkg -l | grep ntp" -o

# Verify specific user exists
ansible all -m shell -a "id deploy" -o

# Check firewall status
ansible all -m shell -a "ufw status" -b -o
```

### Scenario 2: Quick File Distribution
```bash
# Push new SSL certificate to all web servers
ansible webservers -m copy -a "src=./cert.pem dest=/etc/ssl/certs/app.pem mode=0644" -b

# Push and restart
ansible webservers -m copy -a "src=./cert.pem dest=/etc/ssl/certs/app.pem" -b
ansible webservers -m service -a "name=nginx state=reloaded" -b
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| `MODULE FAILURE` | Check module arguments with `ansible-doc <module>` |
| `Permission denied` | Add `-b` for sudo or `-K` to prompt for password |
| `Host unreachable` | Check SSH connectivity, keys, and firewall |
| Command hangs | Set timeout: `-T 30` or check for interactive prompts |
| Wrong output | Increase verbosity: `-vvv` for detailed debugging |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| Ad-hoc command | Single-line Ansible command for quick tasks |
| `-m` | Specify module to use |
| `-a` | Pass arguments to the module |
| `-b` | Execute with privilege escalation (sudo) |
| `-i` | Specify inventory file |
| `ping` module | Test connectivity (not ICMP, Ansible-level) |
| `command` module | Run commands without shell features |
| `shell` module | Run commands with full shell (pipes, redirects) |
| `-o` | One-line condensed output |
| `-vvv` | Verbose output for debugging |

---

## Quick Revision Questions

1. **What is the difference between `command` and `shell` modules?**
   > `command` executes without a shell (no pipes, redirects). `shell` runs through `/bin/sh` and supports shell features.

2. **How do you run an ad-hoc command with sudo privileges?**
   > Add the `-b` (or `--become`) flag. Use `-K` to be prompted for the sudo password.

3. **What does the `ping` module actually test?**
   > It tests Ansible connectivity (SSH + Python), not ICMP ping. A successful response returns `"pong"`.

4. **How do you increase verbosity for debugging?**
   > Add `-v`, `-vv`, `-vvv`, or `-vvvv` flags, with more v's giving more detail.

5. **When should you use ad-hoc commands vs playbooks?**
   > Ad-hoc for quick, one-off tasks (check status, restart service). Playbooks for repeatable, multi-step automation.

6. **What does the `-o` flag do?**
   > It condenses the output to a single line per host for easier reading.

---

[← Previous: Inventory Basics](02-inventory-basics.md) | [Next: Modules Overview →](04-modules-overview.md)
