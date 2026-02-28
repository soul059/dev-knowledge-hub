# Chapter 5.2: Facts and setup Module

[← Previous: Variable Precedence](01-variable-precedence.md) | [Next: Magic Variables →](03-magic-variables.md)

---

## Overview

**Facts** are system properties automatically gathered by Ansible from managed hosts. They include hardware details, network configuration, OS information, and more. Facts are collected by the `setup` module, which runs at the start of each play when `gather_facts: yes` (the default).

---

## How Facts Are Gathered

```
  FACT GATHERING FLOW
  ══════════════════════════════════════════════════════════

  Control Node                     Managed Host
  ┌──────────────┐                ┌──────────────────────┐
  │  Play starts │                │                      │
  │              │───SSH──────────►│ 1. Runs setup module │
  │              │                │    (Python script)   │
  │              │                │                      │
  │              │◄───JSON────────│ 2. Returns facts     │
  │              │                │    as JSON            │
  │  3. Stores   │                └──────────────────────┘
  │  as ansible_ │
  │  variables   │
  └──────────────┘

  Facts available as:  ansible_hostname
                       ansible_os_family
                       ansible_memtotal_mb
                       ansible_default_ipv4.address
                       ...hundreds more
```

---

## Running the setup Module

```bash
# Gather all facts from a host
ansible web01 -m setup

# Filter specific facts
ansible web01 -m setup -a "filter=ansible_distribution*"
ansible web01 -m setup -a "filter=ansible_memory*"
ansible web01 -m setup -a "filter=ansible_default_ipv4"

# Save facts to files
ansible all -m setup --tree /tmp/facts/
```

---

## Key Facts Categories

### System Information

```yaml
# OS and Distribution
ansible_os_family: "Debian"              # Debian, RedHat, etc.
ansible_distribution: "Ubuntu"           # Ubuntu, CentOS, etc.
ansible_distribution_version: "22.04"
ansible_distribution_major_version: "22"
ansible_distribution_release: "jammy"

# Kernel
ansible_kernel: "5.15.0-76-generic"
ansible_kernel_version: "#83-Ubuntu SMP"
ansible_architecture: "x86_64"

# Hostname
ansible_hostname: "web01"                # Short hostname
ansible_fqdn: "web01.example.com"        # FQDN
ansible_nodename: "web01"
```

### Hardware

```yaml
# CPU
ansible_processor_vcpus: 4              # Total vCPUs
ansible_processor_cores: 2              # Cores per socket
ansible_processor_count: 1              # Physical CPUs
ansible_processor:                       # CPU model info
  - "0"
  - "GenuineIntel"
  - "Intel(R) Xeon(R) CPU E5-2686 v4"

# Memory
ansible_memtotal_mb: 7982               # Total RAM in MB
ansible_memfree_mb: 5124                # Free RAM in MB
ansible_swaptotal_mb: 0
ansible_swapfree_mb: 0

# Storage
ansible_mounts:
  - mount: /
    device: /dev/sda1
    fstype: ext4
    size_total: 52710469632
    size_available: 45012762624
```

### Network

```yaml
# Default IPv4
ansible_default_ipv4:
  address: "10.0.1.10"
  gateway: "10.0.1.1"
  interface: "eth0"
  netmask: "255.255.255.0"
  network: "10.0.1.0"

# All interfaces
ansible_interfaces:
  - lo
  - eth0
  - eth1

# Specific interface
ansible_eth0:
  ipv4:
    address: "10.0.1.10"
    netmask: "255.255.255.0"
  macaddress: "0a:1b:2c:3d:4e:5f"

# DNS
ansible_dns:
  nameservers:
    - "8.8.8.8"
    - "8.8.4.4"
```

### Date and Time

```yaml
ansible_date_time:
  date: "2024-01-15"
  time: "14:30:45"
  epoch: "1705327845"
  iso8601: "2024-01-15T14:30:45Z"
  tz: "UTC"
  tz_offset: "+0000"
  weekday: "Monday"
```

---

## Using Facts in Playbooks

