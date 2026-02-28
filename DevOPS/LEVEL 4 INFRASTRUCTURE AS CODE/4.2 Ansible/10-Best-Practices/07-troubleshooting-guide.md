# Chapter 10.7: Troubleshooting Guide

[← Previous: Scaling Ansible](06-scaling-ansible.md) | [Back to README →](../README.md)

---

## Overview

This final chapter is a **comprehensive troubleshooting reference** for Ansible. It covers common errors, debugging techniques, verbose output, and a categorised quick-reference for resolving the most frequent issues.

---

## Debugging Techniques

### Verbosity Levels

```bash
# Increasing verbosity
ansible-playbook site.yml -v       # Show task results
ansible-playbook site.yml -vv      # Show task input parameters
ansible-playbook site.yml -vvv     # Show SSH connection details
ansible-playbook site.yml -vvvv    # Show SSH, connection plugins, full debug
```

```
  VERBOSITY LEVELS
  ══════════════════════════════════════════════════════════

  -v      Task results and changed status
  -vv     Task input parameters and config
  -vvv    SSH connection commands and paths
  -vvvv   Full debug (everything including plugin internals)
```

### Debug Module

```yaml
- name: Print variable value
  debug:
    var: my_variable

- name: Print formatted message
  debug:
    msg: "Host {{ inventory_hostname }} has IP {{ ansible_default_ipv4.address }}"

- name: Print all facts
  debug:
    var: ansible_facts

- name: Conditional debug
  debug:
    msg: "This host is a web server"
  when: "'webservers' in group_names"
```

### Register and Inspect

```yaml
- name: Run command and inspect output
  command: systemctl status nginx
  register: nginx_status
  ignore_errors: true

- name: Show full result
  debug:
    var: nginx_status

- name: Show specific fields
  debug:
    msg: |
      RC: {{ nginx_status.rc }}
      STDOUT: {{ nginx_status.stdout }}
      STDERR: {{ nginx_status.stderr }}
```

---

## Common Errors and Solutions

### SSH Connection Errors

```
  ERROR: "unreachable" / "Permission denied"
  ══════════════════════════════════════════════════════════

  ┌─────────────────────────────────────────┐
  │  Diagnosis Steps:                       │
  │                                         │
  │  1. Test SSH manually:                  │
  │     ssh -i ~/.ssh/key user@host         │
  │                                         │
  │  2. Check SSH key permissions:          │
  │     chmod 600 ~/.ssh/id_rsa             │
  │                                         │
  │  3. Verify user exists on target:       │
  │     ssh user@host whoami                │
  │                                         │
  │  4. Check firewall (port 22):           │
  │     nc -zv host 22                      │
  │                                         │
  │  5. Check SSH config:                   │
  │     ssh -vvv user@host                  │
  └─────────────────────────────────────────┘
```

| Error | Cause | Fix |
|-------|-------|-----|
| `Permission denied (publickey)` | Wrong SSH key or user | Check `remote_user` and `private_key_file` |
| `Host key verification failed` | Unknown host key | Add to `known_hosts` or set `host_key_checking = False` |
| `Connection timed out` | Firewall or wrong IP | Verify IP and port 22 accessibility |
| `Connection refused` | SSH not running | Start sshd on managed node |
| `Too many authentication failures` | SSH agent sending too many keys | Use `IdentitiesOnly=yes` in ssh config |

### Privilege Escalation Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `Missing sudo password` | `become_pass` not set | Add `-K` flag or set `ansible_become_password` |
| `sudo: a password is required` | NOPASSWD not configured | Add `NOPASSWD:` to sudoers |
| `sudo: user not in sudoers` | User lacks sudo rights | Add user to sudo group |
| `requiretty` error with pipelining | sudoers requires tty | Comment out `Defaults requiretty` |

### Module Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `No module named 'apt'` | Wrong module for OS | Use `yum`/`dnf` for RedHat family |
| `Module not found` | Typo or missing collection | Check FQCN: `ansible.builtin.apt` |
| `Could not find or access` | File not found on controller | Verify `src:` path is correct |
| `Failed to lock apt` | Another apt process running | Wait or kill existing apt process |
| `msg: No package matching` | Package name wrong | Check exact package name for the distro |

