# Ansible - Comprehensive Study Notes

> **Infrastructure as Code | Configuration Management | Automation**

```
    ╔══════════════════════════════════════════════════════════════╗
    ║                    A N S I B L E                            ║
    ║              Configuration Management &                     ║
    ║                IT Automation Tool                           ║
    ╠══════════════════════════════════════════════════════════════╣
    ║                                                              ║
    ║   ┌──────────┐    SSH/WinRM    ┌──────────────────────┐     ║
    ║   │ Control  │───────────────► │   Managed Nodes      │     ║
    ║   │  Node    │  Agentless      │  ┌────┐ ┌────┐      │     ║
    ║   │          │  Push Model     │  │Web │ │ DB │ ...   │     ║
    ║   └──────────┘                 │  └────┘ └────┘      │     ║
    ║        │                       └──────────────────────┘     ║
    ║   ┌────┴─────┐                                              ║
    ║   │Playbooks │                                              ║
    ║   │Inventory │                                              ║
    ║   │Roles     │                                              ║
    ║   └──────────┘                                              ║
    ╚══════════════════════════════════════════════════════════════╝
```

---

## 📋 Course Overview

These study notes provide a **comprehensive guide to Ansible**, covering everything from fundamental configuration management concepts to advanced features and best practices. The notes are designed for:

- **DevOps Engineers** looking to automate infrastructure
- **System Administrators** transitioning to IaC
- **Students** preparing for Ansible certifications
- **Developers** wanting to understand deployment automation

### What You'll Learn

- Configuration management principles and the push model
- Writing and organizing Ansible playbooks
- Managing inventory, variables, and secrets
- Creating reusable roles and templates
- Working with modules for cloud, containers, and networking
- Advanced plugins, collections, and testing
- Production-ready best practices

---

## 📚 Complete Table of Contents

### [Unit 1: Configuration Management Concepts](01-Configuration-Management-Concepts/)

| # | Chapter | File |
|---|---------|------|
| 1.1 | What is Configuration Management? | [01-what-is-configuration-management.md](01-Configuration-Management-Concepts/01-what-is-configuration-management.md) |
| 1.2 | Push vs Pull Model | [02-push-vs-pull-model.md](01-Configuration-Management-Concepts/02-push-vs-pull-model.md) |
| 1.3 | Idempotency | [03-idempotency.md](01-Configuration-Management-Concepts/03-idempotency.md) |
| 1.4 | CM Tools Comparison | [04-cm-tools-comparison.md](01-Configuration-Management-Concepts/04-cm-tools-comparison.md) |
| 1.5 | Ansible Overview and Architecture | [05-ansible-overview-and-architecture.md](01-Configuration-Management-Concepts/05-ansible-overview-and-architecture.md) |

### [Unit 2: Ansible Basics](02-Ansible-Basics/)

| # | Chapter | File |
|---|---------|------|
| 2.1 | Installation | [01-installation.md](02-Ansible-Basics/01-installation.md) |
| 2.2 | Inventory Basics | [02-inventory-basics.md](02-Ansible-Basics/02-inventory-basics.md) |
| 2.3 | Ad-hoc Commands | [03-ad-hoc-commands.md](02-Ansible-Basics/03-ad-hoc-commands.md) |
| 2.4 | Modules Overview | [04-modules-overview.md](02-Ansible-Basics/04-modules-overview.md) |
| 2.5 | ansible.cfg Configuration | [05-ansible-cfg-configuration.md](02-Ansible-Basics/05-ansible-cfg-configuration.md) |
| 2.6 | SSH and Authentication | [06-ssh-and-authentication.md](02-Ansible-Basics/06-ssh-and-authentication.md) |

### [Unit 3: Inventory Management](03-Inventory-Management/)