```yaml
---
- name: Configure based on facts
  hosts: all
  become: yes
  tasks:
    # OS-specific package management
    - name: Install packages (Debian)
      apt:
        name: nginx
        state: present
      when: ansible_os_family == "Debian"

    - name: Install packages (RedHat)
      yum:
        name: nginx
        state: present
      when: ansible_os_family == "RedHat"

    # Hardware-based configuration
    - name: Set worker processes based on CPU
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      vars:
        workers: "{{ ansible_processor_vcpus }}"

    # Network-based config
    - name: Configure app to bind to primary IP
      template:
        src: app.conf.j2
        dest: /etc/myapp/config.yml
      vars:
        bind_address: "{{ ansible_default_ipv4.address }}"

    # Memory-based decisions
    - name: Configure Java heap
      lineinfile:
        path: /etc/default/elasticsearch
        regexp: '^ES_JAVA_OPTS'
        line: 'ES_JAVA_OPTS="-Xms{{ (ansible_memtotal_mb * 0.5) | int }}m -Xmx{{ (ansible_memtotal_mb * 0.5) | int }}m"'
```

### In Templates

```jinja2
{# nginx.conf.j2 #}
# Managed by Ansible
# Host: {{ ansible_hostname }}
# OS: {{ ansible_distribution }} {{ ansible_distribution_version }}
# CPUs: {{ ansible_processor_vcpus }}

worker_processes {{ ansible_processor_vcpus }};

events {
    worker_connections {{ ansible_processor_vcpus * 1024 }};
}

http {
    server {
        listen {{ ansible_default_ipv4.address }}:80;
        server_name {{ ansible_fqdn }};
    }
}
```

---

## Controlling Fact Gathering

```yaml
# Disable fact gathering (faster for simple tasks)
- name: Quick task
  hosts: all
  gather_facts: no
  tasks:
    - name: Copy file
      copy:
        src: file.txt
        dest: /tmp/file.txt

# Gather facts explicitly later
- name: Delayed fact gathering
  hosts: all
  gather_facts: no
  tasks:
    - name: Do something first
      debug:
        msg: "No facts collected yet"

    - name: Now gather facts
      setup:                          # Explicit setup call

    - name: Use facts
      debug:
        msg: "OS: {{ ansible_distribution }}"

# Gather subset of facts (faster)
- name: Minimal facts
  hosts: all
  gather_facts: yes
  gather_subset:
    - network               # Only network facts
    - hardware               # Only hardware facts
    - "!facter"              # Exclude facter
    - min                    # Minimal set
```

### Gather Subset Options

```
  ┌─────────────────┬──────────────────────────────────┐
  │ Subset           │ Description                      │
  ├─────────────────┼──────────────────────────────────┤
  │ all              │ Everything (default)             │
  │ min              │ Minimal: hostname, IPs, OS       │
  │ network          │ Network interfaces, IPs, DNS     │
  │ hardware         │ CPU, memory, disk                │
  │ virtual          │ Virtualization info              │
  │ ohai             │ Chef Ohai facts                  │
  │ facter           │ Puppet Facter facts              │
  │ !subset          │ Exclude a subset                 │
  └─────────────────┴──────────────────────────────────┘
```

---

## Custom Facts (local facts)

Create custom facts on managed hosts at `/etc/ansible/facts.d/`:

```
  CUSTOM FACTS
  ══════════════════════════════════════════════════════════

  Managed Host: /etc/ansible/facts.d/
  ├── app.fact            ◄── INI or JSON file
  ├── environment.fact
  └── custom_script.fact  ◄── Executable script (must output JSON)

  Accessible via: ansible_local.app.section.key
```

### Static Custom Facts (INI)

```ini
; /etc/ansible/facts.d/app.fact
[general]
app_name = myapp
app_version = 2.1.0
env = production

[database]
host = db.internal
port = 5432
```

```yaml
# Access in playbook
- debug:
    msg: |
      App: {{ ansible_local.app.general.app_name }}
      Version: {{ ansible_local.app.general.app_version }}
      DB Host: {{ ansible_local.app.database.host }}
```

### Dynamic Custom Facts (Script)

```bash
#!/bin/bash
# /etc/ansible/facts.d/runtime.fact
# Must be executable and output JSON

cat <<EOF
{
  "uptime_hours": $(awk '{print int($1/3600)}' /proc/uptime),
  "load_average": "$(uptime | awk -F'load average:' '{print $2}' | xargs)",
  "disk_usage_percent": $(df / | tail -1 | awk '{print $5}' | tr -d '%'),
  "running_containers": $(docker ps -q 2>/dev/null | wc -l)
}
EOF
```

