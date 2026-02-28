# Chapter 10.6: Scaling Ansible

[← Previous: CI/CD Integration](05-ci-cd-integration.md) | [Next: Troubleshooting Guide →](07-troubleshooting-guide.md)

---

## Overview

As infrastructure grows from tens to hundreds or thousands of hosts, Ansible's default settings can become bottlenecks. This chapter covers strategies for scaling Ansible — from **execution architecture** and **pull mode** to **inventory management at scale** and **Ansible Automation Platform**.

---

## Scaling Challenges

```
  SCALING DIMENSIONS
  ══════════════════════════════════════════════════════════

  ┌─────────────────────────────────────────────────────┐
  │                                                     │
  │  Hosts ──────────────▶  10 → 100 → 1000 → 10,000+  │
  │  Playbooks ──────────▶  Simple → Complex multi-play │
  │  Teams ──────────────▶  1 → Multiple → Enterprise   │
  │  Environments ───────▶  Dev → Dev/Stg/Prod/DR       │
  │  Frequency ──────────▶  Weekly → Daily → Continuous │
  │                                                     │
  └─────────────────────────────────────────────────────┘
```

---

## 1. Execution Performance

### Increase Parallelism

```ini
# ansible.cfg
[defaults]
forks = 50              # Default is 5
strategy = free          # No inter-host waiting
```

### Enable Pipelining

```ini
[ssh_connection]
pipelining = True
ssh_args = -o ControlMaster=auto -o ControlPersist=120s
```

### Fact Caching with Redis

```ini
[defaults]
gathering = smart
fact_caching = redis
fact_caching_connection = redis.example.com:6379:0
fact_caching_timeout = 7200     # 2 hours
```

```
  REDIS FACT CACHE (MULTI-CONTROLLER)
  ══════════════════════════════════════════════════════════

  Controller A ──┐                ┌── Controller B
                 ▼                ▼
           ┌──────────────────────────┐
           │       Redis Server       │
           │                          │
           │  host1: {facts...}       │  Shared cache
           │  host2: {facts...}       │  across all
           │  host3: {facts...}       │  controllers
           └──────────────────────────┘
```

---

## 2. Pull Mode with ansible-pull

```
  PUSH MODE (Default)              PULL MODE
  ═══════════════════              ══════════════════

  ┌──────────┐                     ┌──────────┐
  │Controller│──SSH──▶ host1       │  host1   │──git pull
  │          │──SSH──▶ host2       │  host2   │──git pull
  │          │──SSH──▶ host3       │  host3   │──git pull
  └──────────┘                     │  host4   │──git pull
                                   └──────────┘
  Bottleneck:                      Each host self-manages
  Controller resources             from Git repo
  SSH connections
```

### ansible-pull Setup

```bash
# On each managed node (cron job)
ansible-pull \
  -U https://github.com/org/ansible-config.git \
  -i localhost, \
  -d /opt/ansible-local \
  --clean \
  local.yml
```

### Cron-based Pull

```yaml
# Bootstrap playbook to install ansible-pull cron
- name: Configure ansible-pull
  hosts: all
  tasks:
    - name: Install Ansible
      apt:
        name: ansible
        state: present

    - name: Set up ansible-pull cron
      cron:
        name: "ansible-pull"
        minute: "*/15"
        job: >
          ansible-pull
          -U https://github.com/org/ansible-config.git
          -i localhost,
          -d /opt/ansible-local
          --clean
          local.yml
          >> /var/log/ansible-pull.log 2>&1
```

```
  PULL MODE ADVANTAGES            PULL MODE DRAWBACKS
  ════════════════════            ═══════════════════
  ✓ No SSH bottleneck            ✗ No real-time control
  ✓ Scales linearly              ✗ Harder to orchestrate
  ✓ Nodes self-heal              ✗ Requires Ansible installed
  ✓ Works behind firewalls       ✗ Git repo must be accessible
  ✓ Reduces controller load      ✗ Delayed convergence
```

---

## 3. Multi-Controller Architecture

```
  ══════════════════════════════════════════════════════════

  ┌─────────────────────────────────────────────────────┐
  │              Load Balancer / Job Scheduler           │
  └───────────┬─────────────┬─────────────┬─────────────┘
              │             │             │
        ┌─────▼─────┐ ┌────▼──────┐ ┌────▼──────┐
        │Controller 1│ │Controller 2│ │Controller 3│
        │ Region: EU │ │ Region: US │ │ Region: AP │
        │ forks: 50  │ │ forks: 50  │ │ forks: 50  │
        └─────┬──────┘ └─────┬──────┘ └─────┬──────┘
              │              │               │
        EU hosts        US hosts         AP hosts
```

### Split by Region/Function

```bash
# Controller 1 (EU)
ansible-playbook -i inventories/eu site.yml

# Controller 2 (US)
ansible-playbook -i inventories/us site.yml

# Controller 3 (AP)
ansible-playbook -i inventories/ap site.yml
```

---

## 4. Dynamic Inventory at Scale

```yaml
# inventories/aws_ec2.yml
---
plugin: amazon.aws.aws_ec2
regions:
  - us-east-1
  - us-west-2
  - eu-west-1
keyed_groups:
  - key: tags.Environment
    prefix: env
  - key: tags.Role
    prefix: role
  - key: placement.availability_zone
    prefix: az
compose:
  ansible_host: private_ip_address
filters:
  tag:ManagedBy: ansible
  instance-state-name: running
```

