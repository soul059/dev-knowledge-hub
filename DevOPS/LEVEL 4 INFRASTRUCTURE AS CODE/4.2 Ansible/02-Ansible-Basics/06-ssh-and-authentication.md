# Chapter 2.6: SSH and Authentication

[← Previous: ansible.cfg Configuration](05-ansible-cfg-configuration.md) | [Next: Unit 3 - Static Inventory →](../03-Inventory-Management/01-static-inventory.md)

---

## Overview

Ansible uses **SSH** as its primary transport mechanism for Linux/Unix hosts. Understanding SSH authentication, key management, and connection optimization is essential for reliable Ansible operations.

---

## SSH Authentication Flow

```
  ANSIBLE SSH AUTHENTICATION
  ══════════════════════════════════════════════════════════

  ┌──────────────┐                        ┌──────────────┐
  │ Control Node │                        │ Managed Node │
  │              │                        │              │
  │  ansible-    │   1. TCP Connect       │              │
  │  playbook    │──────────────────────► │  SSH Server  │
  │              │   (port 22)            │  (sshd)      │
  │              │                        │              │
  │  SSH Keys:   │   2. Key Exchange      │  Authorized  │
  │  ┌────────┐  │◄─────────────────────►│  Keys:       │
  │  │Private │  │   3. Authentication    │  ┌────────┐  │
  │  │Key     │  │──────────────────────► │  │Public  │  │
  │  └────────┘  │                        │  │Key     │  │
  │              │   4. Session           │  └────────┘  │
  │              │◄─────────────────────►│              │
  │              │   5. Execute tasks     │              │
  │              │──────────────────────► │              │
  │              │   6. Return results    │              │
  │              │◄──────────────────────│              │
  └──────────────┘                        └──────────────┘

  Authentication Methods (in preference order):
  ═══════════════════════════════════════════
  1. SSH Key (recommended)    ✅ Most secure, no password
  2. SSH Agent                ✅ Key in memory, convenient
  3. Password                 ⚠️  Less secure, use for initial setup
  4. Kerberos                 ✅ Enterprise environments
```

---

## SSH Key Setup

### Generate SSH Key Pair

```bash
# Generate a new SSH key pair
ssh-keygen -t ed25519 -C "ansible@control-node" -f ~/.ssh/ansible_key
# Or RSA (wider compatibility)
ssh-keygen -t rsa -b 4096 -C "ansible@control-node" -f ~/.ssh/ansible_key

# Files created:
# ~/.ssh/ansible_key       ← Private key (KEEP SECRET!)
# ~/.ssh/ansible_key.pub   ← Public key (copy to managed nodes)
```

### Copy Public Key to Managed Nodes

```bash
# Method 1: ssh-copy-id (easiest)
ssh-copy-id -i ~/.ssh/ansible_key.pub deploy@web1.example.com

# Method 2: Manual copy
cat ~/.ssh/ansible_key.pub | ssh deploy@web1.example.com \
  "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"

# Method 3: Using Ansible itself (bootstrap with password)
ansible all -m authorized_key -a \
  "user=deploy key='{{ lookup('file','~/.ssh/ansible_key.pub') }}'" \
  -k -b
```

### Configure SSH for Ansible

```bash
# ~/.ssh/config
Host web*
    User deploy
    IdentityFile ~/.ssh/ansible_key
    StrictHostKeyChecking no
    Port 22

Host db*
    User dbadmin
    IdentityFile ~/.ssh/ansible_db_key
    Port 2222
    ProxyJump bastion.example.com

Host bastion
    HostName bastion.example.com
    User admin
    IdentityFile ~/.ssh/bastion_key
```

---

## Authentication in Inventory

```ini
# inventory.ini

# SSH key authentication (recommended)
[webservers]
web1 ansible_host=192.168.1.10 ansible_user=deploy ansible_ssh_private_key_file=~/.ssh/ansible_key
web2 ansible_host=192.168.1.11 ansible_user=deploy ansible_ssh_private_key_file=~/.ssh/ansible_key

# Password authentication (initial setup only)
[new_servers]
new1 ansible_host=192.168.1.20 ansible_user=root ansible_password=initial_pass

# Group-level variables
[webservers:vars]
ansible_user=deploy
ansible_ssh_private_key_file=~/.ssh/ansible_key
ansible_become=yes
ansible_become_method=sudo

# Windows hosts (WinRM)
[windows]
win1 ansible_host=192.168.1.30 ansible_user=Administrator ansible_password=WinP@ss ansible_connection=winrm ansible_winrm_transport=ntlm
```