### Variable Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `undefined variable` | Variable not defined or typo | Check variable name, scope, and precedence |
| `'dict' has no attribute` | Accessing missing key | Use `var.get('key', default)` or `var.key \| default('')` |
| `recursive loop` | Variable references itself | Break circular reference |
| `Vault password not provided` | Encrypted vars without vault pass | Add `--vault-password-file` |

### Playbook Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `Syntax Error` | Invalid YAML | Run `yamllint` and `--syntax-check` |
| `ERROR! 'task' is not a valid attribute` | Wrong indentation | Fix YAML indentation |
| `duplicate key` | Same key twice in dict | Remove duplicate |
| `could not find role` | Role not installed or wrong path | Run `ansible-galaxy install` or fix `roles_path` |

---

## Diagnostic Commands

```bash
# Verify Ansible installation
ansible --version

# Test host connectivity
ansible all -i inventory -m ping

# List hosts in a group
ansible webservers -i inventory --list-hosts

# Show all facts for a host
ansible host1 -i inventory -m setup

# Show specific fact
ansible host1 -m setup -a "filter=ansible_distribution*"

# Test ad-hoc command
ansible host1 -m command -a "uptime"

# Check inventory parsing
ansible-inventory -i inventory --list
ansible-inventory -i inventory --graph

# Dry run
ansible-playbook site.yml --check --diff

# List tasks in a playbook
ansible-playbook site.yml --list-tasks

# List tags
ansible-playbook site.yml --list-tags

# List hosts targeted
ansible-playbook site.yml --list-hosts

# Step through tasks one by one
ansible-playbook site.yml --step
```

---

## Debugging Workflow

```
  TROUBLESHOOTING FLOWCHART
  ══════════════════════════════════════════════════════════

  Error occurred
       │
       ▼
  ┌─────────────────┐     YES    ┌──────────────────┐
  │ Connection error?│──────────▶│ Test SSH manually │
  └────────┬────────┘           │ ssh user@host     │
           │ NO                  └──────────────────┘
           ▼
  ┌─────────────────┐     YES    ┌──────────────────┐
  │ Permission error?│──────────▶│ Check sudo config │
  └────────┬────────┘           │ visudo            │
           │ NO                  └──────────────────┘
           ▼
  ┌─────────────────┐     YES    ┌──────────────────┐
  │ Module error?   │──────────▶│ Check module docs │
  └────────┬────────┘           │ ansible-doc mod   │
           │ NO                  └──────────────────┘
           ▼
  ┌─────────────────┐     YES    ┌──────────────────┐
  │ Variable error? │──────────▶│ Debug with -vvv   │
  └────────┬────────┘           │ check precedence  │
           │ NO                  └──────────────────┘
           ▼
  ┌─────────────────┐     YES    ┌──────────────────┐
  │ Syntax error?   │──────────▶│ yamllint + lint   │
  └────────┬────────┘           └──────────────────┘
           │ NO
           ▼
  Run with -vvvv and read full output
```

---

## Environment Debugging

```bash
# Check Ansible configuration sources
ansible-config dump --only-changed

# View active configuration
ansible-config view

# List all config options
ansible-config list

# Check which config file is being used
ansible --version
# config file = /path/to/ansible.cfg

# Config file precedence
# 1. ANSIBLE_CONFIG env variable
# 2. ./ansible.cfg (current directory)
# 3. ~/.ansible.cfg (home directory)
# 4. /etc/ansible/ansible.cfg (system)
```

---

## Performance Debugging

```bash
# Enable task profiling
export ANSIBLE_CALLBACKS_ENABLED=timer,profile_tasks

# Output example:
# TASK [Install packages] ─────────────── 45.32s
# TASK [Configure nginx] ──────────────── 3.12s
# TASK [Gather facts] ─────────────────── 2.50s
```

```yaml
# Identify slow tasks in a playbook
- name: Time-sensitive task
  command: /opt/slow-script.sh
  register: result

- name: Report execution time
  debug:
    msg: "Task took {{ result.delta }} seconds"
```

