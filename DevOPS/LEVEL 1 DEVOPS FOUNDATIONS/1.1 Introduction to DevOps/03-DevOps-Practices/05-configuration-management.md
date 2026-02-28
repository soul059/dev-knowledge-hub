# Chapter 3.5 — Configuration Management

[← Previous: Infrastructure as Code](04-infrastructure-as-code.md) | [Next: Monitoring & Observability →](06-monitoring-and-observability.md)

---

## Overview

**Configuration Management (CM)** is the practice of systematically handling changes to a system's configuration so it maintains its integrity over time. While IaC creates infrastructure, CM ensures every server is configured consistently — same packages, same settings, same security policies.

---

## IaC vs Configuration Management

```
┌──────────────────────────────────────────────────────────┐
│  IaC vs CONFIGURATION MANAGEMENT                         │
│                                                          │
│  IaC = "Build the house"                                 │
│  ├── Create VMs, networks, load balancers                │
│  ├── Tools: Terraform, CloudFormation                    │
│  └── Focus: Provisioning infrastructure                  │
│                                                          │
│  CM = "Furnish and maintain the house"                   │
│  ├── Install packages, configure services                │
│  ├── Tools: Ansible, Chef, Puppet, SaltStack             │
│  └── Focus: Configuring what runs ON infrastructure      │
│                                                          │
│  ┌──────────┐  Terraform  ┌──────────┐  Ansible  ┌─────┐│
│  │  Code    │────────────►│  VM      │──────────►│ App ││
│  │  Repo    │  provisions │  Created │  installs │ OK  ││
│  └──────────┘             └──────────┘  + config  └─────┘│
│                                                          │
│  MODERN TREND: Immutable infrastructure (bake everything │
│  into the image with Packer, skip CM on live servers)    │
└──────────────────────────────────────────────────────────┘
```

---

## Push vs Pull Models

```
┌──────────────────────────────────────────────────────────┐
│  PUSH MODEL (Ansible)           PULL MODEL (Puppet/Chef) │
│  ════════════════════           ═══════════════════════   │
│                                                          │
│  ┌─────────┐                    ┌───────────┐           │
│  │ Control │                    │   Master   │           │
│  │ Machine │                    │   Server   │           │
│  └────┬────┘                    └─────┬─────┘           │
│       │ SSH push                      │                  │
│       ├──────► Node 1           Node 1 ───┤              │
│       ├──────► Node 2           Node 2 ───┤ Agents PULL  │
│       └──────► Node 3           Node 3 ───┘ (every 30m)  │
│                                                          │
│  ✅ No agents needed            ✅ Self-healing           │
│  ✅ Simple to start             ✅ Continuous enforcement │
│  ⚠️ One-time execution          ✅ Central management    │
│  ⚠️ Must re-run to enforce      ⚠️ Agent installation    │
└──────────────────────────────────────────────────────────┘
```

---

## Ansible: The Most Popular CM Tool

### Why Ansible?

- **Agentless** — Uses SSH (Linux) or WinRM (Windows), no software to install on targets
- **YAML-based** — Easy to read and write
- **Idempotent** — Safe to run repeatedly
- **Massive community** — 40,000+ modules on Ansible Galaxy

### Ansible Architecture

```
┌──────────────────────────────────────────────────────────┐
│  ANSIBLE ARCHITECTURE                                    │
│                                                          │
│  ┌──────────────┐                                       │
│  │  Playbook    │  ← YAML files defining tasks          │
│  │  (tasks.yml) │                                       │
│  └──────┬───────┘                                       │
│         │                                                │
│  ┌──────▼───────┐                                       │
│  │  Inventory   │  ← List of target hosts               │
│  │  (hosts.ini) │                                       │
│  └──────┬───────┘                                       │
│         │                                                │
│  ┌──────▼───────┐                                       │
│  │   Modules    │  ← apt, yum, copy, service, etc.      │
│  └──────┬───────┘                                       │
│         │ SSH                                            │
│  ┌──────▼──────────────────────────┐                    │
│  │  Managed Nodes                  │                    │
│  │  ┌─────┐  ┌─────┐  ┌─────┐    │                    │
│  │  │Web 1│  │Web 2│  │DB 1 │    │                    │
│  │  └─────┘  └─────┘  └─────┘    │                    │
│  └─────────────────────────────────┘                    │
└──────────────────────────────────────────────────────────┘
```

### Inventory File

```ini
# inventory/hosts.ini

[webservers]
web1.example.com ansible_user=ubuntu
web2.example.com ansible_user=ubuntu
web3.example.com ansible_user=ubuntu

[databases]
db1.example.com ansible_user=admin
db2.example.com ansible_user=admin

[monitoring]
grafana.example.com ansible_user=ubuntu

[production:children]
webservers
databases
monitoring

[production:vars]
ansible_python_interpreter=/usr/bin/python3
env=production
```

