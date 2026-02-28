# Chapter 3.2: Dynamic Inventory

[← Previous: Static Inventory](01-static-inventory.md) | [Next: Inventory Variables →](03-inventory-variables.md)

---

## Overview

Dynamic inventory **automatically discovers** hosts from external sources like cloud providers, CMDBs, or container orchestrators. Instead of manually maintaining a hosts file, Ansible queries APIs in real-time to build the inventory.

---

## Why Dynamic Inventory?

```
  STATIC vs DYNAMIC INVENTORY
  ══════════════════════════════════════════════════════════

  STATIC (Manual)                    DYNAMIC (Automatic)
  ┌──────────────────┐              ┌──────────────────┐
  │  hosts.ini       │              │  AWS API         │
  │                  │              │  Azure API       │
  │  web1 10.0.1.1   │              │  GCP API         │
  │  web2 10.0.1.2   │              │  Docker API      │
  │  web3 10.0.1.3   │              │                  │
  │  ...             │              │  Auto-discover   │
  │  web50 10.0.1.50 │              │  all instances   │
  └──────────────────┘              └──────────────────┘
       │                                │
       │ Problem:                       │ Solution:
       │ - Manual updates               │ - Always current
       │ - Stale entries                 │ - Auto-scaling aware
       │ - Error-prone                   │ - Tag-based grouping
       │ - Doesn't scale                │ - No maintenance
       └────────────────────────────────┘
```

---

## Dynamic Inventory Architecture

```
  DYNAMIC INVENTORY FLOW
  ══════════════════════════════════════════════════════════

  ┌──────────────┐     ┌────────────────┐     ┌──────────┐
  │   Ansible    │────►│   Inventory    │────►│  Cloud   │
  │   Engine     │     │   Plugin/      │     │  API     │
  │              │     │   Script       │     │          │
  │              │     │                │     │ AWS EC2  │
  │              │◄────│  Returns JSON  │◄────│ Azure RM │
  │              │     │  {hosts,groups}│     │ GCP      │
  └──────────────┘     └────────────────┘     └──────────┘

  The plugin/script returns JSON:
  {
    "webservers": {
      "hosts": ["10.0.1.1", "10.0.1.2"],
      "vars": { "http_port": 80 }
    },
    "dbservers": {
      "hosts": ["10.0.2.1"],
      "vars": { "db_port": 5432 }
    },
    "_meta": {
      "hostvars": {
        "10.0.1.1": { "instance_id": "i-abc123" }
      }
    }
  }
```

---

## AWS EC2 Dynamic Inventory

### Plugin Configuration

```yaml
# inventory/aws_ec2.yml
plugin: amazon.aws.aws_ec2

# AWS credentials (or use IAM roles / env vars)
aws_access_key_id: AKIAIOSFODNN7EXAMPLE
aws_secret_access_key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

# Regions to query
regions:
  - us-east-1
  - us-west-2

# Filters (only include matching instances)
filters:
  instance-state-name: running
  "tag:Environment": production

# Group hosts by tags, instance type, etc.
keyed_groups:
  - key: tags.Environment
    prefix: env
    separator: "_"
  - key: instance_type
    prefix: instance_type
  - key: placement.region
    prefix: region

# Use tag "Name" as hostname
hostnames:
  - tag:Name
  - private-ip-address

# Set connection variables
compose:
  ansible_host: private_ip_address
  ansible_user: "'ubuntu'"
```

```bash
# Install required collection
ansible-galaxy collection install amazon.aws

# Install boto3
pip install boto3 botocore

# Test inventory
ansible-inventory -i inventory/aws_ec2.yml --graph
ansible-inventory -i inventory/aws_ec2.yml --list

# Use in playbook
ansible-playbook -i inventory/aws_ec2.yml site.yml
```

---

## Azure Dynamic Inventory

```yaml
# inventory/azure_rm.yml
plugin: azure.azcollection.azure_rm

# Authentication (or use env vars / managed identity)
auth_source: auto

# Include VMs from specific resource groups
include_vm_resource_groups:
  - production-rg
  - staging-rg

# Group by tags
keyed_groups:
  - prefix: tag
    key: tags
  - prefix: os
    key: os_profile.system

# Conditionally add hosts
conditional_groups:
  webservers: "'web' in name"
  dbservers: "'db' in name"

compose:
  ansible_host: public_ip_addresses[0] | default(private_ip_addresses[0], true)
```

