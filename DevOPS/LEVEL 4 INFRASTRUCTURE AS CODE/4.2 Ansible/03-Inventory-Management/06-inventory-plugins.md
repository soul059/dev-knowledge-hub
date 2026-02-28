# Chapter 3.6: Inventory Plugins

[← Previous: Host Patterns](05-host-patterns.md) | [Next: Unit 4 - Playbook Structure →](../04-Playbooks/01-playbook-structure.md)

---

## Overview

Inventory plugins are the **modern way** to create dynamic inventories. They replace legacy inventory scripts with a declarative YAML configuration, providing caching, filtering, and grouping capabilities built into Ansible.

---

## Built-in Inventory Plugins

| Plugin | Source | Collection |
|--------|--------|------------|
| `ini` | INI files | built-in |
| `yaml` | YAML files | built-in |
| `auto` | Auto-detect format | built-in |
| `constructed` | Build groups from vars | built-in |
| `host_list` | Comma-separated list | built-in |
| `amazon.aws.aws_ec2` | AWS EC2 | amazon.aws |
| `azure.azcollection.azure_rm` | Azure VMs | azure.azcollection |
| `google.cloud.gcp_compute` | GCP instances | google.cloud |
| `community.docker.docker_containers` | Docker | community.docker |
| `kubernetes.core.k8s` | Kubernetes | kubernetes.core |

---

## Constructed Inventory Plugin

The `constructed` plugin creates **new groups and variables** from existing inventory data:

```yaml
# inventory/constructed.yml
plugin: constructed
strict: false

# Create groups based on existing variables
groups:
  # Group by OS family
  debian_hosts: "ansible_os_family == 'Debian'"
  redhat_hosts: "ansible_os_family == 'RedHat'"
  
  # Group by environment tag
  production_servers: "env == 'production'"
  staging_servers: "env == 'staging'"
  
  # Group by custom logic
  large_instances: "ansible_memtotal_mb > 8192"

# Create groups from variable values
keyed_groups:
  - key: env
    prefix: environment
  - key: ansible_os_family
    prefix: os

# Set new variables
compose:
  # Create a display name
  display_name: "inventory_hostname ~ ' (' ~ ansible_host ~ ')'"
  # Calculate derived values
  is_production: "env == 'production'"
```

```bash
# Use constructed with another inventory
ansible-inventory -i inventory/hosts.ini -i inventory/constructed.yml --graph
```

---

## Enabling Plugins

```ini
# ansible.cfg
[inventory]
enable_plugins = host_list, ini, yaml, auto, amazon.aws.aws_ec2, constructed
```

---

## Plugin File Naming

Inventory plugins are identified by their file suffix:

| Plugin | Required Suffix |
|--------|----------------|
| AWS EC2 | `*aws_ec2.yml` or `*aws_ec2.yaml` |
| Azure | `*azure_rm.yml` |
| GCP | `*gcp.yml` |
| Docker | `*docker.yml` |
| Constructed | Must have `plugin: constructed` |

```bash
# These filenames trigger the correct plugin:
inventory/prod_aws_ec2.yml      # → AWS EC2 plugin
inventory/my_azure_rm.yml       # → Azure plugin
inventory/cloud_gcp.yml         # → GCP plugin
```

---

## Caching

```yaml
# Enable caching in plugin config
plugin: amazon.aws.aws_ec2
cache: true
cache_plugin: jsonfile
cache_connection: /tmp/aws_inventory_cache
cache_timeout: 3600    # seconds (1 hour)
```

```ini
# Or in ansible.cfg
[inventory]
cache = True
cache_plugin = jsonfile
cache_connection = /tmp/inventory_cache
cache_timeout = 3600
```

---

## Real-World Scenarios

### Scenario: Multi-Cloud Inventory
```
inventory/
├── aws_ec2.yml          ← AWS instances
├── azure_rm.yml         ← Azure VMs
├── on_prem_hosts.ini    ← On-premises static
├── constructed.yml      ← Derived groups
└── group_vars/
    └── all.yml
```

```bash
# Ansible merges all sources
ansible-playbook -i inventory/ site.yml
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Plugin not detected | Check file naming convention and `enable_plugins` |
| Empty inventory from plugin | Verify credentials and API connectivity |
| Cache stale | Delete cache files or reduce `cache_timeout` |
| Constructed groups empty | Set `strict: false` and check Jinja2 expressions |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| Inventory plugin | YAML-configured dynamic inventory source |
| Constructed plugin | Builds groups/vars from existing inventory data |
| `keyed_groups` | Auto-create groups from host attributes |
| `compose` | Set computed variables per host |
| `groups` | Conditional group membership |
| Cache | Store inventory results to avoid repeated API calls |
| `enable_plugins` | Whitelist which plugins Ansible can use |

---

## Quick Revision Questions

1. **How do inventory plugins differ from legacy inventory scripts?**
   > Plugins are YAML-configured, support caching natively, and integrate with Ansible's plugin system. Scripts are executable files returning JSON.

2. **What is the constructed inventory plugin used for?**
   > Building new groups and computed variables from existing inventory data using Jinja2 expressions.

3. **How does Ansible determine which inventory plugin to use?**
   > By the file suffix (e.g., `*aws_ec2.yml`) and the `plugin:` key in the YAML file.

4. **How do you enable inventory caching?**
   > Set `cache: true` in the plugin config or `ansible.cfg`, along with `cache_plugin` and `cache_timeout`.

5. **Can you combine multiple inventory plugins?**
   > Yes. Place all plugin configs in a directory and use `-i inventory/`. Ansible merges all sources.

6. **What does `strict: false` do in constructed inventory?**
   > It tells the plugin to skip hosts where expressions fail (undefined variables) instead of raising errors.

---

[← Previous: Host Patterns](05-host-patterns.md) | [Next: Unit 4 - Playbook Structure →](../04-Playbooks/01-playbook-structure.md)