### Complete Ansible Playbook

```yaml
# playbooks/setup-webserver.yml
---
- name: Configure Web Servers
  hosts: webservers
  become: yes  # Run as root
  vars:
    app_port: 8080
    app_user: appuser
    nginx_version: "1.24.*"

  tasks:
    # 1. System updates
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600

    # 2. Install packages
    - name: Install required packages
      apt:
        name:
          - nginx={{ nginx_version }}
          - python3
          - python3-pip
          - ufw
          - fail2ban
        state: present

    # 3. Create application user
    - name: Create app user
      user:
        name: "{{ app_user }}"
        shell: /bin/bash
        groups: www-data
        create_home: yes

    # 4. Deploy Nginx configuration
    - name: Copy Nginx config
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/sites-available/default
        owner: root
        group: root
        mode: '0644'
      notify: Restart Nginx

    # 5. Configure firewall
    - name: Allow SSH
      ufw:
        rule: allow
        name: OpenSSH

    - name: Allow HTTP/HTTPS
      ufw:
        rule: allow
        port: "{{ item }}"
        proto: tcp
      loop:
        - "80"
        - "443"

    - name: Enable UFW
      ufw:
        state: enabled
        policy: deny

    # 6. Ensure services are running
    - name: Start and enable Nginx
      service:
        name: nginx
        state: started
        enabled: yes

  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

### Ansible Roles (Reusable Components)

```
roles/
└── webserver/
    ├── defaults/
    │   └── main.yml       # Default variables
    ├── files/
    │   └── index.html     # Static files to copy
    ├── handlers/
    │   └── main.yml       # Handler definitions
    ├── meta/
    │   └── main.yml       # Role metadata & dependencies
    ├── tasks/
    │   └── main.yml       # Main task list
    ├── templates/
    │   └── nginx.conf.j2  # Jinja2 templates
    └── vars/
        └── main.yml       # Role variables
```

### Common Ansible Commands

```bash
# Run a playbook
ansible-playbook -i inventory/hosts.ini playbooks/setup.yml

# Dry run (check mode)
ansible-playbook --check playbooks/setup.yml

# Run with verbose output
ansible-playbook -vvv playbooks/setup.yml

# Limit to specific hosts
ansible-playbook playbooks/setup.yml --limit web1.example.com

# Run ad-hoc command on all webservers
ansible webservers -m ping
ansible webservers -m shell -a "uptime"

# List all hosts in inventory
ansible-inventory --list

# Encrypt secrets with Ansible Vault
ansible-vault encrypt secrets.yml
ansible-vault decrypt secrets.yml
ansible-playbook --ask-vault-pass playbooks/setup.yml
```

---

## Idempotency Deep Dive

```
┌──────────────────────────────────────────────────────────┐
│  IDEMPOTENCY = "Same input, same result, every time"     │
│                                                          │
│  Run 1: "Install nginx"                                  │
│  → nginx not found → installs nginx → CHANGED ✅         │
│                                                          │
│  Run 2: "Install nginx"                                  │
│  → nginx already installed → nothing happens → OK ✅     │
│                                                          │
│  Run 3: "Install nginx"                                  │
│  → nginx already installed → nothing happens → OK ✅     │
│                                                          │
│  ❌ NOT idempotent (bad):                                │
│     shell: echo "export VAR=val" >> /etc/profile         │
│     (adds line EVERY run — file grows forever)           │
│                                                          │
│  ✅ Idempotent (good):                                   │
│     lineinfile:                                          │
│       path: /etc/profile                                 │
│       line: "export VAR=val"                             │
│     (ensures line exists — adds only if missing)         │
└──────────────────────────────────────────────────────────┘
```

---

## Mutable vs Immutable Infrastructure

```
┌──────────────────────────────────────────────────────────┐
│  MUTABLE (Pets)                  IMMUTABLE (Cattle)      │
│  ══════════════                  ═════════════════        │
│                                                          │
│  ┌──────┐                        ┌──────┐               │
│  │Server│──► Update ──►          │Image │──► Deploy new  │
│  │  v1  │    in-place   ┌──────┐ │  v2  │    instances   │
│  └──────┘              │Server│ └──────┘               │
│                         │ v1.1 │                        │
│                         └──────┘ Destroy old instances   │
│                                                          │
│  CM keeps changing      New image per release            │
│  live servers           No changes to running servers    │
│                                                          │
│  Problems:              Benefits:                        │
│  ├── Config drift       ├── No drift                    │
│  ├── Snowflake servers  ├── Consistent state            │
│  ├── "It works on       ├── Easy rollback (redeploy     │
│  │    that server"      │    old image)                  │
│  └── Hard to reproduce  └── Fully reproducible          │
│                                                          │
│  Build tool: CM (Ansible)  Build tool: Packer + Docker   │
└──────────────────────────────────────────────────────────┘
```

---

## Real-World Scenario: Configuring 200 Servers

### 🏢 E-Commerce Company with Black Friday Surge

```
SITUATION:
├── Company needs to add 150 web servers for Black Friday
├── Each server needs: Nginx, app code, SSL certs, monitoring agent
├── Must be ready in 2 hours
└── Must be identical configurations