| # | Chapter | File |
|---|---------|------|
| 3.1 | Static Inventory | [01-static-inventory.md](03-Inventory-Management/01-static-inventory.md) |
| 3.2 | Dynamic Inventory | [02-dynamic-inventory.md](03-Inventory-Management/02-dynamic-inventory.md) |
| 3.3 | Inventory Variables | [03-inventory-variables.md](03-Inventory-Management/03-inventory-variables.md) |
| 3.4 | Groups and Group Variables | [04-groups-and-group-variables.md](03-Inventory-Management/04-groups-and-group-variables.md) |
| 3.5 | Host Patterns | [05-host-patterns.md](03-Inventory-Management/05-host-patterns.md) |
| 3.6 | Inventory Plugins | [06-inventory-plugins.md](03-Inventory-Management/06-inventory-plugins.md) |

### [Unit 4: Playbooks](04-Playbooks/)

| # | Chapter | File |
|---|---------|------|
| 4.1 | Playbook Structure | [01-playbook-structure.md](04-Playbooks/01-playbook-structure.md) |
| 4.2 | Plays and Tasks | [02-plays-and-tasks.md](04-Playbooks/02-plays-and-tasks.md) |
| 4.3 | Handlers and Notify | [03-handlers-and-notify.md](04-Playbooks/03-handlers-and-notify.md) |
| 4.4 | Variables in Playbooks | [04-variables-in-playbooks.md](04-Playbooks/04-variables-in-playbooks.md) |
| 4.5 | Conditionals (when) | [05-conditionals-when.md](04-Playbooks/05-conditionals-when.md) |
| 4.6 | Loops | [06-loops.md](04-Playbooks/06-loops.md) |
| 4.7 | Blocks and Error Handling | [07-blocks-and-error-handling.md](04-Playbooks/07-blocks-and-error-handling.md) |
| 4.8 | Tags | [08-tags.md](04-Playbooks/08-tags.md) |

### [Unit 5: Variables and Facts](05-Variables-and-Facts/)

| # | Chapter | File |
|---|---------|------|
| 5.1 | Variable Types | [01-variable-types.md](05-Variables-and-Facts/01-variable-types.md) |
| 5.2 | Variable Precedence | [02-variable-precedence.md](05-Variables-and-Facts/02-variable-precedence.md) |
| 5.3 | Defining Variables | [03-defining-variables.md](05-Variables-and-Facts/03-defining-variables.md) |
| 5.4 | Facts Gathering | [04-facts-gathering.md](05-Variables-and-Facts/04-facts-gathering.md) |
| 5.5 | Custom Facts | [05-custom-facts.md](05-Variables-and-Facts/05-custom-facts.md) |
| 5.6 | Magic Variables | [06-magic-variables.md](05-Variables-and-Facts/06-magic-variables.md) |
| 5.7 | Vault for Secrets | [07-vault-for-secrets.md](05-Variables-and-Facts/07-vault-for-secrets.md) |

### [Unit 6: Roles](06-Roles/)

| # | Chapter | File |
|---|---------|------|
| 6.1 | Role Structure | [01-role-structure.md](06-Roles/01-role-structure.md) |
| 6.2 | Creating Roles | [02-creating-roles.md](06-Roles/02-creating-roles.md) |
| 6.3 | Using Roles | [03-using-roles.md](06-Roles/03-using-roles.md) |
| 6.4 | Role Dependencies | [04-role-dependencies.md](06-Roles/04-role-dependencies.md) |
| 6.5 | Role Defaults vs Vars | [05-role-defaults-vs-vars.md](06-Roles/05-role-defaults-vs-vars.md) |
| 6.6 | Ansible Galaxy | [06-ansible-galaxy.md](06-Roles/06-ansible-galaxy.md) |

### [Unit 7: Templates and Files](07-Templates-and-Files/)

| # | Chapter | File |
|---|---------|------|
| 7.1 | Jinja2 Templates | [01-jinja2-templates.md](07-Templates-and-Files/01-jinja2-templates.md) |
| 7.2 | Template Module | [02-template-module.md](07-Templates-and-Files/02-template-module.md) |
| 7.3 | File Management Modules | [03-file-management-modules.md](07-Templates-and-Files/03-file-management-modules.md) |
| 7.4 | Line in File Operations | [04-line-in-file-operations.md](07-Templates-and-Files/04-line-in-file-operations.md) |
| 7.5 | Synchronize Module | [05-synchronize-module.md](07-Templates-and-Files/05-synchronize-module.md) |