```bash
# Install Azure collection
ansible-galaxy collection install azure.azcollection
pip install msrestazure azure-mgmt-compute azure-mgmt-network
```

---

## Custom Inventory Script

```python
#!/usr/bin/env python3
# inventory/custom_inventory.py
"""Custom dynamic inventory script."""

import json
import argparse
import requests

def get_inventory():
    """Query your CMDB/API for hosts."""
    # Example: Query internal CMDB
    # response = requests.get("https://cmdb.internal/api/servers")
    # servers = response.json()

    inventory = {
        "webservers": {
            "hosts": ["web1.internal", "web2.internal"],
            "vars": {
                "http_port": 80,
                "ansible_user": "deploy"
            }
        },
        "dbservers": {
            "hosts": ["db1.internal"],
            "vars": {
                "db_port": 5432
            }
        },
        "_meta": {
            "hostvars": {
                "web1.internal": {
                    "ansible_host": "192.168.1.10",
                    "server_role": "primary"
                },
                "web2.internal": {
                    "ansible_host": "192.168.1.11",
                    "server_role": "secondary"
                },
                "db1.internal": {
                    "ansible_host": "192.168.2.10"
                }
            }
        }
    }
    return inventory

def get_host(hostname):
    """Return variables for a specific host."""
    inventory = get_inventory()
    return inventory.get("_meta", {}).get("hostvars", {}).get(hostname, {})

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--list", action="store_true")
    parser.add_argument("--host")
    args = parser.parse_args()

    if args.list:
        print(json.dumps(get_inventory(), indent=2))
    elif args.host:
        print(json.dumps(get_host(args.host), indent=2))
```

```bash
# Make executable
chmod +x inventory/custom_inventory.py

# Test script
./inventory/custom_inventory.py --list
./inventory/custom_inventory.py --host web1.internal

# Use with Ansible
ansible-playbook -i inventory/custom_inventory.py site.yml
```

---

## Combining Static and Dynamic Inventory

```
  INVENTORY DIRECTORY (Mixed)
  ══════════════════════════════════════════════════

  inventory/
  ├── static_hosts.ini         ← Static hosts
  ├── aws_ec2.yml              ← AWS dynamic
  ├── azure_rm.yml             ← Azure dynamic
  └── group_vars/
      └── all.yml              ← Variables for all

  # Point Ansible to the directory
  ansible-playbook -i inventory/ site.yml

  # Ansible merges ALL sources in the directory
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Dynamic inventory returns empty | Check API credentials and filters |
| Script not executing | Ensure file is executable (`chmod +x`) |
| Cloud plugin not found | Install required collection (`ansible-galaxy collection install`) |
| Slow inventory loading | Use caching: `cache = true`, `cache_plugin = jsonfile` |
| `--list` returns error | Test script manually with `python3 script.py --list` |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| Dynamic inventory | Auto-discovers hosts from external sources |
| Inventory plugin | Built-in Ansible plugin for cloud providers |
| Inventory script | Custom executable returning JSON |
| `--list` | Script flag to return full inventory |
| `--host <name>` | Script flag to return host-specific vars |
| `_meta.hostvars` | Per-host variables in JSON output |
| `keyed_groups` | Auto-create groups from host attributes |
| `compose` | Set host variables from attributes |

---

## Quick Revision Questions

1. **When should you use dynamic inventory over static?**
   > When infrastructure is cloud-based, auto-scaling, or frequently changing. Dynamic inventory always reflects the current state.

2. **What JSON structure must a dynamic inventory script return?**
   > Groups with hosts arrays, optional group vars, and a `_meta.hostvars` section for per-host variables.

3. **What are the two required flags for inventory scripts?**
   > `--list` (returns complete inventory) and `--host <hostname>` (returns variables for one host).

4. **How do you combine static and dynamic inventory?**
   > Place all inventory files (static and dynamic) in a directory and point Ansible to that directory with `-i inventory/`.

5. **What is `keyed_groups` in inventory plugins?**
   > It automatically creates Ansible groups based on host attributes like tags, instance types, or regions.

6. **How do you speed up dynamic inventory that queries slow APIs?**
   > Enable inventory caching with `cache = true` and set a cache plugin (e.g., jsonfile) with an appropriate timeout.

---

[← Previous: Static Inventory](01-static-inventory.md) | [Next: Inventory Variables →](03-inventory-variables.md)
