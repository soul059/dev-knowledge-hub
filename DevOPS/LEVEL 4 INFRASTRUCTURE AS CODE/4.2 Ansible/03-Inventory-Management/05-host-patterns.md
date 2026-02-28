# Chapter 3.5: Host Patterns

[← Previous: Groups and Group Variables](04-groups-and-group-variables.md) | [Next: Inventory Plugins →](06-inventory-plugins.md)

---

## Overview

Host patterns determine **which hosts** from your inventory a play or ad-hoc command targets. Ansible supports simple group names, wildcards, intersections, exclusions, and regex patterns.

---

## Pattern Types

| Pattern | Example | Matches |
|---------|---------|---------|
| All hosts | `all` or `*` | Every host in inventory |
| Group name | `webservers` | All hosts in webservers group |
| Single host | `web1.example.com` | Just that host |
| Wildcard | `*.example.com` | All matching hosts |
| Union (OR) | `webservers:dbservers` | Both groups combined |
| Intersection (AND) | `webservers:&production` | Hosts in BOTH groups |
| Exclusion (NOT) | `all:!dbservers` | All except dbservers |
| Regex | `~web[0-9]+\.example\.com` | Regex matched hosts |
| Index/slice | `webservers[0]` | First host in group |
| Range | `webservers[0:2]` | First three hosts |

---

## Pattern Examples

```bash
# Target all hosts
ansible all -m ping
ansible '*' -m ping

# Target a specific group
ansible webservers -m ping

# Target a single host
ansible web1.example.com -m ping

# Wildcard matching
ansible '*.example.com' -m ping
ansible 'web*' -m ping

# Union - hosts in webservers OR dbservers
ansible 'webservers:dbservers' -m ping

# Intersection - hosts in webservers AND production
ansible 'webservers:&production' -m ping

# Exclusion - all hosts EXCEPT dbservers
ansible 'all:!dbservers' -m ping

# Complex patterns
# Production web servers, excluding those in maintenance
ansible 'webservers:&production:!maintenance' -m ping

# All servers in datacenter1 OR datacenter2, but NOT staging
ansible 'datacenter1:datacenter2:!staging' -m ping

# Regex pattern (prefix with ~)
ansible '~web[0-9]+\.example\.com' -m ping

# Index-based selection
ansible 'webservers[0]' -m ping       # First host
ansible 'webservers[-1]' -m ping      # Last host
ansible 'webservers[0:3]' -m ping     # First 3 hosts
ansible 'webservers[2:]' -m ping      # Third host onward
```

---

## Patterns in Playbooks

```yaml
# Direct group
- hosts: webservers
  tasks: [...]

# Multiple groups (union)
- hosts: webservers:appservers
  tasks: [...]

# Intersection
- hosts: webservers:&production
  tasks: [...]

# Exclusion
- hosts: all:!monitoring_servers
  tasks: [...]

# Complex
- hosts: "webservers:&production:!maintenance"
  tasks: [...]

# Wildcard
- hosts: "*.us-east-1.compute.amazonaws.com"
  tasks: [...]
```

---

## Limiting at Runtime

```bash
# Run playbook on all hosts, but limit to one
ansible-playbook site.yml --limit web1.example.com

# Limit to a group
ansible-playbook site.yml --limit webservers

# Limit to multiple hosts
ansible-playbook site.yml --limit "web1.example.com,web2.example.com"

# Limit using a pattern
ansible-playbook site.yml --limit 'webservers:&production'

# Limit from a file
ansible-playbook site.yml --limit @failed_hosts.txt

# Retry failed hosts from last run
ansible-playbook site.yml --limit @site.retry
```

---

## Real-World Scenarios

### Scenario: Rolling Deployment
```bash
# Deploy to first web server only (canary)
ansible-playbook deploy.yml --limit webservers[0]

# If canary succeeds, deploy to remaining
ansible-playbook deploy.yml --limit 'webservers[1:]'
```

### Scenario: Target by Environment + Role
```yaml
# Only production database servers
- hosts: dbservers:&production
  tasks:
    - name: Run database backup
      shell: pg_dump mydb > /backups/mydb.sql
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| "No hosts matched" | Check pattern syntax and inventory content |
| Pattern with `!` not working in shell | Quote the pattern: `ansible 'all:!group' -m ping` |
| Wrong hosts matched | Use `--list-hosts` to preview: `ansible 'pattern' --list-hosts` |
| Regex not matching | Ensure prefix with `~` and proper escaping |

---

## Summary Table

| Pattern | Syntax | Description |
|---------|--------|-------------|
| All | `all` | Every host |
| Group | `webservers` | All hosts in group |
| Union | `A:B` | Hosts in A or B |
| Intersection | `A:&B` | Hosts in both A and B |
| Exclusion | `A:!B` | Hosts in A but not B |
| Wildcard | `web*` | Glob matching |
| Regex | `~pattern` | Regular expression |
| Index | `group[0]` | Specific position |
| Limit | `--limit` | Runtime restriction |

---

## Quick Revision Questions

1. **How do you target hosts that are in BOTH webservers and production?**
   > Use intersection: `webservers:&production`

2. **How do you exclude a group from a pattern?**
   > Use `!`: `all:!maintenance` targets all hosts except those in maintenance.

3. **What does the `~` prefix signify in a host pattern?**
   > It indicates a regex pattern: `~web[0-9]+` matches using regular expressions.

4. **How do you run a playbook on just the first host in a group?**
   > Use index notation: `--limit webservers[0]`

5. **How to preview which hosts a pattern matches without running tasks?**
   > Use `ansible <pattern> --list-hosts` to see matched hosts.

6. **How do you re-run a playbook on only the hosts that failed last time?**
   > Use `--limit @playbook.retry` which reads the retry file from the previous failed run.

---

[← Previous: Groups and Group Variables](04-groups-and-group-variables.md) | [Next: Inventory Plugins →](06-inventory-plugins.md)