---

## SSH Agent

```
  SSH AGENT WORKFLOW
  ══════════════════════════════════════════════════════════

  ┌──────────────┐
  │   SSH Agent  │  ← Holds private keys in memory
  │   (process)  │
  │              │
  │  ┌────────┐  │
  │  │Key 1   │  │  ← ansible_key
  │  │Key 2   │  │  ← deploy_key
  │  │Key 3   │  │  ← bastion_key
  │  └────────┘  │
  └──────┬───────┘
         │
         │  Ansible requests key signing
         │
  ┌──────┴───────┐
  │   Ansible    │  ← Never touches private key directly
  │   Process    │
  └──────────────┘
```

```bash
# Start SSH agent
eval $(ssh-agent -s)

# Add keys to agent
ssh-add ~/.ssh/ansible_key
ssh-add ~/.ssh/deploy_key

# List loaded keys
ssh-add -l

# In ansible.cfg, enable agent forwarding:
# [ssh_connection]
# ssh_args = -o ForwardAgent=yes
```

---

## Privilege Escalation (become)

```
  PRIVILEGE ESCALATION FLOW
  ══════════════════════════════════════════════════════════

  ┌──────────────────────────────────────────────┐
  │  Step 1: SSH as 'deploy' user                │
  │  ┌────────────────────────────┐              │
  │  │ $ ssh deploy@web1          │              │
  │  │ deploy@web1:~$             │              │
  │  └────────────────────────────┘              │
  │                                               │
  │  Step 2: Escalate to 'root' via sudo          │
  │  ┌────────────────────────────┐              │
  │  │ deploy@web1:~$ sudo <cmd>  │              │
  │  │ root@web1:~#               │              │
  │  └────────────────────────────┘              │
  └──────────────────────────────────────────────┘
```

```yaml
# In playbook
- name: Configure server
  hosts: webservers
  become: yes              # Enable privilege escalation
  become_method: sudo      # Use sudo (default)
  become_user: root        # Become root (default)
  tasks:
    - name: Install nginx (needs root)
      apt:
        name: nginx
        state: present
```

```ini
# In ansible.cfg
[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False
```

```bash
# Sudoers configuration on managed nodes
# /etc/sudoers.d/ansible
deploy ALL=(ALL) NOPASSWD: ALL

# More restrictive (recommended for production)
deploy ALL=(ALL) NOPASSWD: /usr/bin/apt-get, /usr/bin/systemctl, /usr/sbin/service
```

---

## Jump/Bastion Hosts

```
  BASTION HOST / JUMP BOX PATTERN
  ══════════════════════════════════════════════════════════

  ┌──────────┐        ┌──────────┐        ┌──────────────┐
  │ Control  │  SSH   │ Bastion  │  SSH   │ Internal     │
  │ Node     │───────►│ Host     │───────►│ Servers      │
  │          │        │ (Public) │        │ (Private)    │
  │ Internet │        │          │        │ web1, db1    │
  └──────────┘        └──────────┘        └──────────────┘

  Firewall allows only Bastion → Internal
```

```ini
# ansible.cfg
[ssh_connection]
ssh_args = -C -o ControlMaster=auto -o ControlPersist=60s -o ProxyCommand="ssh -W %h:%p -q bastion.example.com"
```

```ini
# inventory.ini
[bastion]
bastion.example.com ansible_user=admin

[internal_servers]
web1 ansible_host=10.0.1.10
db1  ansible_host=10.0.1.20

[internal_servers:vars]
ansible_ssh_common_args='-o ProxyCommand="ssh -W %h:%p -q admin@bastion.example.com"'
```

---

## Windows Authentication (WinRM)

```
  WINDOWS CONNECTION
  ══════════════════════════════════════════════════════════

  ┌───────────────┐     WinRM (HTTPS/5986)    ┌────────────┐
  │ Control Node  │─────────────────────────►  │  Windows   │
  │ (Linux)       │     or HTTP/5985           │  Server    │
  │               │                            │            │
  │  pywinrm      │     Authentication:        │ PowerShell │
  │  library      │     • NTLM                 │            │
  │               │     • Kerberos              │            │
  │               │     • CredSSP               │            │
  └───────────────┘                            └────────────┘
```

