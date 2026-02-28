# Chapter 2.1: Installation

[← Previous: Ansible Overview](../01-Configuration-Management-Concepts/05-ansible-overview-and-architecture.md) | [Next: Inventory Basics →](02-inventory-basics.md)

---

## Overview

Ansible can be installed on most Linux distributions, macOS, and Windows (via WSL). This chapter covers installation methods across platforms and verifying the installation.

---

## Installation Methods

```
  ANSIBLE INSTALLATION OPTIONS
  ══════════════════════════════════════════════════════════

  ┌──────────────────────────────────────────────────────┐
  │                                                      │
  │   ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │
  │   │   pip    │  │  Package │  │   Container      │  │
  │   │ (Python) │  │  Manager │  │   (Docker/EE)    │  │
  │   │          │  │  (apt,   │  │                  │  │
  │   │ ✅ Latest│  │   yum,   │  │ ✅ Isolated      │  │
  │   │ ✅ Any OS│  │   brew)  │  │ ✅ Reproducible  │  │
  │   │          │  │          │  │                  │  │
  │   └──────────┘  └──────────┘  └──────────────────┘  │
  │                                                      │
  │         ▲               ▲                            │
  │         │               │                            │
  │    Recommended     OS-specific                       │
  │    (most control)  (easiest)                         │
  └──────────────────────────────────────────────────────┘
```

---

## Installation on Ubuntu/Debian

```bash
# Method 1: Using apt (OS package manager)
sudo apt update
sudo apt install -y software-properties-common
sudo apt-add-repository --yes --update ppa:ansible/ansible
sudo apt install -y ansible

# Method 2: Using pip (recommended for latest version)
sudo apt install -y python3 python3-pip
pip3 install ansible

# Method 3: Using pip with virtual environment (best practice)
sudo apt install -y python3 python3-venv
python3 -m venv ~/ansible-venv
source ~/ansible-venv/bin/activate
pip install ansible
```

## Installation on RHEL/CentOS/Fedora

```bash
# RHEL 8/9 with subscription
sudo subscription-manager repos --enable ansible-automation-platform
sudo dnf install -y ansible-core

# Fedora
sudo dnf install -y ansible

# CentOS Stream
sudo dnf install -y epel-release
sudo dnf install -y ansible

# Using pip
sudo dnf install -y python3 python3-pip
pip3 install ansible
```

## Installation on macOS

```bash
# Using Homebrew
brew install ansible

# Using pip
pip3 install ansible

# Using pip with virtual environment
python3 -m venv ~/ansible-venv
source ~/ansible-venv/bin/activate
pip install ansible
```

## Installation on Windows (via WSL)

```powershell
# Step 1: Install WSL (PowerShell as Admin)
wsl --install

# Step 2: Open WSL (Ubuntu) terminal
# Step 3: Inside WSL, install Ansible
sudo apt update
sudo apt install -y python3 python3-pip
pip3 install ansible
```

> **Note**: Ansible cannot run natively on Windows as a control node. Always use WSL.

---

## Ansible Packages Explained

```
  ANSIBLE PACKAGE STRUCTURE
  ══════════════════════════════════════════════════════════

  ┌───────────────────────────────────────────┐
  │              ansible (full)               │
  │                                           │
  │  ┌─────────────────────────────────────┐  │
  │  │         ansible-core                │  │
  │  │                                     │  │
  │  │  • ansible-playbook                 │  │
  │  │  • ansible-galaxy                   │  │
  │  │  • ansible-vault                    │  │
  │  │  • ansible-doc                      │  │
  │  │  • Core modules (~70)               │  │
  │  │  • Plugin framework                 │  │
  │  └─────────────────────────────────────┘  │
  │                                           │
  │  + Community Collections:                 │
  │    • amazon.aws                           │
  │    • community.general                    │
  │    • community.docker                     │
  │    • ... 85+ collections                  │
  │                                           │
  └───────────────────────────────────────────┘

  pip install ansible-core    → Minimal (just core)
  pip install ansible         → Full (core + collections)
```

---

## Verifying Installation

```bash
# Check Ansible version
ansible --version

# Expected output:
# ansible [core 2.16.x]
#   config file = /etc/ansible/ansible.cfg
#   configured module search path = ['~/.ansible/plugins/modules']
#   ansible python module location = /usr/lib/python3/dist-packages/ansible
#   ansible collection location = ~/.ansible/collections
#   executable location = /usr/bin/ansible
#   python version = 3.10.x
#   jinja version = 3.1.x
#   libyaml = True

# Check available modules
ansible-doc --list | head -20

# Check specific module documentation
ansible-doc ping
ansible-doc apt

# Test basic connectivity (localhost)
ansible localhost -m ping

# Expected output:
# localhost | SUCCESS => {
#     "changed": false,
#     "ping": "pong"
# }
```

---

## Post-Installation Setup