---

## Common Patterns for Resilience

```yaml
# Retry on failure
- name: Wait for host to come back
  wait_for_connection:
    delay: 10
    timeout: 300

# Retry a flaky task
- name: Download with retry
  get_url:
    url: "https://downloads.example.com/package.tar.gz"
    dest: /tmp/package.tar.gz
  retries: 3
  delay: 5
  register: download_result
  until: download_result is success

# Ignore non-critical failures
- name: Optional cleanup
  file:
    path: /tmp/old-cache
    state: absent
  ignore_errors: true

# Fail with a helpful message
- name: Validate prerequisites
  assert:
    that:
      - ansible_distribution == "Ubuntu"
      - ansible_distribution_major_version | int >= 20
    fail_msg: "This playbook requires Ubuntu 20.04+ (found {{ ansible_distribution }} {{ ansible_distribution_version }})"
    success_msg: "OS check passed"
```

---

## Quick Reference: Error → Solution

```
  ══════════════════════════════════════════════════════════
  ERROR                          QUICK FIX
  ══════════════════════════════════════════════════════════
  "unreachable"                  Test SSH: ssh user@host
  "Permission denied"            Check SSH key & remote_user
  "Missing sudo password"        Add -K or NOPASSWD in sudoers
  "undefined variable"           Check var name & scope (-vvv)
  "No module named"              Check OS family, use FQCN
  "Syntax Error"                 Run yamllint & --syntax-check
  "Vault password required"      Add --vault-password-file
  "role not found"               ansible-galaxy install / roles_path
  "duplicate key"                Fix YAML (remove duplicate)
  "timeout"                      Check firewall / increase timeout
  "changed" on every run         Add changed_when / use modules
  "could not find or access"     Verify src path on controller
  ══════════════════════════════════════════════════════════
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Can't find which config file is active | Run `ansible --version` to see `config file =` line |
| Variables not what you expect | Use `debug: var=my_var` and check variable precedence |
| Playbook changes every run | Ensure idempotency — use modules, add `changed_when` |
| Task succeeds but app broken | Add smoke tests / post-deployment verify tasks |
| Want to test one task | Use `--start-at-task "task name"` or tags |
| Need to skip a failing task | Use `--skip-tags` or `ignore_errors: true` temporarily |

---

## Summary Table

| Technique | Command / Setting |
|-----------|-------------------|
| Verbose output | `-v` through `-vvvv` |
| Debug variable | `debug: var=my_var` |
| Register output | `register: result` + `debug: var=result` |
| Syntax check | `--syntax-check` |
| Dry run | `--check --diff` |
| List tasks | `--list-tasks` |
| Step mode | `--step` |
| Config check | `ansible-config dump --only-changed` |
| Host connectivity | `ansible all -m ping` |
| Fact inspection | `ansible host -m setup` |
| Inventory check | `ansible-inventory --graph` |
| Task profiling | `callbacks_enabled = profile_tasks` |

---

## Quick Revision Questions

1. **What are the four verbosity levels and what does each show?**
   > `-v`: task results; `-vv`: task input parameters; `-vvv`: SSH connection details; `-vvvv`: full debug including plugin internals.

2. **How do you test connectivity to all hosts?**
   > Run `ansible all -i inventory -m ping` — this tests SSH and Python availability on each host.

3. **How can you see all facts for a specific host?**
   > Run `ansible hostname -m setup` or use `debug: var=ansible_facts` in a playbook.

4. **What is the `--check --diff` flag combination used for?**
   > `--check` runs in dry-run mode (no changes), and `--diff` shows what would change, allowing you to preview modifications safely.

5. **How do you identify which ansible.cfg file is being used?**
   > Run `ansible --version` which shows the active config file path, or use `ansible-config dump --only-changed` to see all active non-default settings.

6. **How do you handle a flaky task that occasionally fails?**
   > Use `retries`, `delay`, and `until` to retry the task, e.g., `retries: 3`, `delay: 5`, `until: result is success`.

---

[← Previous: Scaling Ansible](06-scaling-ansible.md) | [Back to README →](../README.md)