```ini
# Windows hosts in inventory
[windows]
win_server1 ansible_host=192.168.1.100

[windows:vars]
ansible_user=Administrator
ansible_password=SecureP@ss123
ansible_connection=winrm
ansible_winrm_server_cert_validation=ignore
ansible_winrm_transport=ntlm
ansible_port=5986
```

```bash
# Install WinRM Python library
pip install pywinrm

# Enable WinRM on Windows (PowerShell as Admin)
# winrm quickconfig
# Enable-PSRemoting -Force
```

---

## Real-World Scenarios

### Scenario 1: Initial Server Bootstrap
```bash
# New servers only have root + password
# Step 1: Create deploy user with password auth
ansible new_servers -m user -a "name=deploy groups=sudo shell=/bin/bash" \
  -u root -k -b

# Step 2: Copy SSH key
ansible new_servers -m authorized_key \
  -a "user=deploy key='$(cat ~/.ssh/ansible_key.pub)'" \
  -u root -k -b

# Step 3: Disable password auth
ansible new_servers -m lineinfile \
  -a "path=/etc/ssh/sshd_config regexp='^PasswordAuthentication' line='PasswordAuthentication no'" \
  -u deploy --key-file ~/.ssh/ansible_key -b

# Step 4: Restart SSH
ansible new_servers -m service -a "name=sshd state=restarted" \
  -u deploy --key-file ~/.ssh/ansible_key -b
```

### Scenario 2: Multi-Key Environment
```bash
# ~/.ssh/config for different key per environment
Host dev-*
    IdentityFile ~/.ssh/dev_key
    User devops

Host staging-*
    IdentityFile ~/.ssh/staging_key
    User deploy

Host prod-*
    IdentityFile ~/.ssh/prod_key
    User ansible-svc
    ProxyJump bastion-prod
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| "Permission denied (publickey)" | Check key permissions (600), verify key is in authorized_keys |
| "Host key verification failed" | Set `host_key_checking = False` or add host key |
| "sudo: a password is required" | Add NOPASSWD to sudoers or use `-K` flag |
| "Connection timed out" | Check firewall, port 22, and network connectivity |
| SSH agent not forwarding keys | Add `-o ForwardAgent=yes` to ssh_args |
| WinRM connection refused | Verify WinRM is enabled and port 5986 is open |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| SSH Key Auth | Preferred authentication using public/private key pair |
| ssh-copy-id | Copy public key to remote authorized_keys |
| SSH Agent | Holds private keys in memory for convenience |
| become | Privilege escalation (sudo) for elevated tasks |
| Bastion Host | Jump host for accessing private networks |
| WinRM | Windows remote management protocol |
| host_key_checking | Whether to verify SSH host keys (disable in dev) |
| ControlPersist | Keep SSH connections alive for reuse |
| ForwardAgent | Forward local SSH agent to remote hosts |

---

## Quick Revision Questions

1. **What is the recommended authentication method for Ansible?**
   > SSH key-based authentication. It's more secure than passwords and doesn't require interactive input.

2. **How do you set up passwordless sudo for an Ansible user?**
   > Add `deploy ALL=(ALL) NOPASSWD: ALL` to `/etc/sudoers.d/ansible` on managed nodes.

3. **What is a bastion/jump host and why is it used?**
   > A publicly accessible server used as a gateway to reach private network servers. It limits the attack surface by requiring all SSH traffic to pass through one point.

4. **What is SSH ControlPersist and how does it help Ansible?**
   > It keeps SSH connections alive and reuses them for subsequent tasks, avoiding the overhead of establishing a new SSH connection for each task.

5. **How does Ansible connect to Windows hosts?**
   > Via WinRM (Windows Remote Management) protocol using the `pywinrm` Python library, typically over HTTPS (port 5986).

6. **What file permissions should SSH private keys have?**
   > 600 (owner read/write only). SSH refuses to use keys with broader permissions.

---

[← Previous: ansible.cfg Configuration](05-ansible-cfg-configuration.md) | [Next: Unit 3 - Static Inventory →](../03-Inventory-Management/01-static-inventory.md)
