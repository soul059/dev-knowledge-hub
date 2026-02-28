# Chapter 8.1: Package Modules

[← Previous: Managing Directories](../07-Templates-and-Files/05-managing-directories.md) | [Next: Service Modules →](02-service-modules.md)

---

## Overview

Package modules manage software installation, removal, and updates on managed hosts. Ansible provides both **OS-specific** modules (`apt`, `yum`, `dnf`) and a **generic** module (`package`) that abstracts the package manager.

---

## package Module (Generic)

```yaml
# Works on any supported OS — uses the native package manager
- name: Install Nginx
  package:
    name: nginx
    state: present
```

> **Note**: `package` auto-detects the OS package manager. Use it for simple install/remove tasks across mixed-OS inventory. For advanced options (repos, GPG keys), use the OS-specific module.

---

## apt Module (Debian/Ubuntu)

```yaml
# Install a package
- name: Install Nginx
  apt:
    name: nginx
    state: present

# Install specific version
- name: Install specific version
  apt:
    name: nginx=1.22.0-1ubuntu1
    state: present

# Install multiple packages
- name: Install web stack
  apt:
    name:
      - nginx
      - php-fpm
      - php-mysql
      - certbot
    state: present

# Update cache before install
- name: Install with cache update
  apt:
    name: nginx
    state: present
    update_cache: yes
    cache_valid_time: 3600    # Don't update if updated < 1 hour ago

# Update all packages
- name: Update all packages
  apt:
    upgrade: dist
    update_cache: yes

# Remove a package
- name: Remove package
  apt:
    name: apache2
    state: absent

# Remove with purge (config files too)
- name: Purge package
  apt:
    name: apache2
    state: absent
    purge: yes
    autoremove: yes

# Install from a .deb file
- name: Install local deb
  apt:
    deb: /tmp/package.deb
```

### apt Key Parameters

| Parameter | Description |
|-----------|-------------|
| `name` | Package name(s) |
| `state` | `present`, `absent`, `latest`, `fixed` |
| `update_cache` | Run `apt update` first |
| `cache_valid_time` | Skip update if cache is recent (seconds) |
| `upgrade` | `dist`, `full`, `safe`, `yes` |
| `purge` | Remove config files with package |
| `autoremove` | Remove unused dependencies |
| `deb` | Install from `.deb` file path/URL |
| `install_recommends` | Install recommended packages |

---

## apt_repository Module

```yaml
# Add a PPA
- name: Add Nginx PPA
  apt_repository:
    repo: ppa:nginx/stable
    state: present

# Add custom repository
- name: Add Docker repository
  apt_repository:
    repo: "deb [arch=amd64] https://download.docker.com/linux/ubuntu {{ ansible_distribution_release }} stable"
    state: present
    filename: docker

# Remove a repository
- name: Remove old repo
  apt_repository:
    repo: ppa:old/repo
    state: absent
```

### apt_key Module

```yaml
- name: Add Docker GPG key
  apt_key:
    url: https://download.docker.com/linux/ubuntu/gpg
    state: present
```

---

## yum Module (RHEL/CentOS 7)

```yaml
# Install a package
- name: Install httpd
  yum:
    name: httpd
    state: present

# Install multiple packages
- name: Install packages
  yum:
    name:
      - httpd
      - php
      - mariadb-server
    state: present

# Install specific version
- name: Install specific version
  yum:
    name: httpd-2.4.6-97.el7
    state: present

# Install from a URL
- name: Install RPM from URL
  yum:
    name: https://example.com/package.rpm
    state: present

# Install from local file
- name: Install local RPM
  yum:
    name: /tmp/package.rpm
    state: present
    disable_gpg_check: yes

# Update all packages
- name: Update all
  yum:
    name: '*'
    state: latest

# Remove a package
- name: Remove package
  yum:
    name: httpd
    state: absent

# Enable a specific repo for install
- name: Install from EPEL
  yum:
    name: nginx
    enablerepo: epel
    state: present

# Install a group
- name: Install Development Tools
  yum:
    name: "@Development Tools"
    state: present
```

---

## dnf Module (RHEL 8+/Fedora)

```yaml
# Install packages
- name: Install packages
  dnf:
    name:
      - nginx
      - python3
      - firewalld
    state: present

# Install with module stream
- name: Enable and install PHP 8.1
  dnf:
    name: '@php:8.1'
    state: present

# Update all
- name: Update all packages
  dnf:
    name: '*'
    state: latest

# Clean cache
- name: Clean DNF cache
  dnf:
    autoremove: yes
```