```
  DYNAMIC INVENTORY AT SCALE
  ══════════════════════════════════════════════════════════

  Cloud Provider API
       │
       ▼
  ┌─────────────────────────────────┐
  │  Dynamic Inventory Plugin       │
  │                                 │
  │  Filters:                       │
  │  - ManagedBy: ansible           │  Only relevant hosts
  │  - state: running               │
  │                                 │
  │  Caching:                       │
  │  - cache: true                  │  Don't query API
  │  - cache_timeout: 300           │  every run
  └─────────────────────────────────┘
       │
       ▼
  Grouped hosts ready for playbook
```

### Inventory Caching

```yaml
# inventories/aws_ec2.yml (with caching)
---
plugin: amazon.aws.aws_ec2
cache: true
cache_plugin: jsonfile
cache_connection: /tmp/aws_inventory_cache
cache_timeout: 300      # 5 minutes
```

---

## 5. Ansible Automation Platform (AAP)

```
  ANSIBLE AUTOMATION PLATFORM
  ══════════════════════════════════════════════════════════

  ┌─────────────────────────────────────────────────────┐
  │                                                     │
  │  ┌──────────────────────────────────────────────┐   │
  │  │  Automation Controller (Tower)                │   │
  │  │  - Web UI, API, RBAC, Scheduling              │   │
  │  │  - Workflow automation                        │   │
  │  └──────────────────────────────────────────────┘   │
  │                                                     │
  │  ┌──────────────────────────────────────────────┐   │
  │  │  Automation Hub                               │   │
  │  │  - Certified collections                      │   │
  │  │  - Private content management                 │   │
  │  └──────────────────────────────────────────────┘   │
  │                                                     │
  │  ┌──────────────────────────────────────────────┐   │
  │  │  Execution Environments                       │   │
  │  │  - Containerised Ansible runtimes             │   │
  │  │  - Consistent across controllers              │   │
  │  └──────────────────────────────────────────────┘   │
  │                                                     │
  │  ┌──────────────────────────────────────────────┐   │
  │  │  Event-Driven Ansible (EDA)                   │   │
  │  │  - React to events automatically              │   │
  │  │  - Webhook, log, monitoring triggers          │   │
  │  └──────────────────────────────────────────────┘   │
  │                                                     │
  └─────────────────────────────────────────────────────┘
```

### Execution Environments

```dockerfile
# execution-environment.yml
---
version: 3
build_arg_defaults:
  ANSIBLE_GALAXY_CLI_COLLECTION_OPTS: '--pre'

dependencies:
  galaxy: requirements.yml
  python: requirements.txt
  system: bindep.txt

additional_build_steps:
  append_final:
    - RUN pip install custom-library
```

```bash
# Build execution environment
ansible-builder build -t my-ee:latest -f execution-environment.yml
```

---

## 6. Scaling Patterns Summary

```
  ┌──────────────────────────────────────────────────────┐
  │  HOSTS    │  RECOMMENDED APPROACH                    │
  │───────────┼──────────────────────────────────────────│
  │  < 100    │  Single controller, forks=20             │
  │  100-500  │  Pipelining, fact caching, forks=50      │
  │  500-2000 │  Multi-controller, dynamic inventory     │
  │  2000+    │  ansible-pull OR AAP with mesh            │
  │  10,000+  │  Pull mode + AAP + event-driven          │
  └──────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Controller runs out of memory | Reduce forks; each fork uses ~100MB RAM |
| SSH connections timing out | Increase `ControlPersist`; use pipelining |
| Fact gathering too slow at scale | Enable fact caching (Redis for multi-controller) |
| Dynamic inventory API rate-limited | Enable inventory caching with `cache_timeout` |
| Pull mode nodes not converging | Check cron logs, Git access, network connectivity |
| Inconsistent environments | Use Execution Environments (containers) |

---

## Summary Table

| Strategy | Scale | Mechanism |
|----------|-------|-----------|
| Increase forks | 10-500 hosts | More parallel SSH connections |
| Pipelining | All scales | Eliminate file transfer overhead |
| Fact caching | 100+ hosts | Skip redundant fact gathering |
| Free strategy | 50+ hosts | Remove inter-host wait time |
| Multi-controller | 500+ hosts | Regional controllers |
| ansible-pull | 1000+ hosts | Nodes self-manage from Git |
| Dynamic inventory | Cloud | Auto-discover hosts from APIs |
| AAP/AWX | Enterprise | Central management, RBAC, UI |
| Execution Environments | Teams | Containerised consistent runtimes |
| Event-Driven Ansible | Reactive | Trigger automation on events |

---

## Quick Revision Questions

1. **What is the difference between push and pull mode in Ansible?**
   > Push mode (default) has the controller SSHing to nodes. Pull mode has each node running `ansible-pull` to fetch and apply configuration from a Git repository.

2. **When should you consider ansible-pull over push mode?**
   > When managing thousands of hosts, when nodes are behind firewalls, or when you need nodes to self-heal, as pull mode scales linearly.

3. **How does fact caching with Redis help in multi-controller setups?**
   > Redis provides a shared cache accessible by all controllers, so facts gathered by one controller are available to others without re-gathering.

4. **What are Execution Environments?**
   > Containerised Ansible runtimes that bundle Ansible, collections, Python dependencies, and system packages into a consistent, portable image.

5. **What scaling pattern is recommended for 500+ hosts?**
   > Multi-controller architecture with regional controllers, dynamic inventory with caching, pipelining, high forks count, and fact caching.

---

[← Previous: CI/CD Integration](05-ci-cd-integration.md) | [Next: Troubleshooting Guide →](07-troubleshooting-guide.md)