SOLUTION WITH ANSIBLE:
1. Terraform provisions 150 EC2 instances (5 minutes)
2. Dynamic inventory auto-discovers new instances
3. Ansible playbook runs against all 150 servers:
   ├── Install & configure Nginx        (parallel across 30 hosts)
   ├── Deploy application code           (parallel)
   ├── Copy SSL certificates             (parallel)
   ├── Install Datadog monitoring agent  (parallel)
   └── Run smoke tests                   (parallel)
4. Total time: ~15 minutes for all 150 servers
5. After Black Friday: terraform destroy (remove extra servers)
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Playbook fails on connection | SSH key not configured | Use `ansible -m ping` to test connectivity first |
| Not idempotent | Using `shell`/`command` modules | Use purpose-built modules (apt, service, copy) |
| Secrets in plain text | Variables contain passwords | Use Ansible Vault to encrypt sensitive data |
| Playbook too slow | Running tasks sequentially | Use `serial`, `async`, `poll`, or increase forks |
| Works on dev, fails on prod | Different OS versions | Use `ansible_os_family` facts for conditional tasks |
| Configuration drift | Manual changes on servers | Run playbooks on schedule (cron/Jenkins), use immutable infra |

---

## Summary Table

| CM Concept | Description |
|-----------|-------------|
| **Configuration Management** | Maintaining consistent server configurations through automation |
| **Push Model** | Control machine pushes configs to nodes (Ansible — SSH-based) |
| **Pull Model** | Agents on nodes pull configs from master (Puppet, Chef) |
| **Idempotency** | Same playbook run multiple times = same result |
| **Playbook** | YAML file defining a set of tasks to execute |
| **Role** | Reusable, structured collection of tasks, files, and templates |
| **Inventory** | List of managed hosts with grouping and variables |
| **Ansible Vault** | Encryption tool for secrets used in playbooks |

---

## Quick Revision Questions

1. **What is the difference between IaC and Configuration Management?**
   <details><summary>Answer</summary>IaC provisions the infrastructure itself (VMs, networks, databases) using tools like Terraform. Configuration Management configures what runs on that infrastructure (install packages, manage services, deploy configs) using tools like Ansible. IaC = "build the house," CM = "furnish and maintain the house."</details>

2. **What is the difference between push and pull configuration models?**
   <details><summary>Answer</summary>Push (Ansible): A control machine pushes configuration to nodes via SSH on demand. Pull (Puppet/Chef): An agent on each node periodically pulls its configuration from a central master server. Push is simpler to set up but doesn't self-heal; Pull continuously enforces desired state.</details>

3. **Why is idempotency important in configuration management?**
   <details><summary>Answer</summary>Idempotency ensures running the same configuration multiple times produces the same result. This means you can safely re-run playbooks without creating duplicates, adding extra lines, or breaking existing configurations. It's what makes CM reliable and predictable.</details>

4. **What is Ansible Vault, and when would you use it?**
   <details><summary>Answer</summary>Ansible Vault is a feature that encrypts sensitive data (passwords, API keys, certificates) within Ansible files. You use it whenever playbooks need to handle secrets, so they can be safely stored in version control without exposing credentials in plain text.</details>

5. **What is immutable infrastructure, and how does it relate to CM?**
   <details><summary>Answer</summary>Immutable infrastructure means never modifying running servers. Instead, you build a new image (using Packer/Docker) with all configurations baked in, and deploy new instances. This eliminates configuration drift. It reduces the need for CM tools on live servers, since servers are replaced rather than updated.</details>

6. **Why is Ansible called "agentless," and what advantage does this provide?**
   <details><summary>Answer</summary>Ansible is agentless because it doesn't require any software to be installed on managed nodes — it connects over SSH (Linux) or WinRM (Windows). This advantage means: less overhead on managed nodes, no agent update maintenance, faster onboarding of new servers, and easier security compliance.</details>

---

[← Previous: Infrastructure as Code](04-infrastructure-as-code.md) | [Next: Monitoring & Observability →](06-monitoring-and-observability.md)