---

## pip Module (Python)

```yaml
# Install a Python package
- name: Install Flask
  pip:
    name: flask
    state: present

# Install specific version
- name: Install specific version
  pip:
    name: flask==2.3.0

# Install from requirements file
- name: Install requirements
  pip:
    requirements: /opt/myapp/requirements.txt
    virtualenv: /opt/myapp/venv

# Install in virtualenv
- name: Install in virtualenv
  pip:
    name:
      - flask
      - gunicorn
      - celery
    virtualenv: /opt/myapp/venv
    virtualenv_command: python3 -m venv

# Install with extra args
- name: Install with extra index
  pip:
    name: private-package
    extra_args: "--index-url https://pypi.internal.company.com/simple"
```

---

## Cross-Platform Pattern

```yaml
# Using variables for package name differences
# group_vars/Debian.yml
packages:
  web: nginx
  db: mariadb-server
  firewall: ufw

# group_vars/RedHat.yml
packages:
  web: nginx
  db: mariadb-server
  firewall: firewalld
```

```yaml
- name: Include OS variables
  include_vars: "{{ ansible_os_family }}.yml"

- name: Install packages
  package:
    name: "{{ packages[item] }}"
    state: present
  loop:
    - web
    - db
    - firewall
```

---

## Real-World Scenario: Full Stack Installation

```yaml
---
- name: Install application stack
  hosts: app_servers
  become: yes
  tasks:
    - name: Update package cache (Debian)
      apt:
        update_cache: yes
        cache_valid_time: 3600
      when: ansible_os_family == "Debian"

    - name: Install system packages
      package:
        name:
          - git
          - curl
          - unzip
          - htop
        state: present

    - name: Install Nginx
      package:
        name: nginx
        state: present
      notify: start nginx

    - name: Install Python and venv
      package:
        name:
          - python3
          - python3-pip
          - python3-venv
        state: present

    - name: Install application dependencies
      pip:
        requirements: /opt/myapp/requirements.txt
        virtualenv: /opt/myapp/venv
        virtualenv_command: python3 -m venv

    - name: Ensure no unwanted packages
      package:
        name:
          - telnet
          - rsh
        state: absent

  handlers:
    - name: start nginx
      service:
        name: nginx
        state: started
        enabled: yes
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| `No package matching 'x' found` | Check package name; differs between OS families |
| `apt` cache stale | Add `update_cache: yes` |
| GPG key error | Import key with `apt_key` or `rpm_key` first |
| Version not available | Check with `apt-cache policy` or `yum info` |
| `pip` not found | Install `python3-pip` first |
| `package` module fails | Use OS-specific module for advanced features |
| Virtualenv creation fails | Install `python3-venv` package |

---

## Summary Table

| Module | OS Family | Key Feature |
|--------|-----------|-------------|
| `package` | Any | Generic, auto-detects manager |
| `apt` | Debian/Ubuntu | Cache update, deb install, purge |
| `yum` | RHEL/CentOS 7 | Repo enable, groups |
| `dnf` | RHEL 8+/Fedora | Module streams |
| `pip` | Any (Python) | Virtualenv, requirements |
| `apt_repository` | Debian | Manage APT repos |
| `apt_key` | Debian | Manage GPG keys |

---

## Quick Revision Questions

1. **What is the difference between the `package` and `apt` modules?**
   > `package` is generic and auto-detects the OS package manager. `apt` is Debian-specific and supports advanced features like cache management, purge, and deb file installs.

2. **How do you avoid running `apt update` on every playbook run?**
   > Set `cache_valid_time: 3600` — the cache is only updated if it's older than the specified seconds.

3. **What does `state: latest` do?**
   > It installs the package if missing, or upgrades it to the latest available version if already installed.

4. **How do you install Python packages in a virtual environment?**
   > Use the `pip` module with `virtualenv: /path/to/venv` and optionally `virtualenv_command: python3 -m venv`.

5. **How do you handle different package names across OS families?**
   > Use `include_vars` to load OS-specific variable files that map generic names to OS-specific package names.

---

[← Previous: Managing Directories](../07-Templates-and-Files/05-managing-directories.md) | [Next: Service Modules →](02-service-modules.md)