### [Unit 8: Ansible Modules](08-Ansible-Modules/)

| # | Chapter | File |
|---|---------|------|
| 8.1 | Common Modules | [01-common-modules.md](08-Ansible-Modules/01-common-modules.md) |
| 8.2 | Cloud Modules (AWS, Azure, GCP) | [02-cloud-modules.md](08-Ansible-Modules/02-cloud-modules.md) |
| 8.3 | Container Modules (Docker, K8s) | [03-container-modules.md](08-Ansible-Modules/03-container-modules.md) |
| 8.4 | Network Modules | [04-network-modules.md](08-Ansible-Modules/04-network-modules.md) |
| 8.5 | Custom Modules | [05-custom-modules.md](08-Ansible-Modules/05-custom-modules.md) |

### [Unit 9: Advanced Features](09-Advanced-Features/)

| # | Chapter | File |
|---|---------|------|
| 9.1 | Ansible Collections | [01-ansible-collections.md](09-Advanced-Features/01-ansible-collections.md) |
| 9.2 | Callback Plugins | [02-callback-plugins.md](09-Advanced-Features/02-callback-plugins.md) |
| 9.3 | Filter Plugins | [03-filter-plugins.md](09-Advanced-Features/03-filter-plugins.md) |
| 9.4 | Lookup Plugins | [04-lookup-plugins.md](09-Advanced-Features/04-lookup-plugins.md) |
| 9.5 | Ansible Tower/AWX | [05-ansible-tower-awx.md](09-Advanced-Features/05-ansible-tower-awx.md) |
| 9.6 | Molecule for Testing | [06-molecule-for-testing.md](09-Advanced-Features/06-molecule-for-testing.md) |

### [Unit 10: Best Practices](10-Best-Practices/)

| # | Chapter | File |
|---|---------|------|
| 10.1 | Directory Structure | [01-directory-structure.md](10-Best-Practices/01-directory-structure.md) |
| 10.2 | Naming Conventions | [02-naming-conventions.md](10-Best-Practices/02-naming-conventions.md) |
| 10.3 | Idempotency Guidelines | [03-idempotency-guidelines.md](10-Best-Practices/03-idempotency-guidelines.md) |
| 10.4 | Security Practices | [04-security-practices.md](10-Best-Practices/04-security-practices.md) |
| 10.5 | Performance Optimization | [05-performance-optimization.md](10-Best-Practices/05-performance-optimization.md) |
| 10.6 | CI/CD Integration | [06-cicd-integration.md](10-Best-Practices/06-cicd-integration.md) |
| 10.7 | Testing Playbooks | [07-testing-playbooks.md](10-Best-Practices/07-testing-playbooks.md) |

---

## 🗂 Quick Reference

| Topic | Key Command |
|-------|-------------|
| Check version | `ansible --version` |
| Ping all hosts | `ansible all -m ping` |
| Run playbook | `ansible-playbook site.yml` |
| Dry run | `ansible-playbook site.yml --check` |
| List hosts | `ansible all --list-hosts` |
| Encrypt file | `ansible-vault encrypt secrets.yml` |
| Install role | `ansible-galaxy install geerlingguy.docker` |
| Syntax check | `ansible-playbook site.yml --syntax-check` |

---

## 🛠 Prerequisites

- Linux/macOS terminal (or WSL on Windows)
- Python 3.8+
- SSH access to target machines
- Basic understanding of YAML syntax
- Familiarity with Linux system administration

---

## 📖 How to Use These Notes

1. **Sequential Study**: Follow units in order for a structured learning path
2. **Quick Reference**: Jump to specific topics using the Table of Contents
3. **Hands-On Practice**: Try all configuration examples in a lab environment
4. **Revision**: Use summary tables and quick questions at the end of each chapter

---

*Last updated: February 2026*