### Deploying Custom Facts

```yaml
tasks:
  - name: Create facts directory
    file:
      path: /etc/ansible/facts.d
      state: directory
      mode: '0755'

  - name: Deploy custom fact file
    template:
      src: app.fact.j2
      dest: /etc/ansible/facts.d/app.fact
      mode: '0644'

  - name: Re-gather facts to pick up custom facts
    setup:
      filter: ansible_local
```

---

## Accessing Other Hosts' Facts

```yaml
tasks:
  # Access facts from another host
  - name: Get database IP from db server
    debug:
      msg: "DB IP: {{ hostvars['db01']['ansible_default_ipv4']['address'] }}"

  # Configure app with DB host's IP
  - name: Configure database connection
    template:
      src: db-config.j2
      dest: /etc/myapp/database.yml
    vars:
      db_ip: "{{ hostvars[groups['dbservers'][0]]['ansible_default_ipv4']['address'] }}"
```

---

## Real-World Scenario: Auto-Tuning Application

```yaml
---
- name: Auto-tune application based on hardware
  hosts: appservers
  become: yes
  tasks:
    - name: Calculate optimal settings
      set_fact:
        java_heap_mb: "{{ (ansible_memtotal_mb * 0.6) | int }}"
        app_workers: "{{ ansible_processor_vcpus * 2 }}"
        app_threads: "{{ ansible_processor_vcpus * 4 }}"
        max_connections: "{{ (ansible_memtotal_mb / 10) | int }}"

    - name: Deploy tuned configuration
      template:
        src: app-config.j2
        dest: /etc/myapp/config.yml
      notify: Restart app

    - name: Show applied settings
      debug:
        msg: |
          Host: {{ ansible_hostname }} ({{ ansible_distribution }} {{ ansible_distribution_version }})
          CPUs: {{ ansible_processor_vcpus }}
          RAM: {{ ansible_memtotal_mb }}MB
          Heap: {{ java_heap_mb }}MB
          Workers: {{ app_workers }}
          Threads: {{ app_threads }}
          Max Connections: {{ max_connections }}

  handlers:
    - name: Restart app
      service:
        name: myapp
        state: restarted
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Fact is undefined | Check `gather_facts: yes` (default); variable name is exact |
| Facts too slow | Use `gather_subset: min` or `gather_facts: no` |
| Custom fact not showing | Ensure file is in `/etc/ansible/facts.d/` with `.fact` extension |
| Custom script fact empty | Check script is executable and outputs valid JSON |
| Can't access other host's facts | Ensure the other host is in the play or facts are cached |
| Nested fact access fails | Use bracket notation: `ansible_eth0['ipv4']['address']` |

---

## Summary Table

| Feature | Description |
|---------|-------------|
| `gather_facts: yes` | Auto-collect facts at play start (default) |
| `setup` module | Collect facts explicitly |
| `gather_subset` | Limit fact collection scope |
| `ansible_*` | Fact variable prefix |
| `ansible_local` | Custom facts from `/etc/ansible/facts.d/` |
| `filter=` | Filter facts in ad-hoc commands |
| `hostvars` | Access other hosts' facts |
| `.fact` files | INI, JSON, or executable scripts |
| `--tree` | Save facts to directory as JSON files |

---

## Quick Revision Questions

1. **What module collects facts from managed hosts?**
   > The `setup` module. It runs automatically at the start of each play when `gather_facts: yes`.

2. **How do you access a host's primary IP address in a playbook?**
   > `{{ ansible_default_ipv4.address }}`

3. **How do you speed up fact gathering?**
   > Use `gather_subset: min` to collect only basic facts, or `gather_facts: no` to skip entirely.

4. **Where do custom facts (local facts) live on managed hosts?**
   > In `/etc/ansible/facts.d/` as `.fact` files (INI, JSON, or executable scripts). They appear under `ansible_local`.

5. **How do you access facts from a different host?**
   > Use `hostvars['hostname']['ansible_fact_name']`. The other host must have facts gathered.

6. **What prefix do all Ansible facts use?**
   > `ansible_` — e.g., `ansible_hostname`, `ansible_os_family`, `ansible_memtotal_mb`.

---

[← Previous: Variable Precedence](01-variable-precedence.md) | [Next: Magic Variables →](03-magic-variables.md)