```bash
# Create project directory structure
mkdir -p ~/ansible-project
cd ~/ansible-project

# Create basic inventory file
cat > inventory.ini << 'EOF'
[local]
localhost ansible_connection=local

[webservers]
web1.example.com
web2.example.com

[dbservers]
db1.example.com
EOF

# Create ansible.cfg
cat > ansible.cfg << 'EOF'
[defaults]
inventory = ./inventory.ini
remote_user = deploy
host_key_checking = False
forks = 10
timeout = 30

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False
EOF

# Test setup
ansible all --list-hosts
ansible local -m ping
```

---

## Upgrading Ansible

```bash
# Using pip
pip install --upgrade ansible

# Using apt
sudo apt update && sudo apt upgrade ansible

# Check current version
ansible --version

# Check for specific version
pip install ansible==9.0.0
```

---

## Virtual Environment Best Practice

```
  WHY USE A VIRTUAL ENVIRONMENT?
  ══════════════════════════════════════════════════

  System Python                Virtual Environment
  ┌──────────────────┐        ┌──────────────────┐
  │  Python 3.10     │        │  Python 3.10     │
  │                  │        │  (isolated)      │
  │  ansible 2.14    │        │                  │
  │  boto3 1.24  ◄───┤ Conflict! │  ansible 2.17│
  │  requests 2.28   │        │  boto3 1.34      │
  │  ... system pkgs │        │  requests 2.31   │
  │                  │        │  ... project pkgs│
  └──────────────────┘        └──────────────────┘
       ❌ Conflicts              ✅ Isolated
```

```bash
# Create virtual environment
python3 -m venv ~/ansible-env

# Activate it
source ~/ansible-env/bin/activate    # Linux/macOS
# .\ansible-env\Scripts\Activate     # Windows (WSL)

# Install Ansible
pip install ansible ansible-lint molecule

# Deactivate when done
deactivate

# Pro tip: Add to .bashrc
echo 'alias ansible-activate="source ~/ansible-env/bin/activate"' >> ~/.bashrc
```

---

## Real-World Scenarios

### Scenario 1: Team Setup
Every team member uses the same Ansible version:
```bash
# requirements.txt in project repo
ansible==9.2.0
ansible-lint==24.2.0
molecule==24.2.0
jmespath==1.0.1

# Each team member runs:
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### Scenario 2: Multiple Ansible Versions
Working on projects requiring different versions:
```bash
# Project A needs Ansible 8
python3 -m venv ~/venv-ansible8
source ~/venv-ansible8/bin/activate
pip install ansible==8.7.0

# Project B needs Ansible 9
python3 -m venv ~/venv-ansible9
source ~/venv-ansible9/bin/activate
pip install ansible==9.2.0
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| `ansible: command not found` | Add pip install path to `$PATH` or use venv |
| `No module named ansible` | Ensure correct Python/pip version is used |
| Permission denied on pip install | Use `--user` flag or virtual environment |
| Old version despite upgrade | Check if system and pip versions conflict |
| WSL SSH issues | Install openssh-client: `sudo apt install openssh-client` |
| Python version mismatch | Check with `python3 --version`, install Python 3.8+ |

---

## Summary Table

| Method | Command | Best For |
|--------|---------|----------|
| pip (system) | `pip3 install ansible` | Quick setup |
| pip (venv) | `python3 -m venv` + `pip install` | Production, teams |
| apt (Ubuntu) | `sudo apt install ansible` | Ubuntu servers |
| dnf (RHEL) | `sudo dnf install ansible` | RHEL/CentOS |
| brew (macOS) | `brew install ansible` | macOS users |
| WSL (Windows) | Install WSL + pip inside | Windows users |
| ansible-core | `pip install ansible-core` | Minimal install |
| ansible (full) | `pip install ansible` | Full with collections |

---

## Quick Revision Questions

1. **Can Ansible be installed natively on Windows as a control node?**
   > No. Windows is not supported as a native control node. Use WSL (Windows Subsystem for Linux) instead.

2. **What is the difference between `ansible-core` and `ansible` pip packages?**
   > `ansible-core` includes only the engine and ~70 core modules. `ansible` includes core plus 85+ community collections.

3. **Why is a Python virtual environment recommended for Ansible?**
   > It isolates Ansible and its dependencies from the system Python, preventing version conflicts and ensuring reproducibility.

4. **How do you verify Ansible is installed correctly?**
   > Run `ansible --version` to see version details and `ansible localhost -m ping` to test basic functionality.

5. **What are the minimum requirements on a managed Linux node?**
   > Python (2.7+ or 3.5+) and an SSH server. No Ansible installation is needed on managed nodes.

6. **How do you ensure all team members use the same Ansible version?**
   > Create a `requirements.txt` in the project repository and have everyone install from it in a virtual environment.

---

[← Previous: Ansible Overview](../01-Configuration-Management-Concepts/05-ansible-overview-and-architecture.md) | [Next: Inventory Basics →](02-inventory-basics.md)
