# Chapter 6.2: Creating Roles

[← Previous: Role Structure and Concepts](01-role-structure.md) | [Next: Role Dependencies →](03-role-dependencies.md)

---

## Overview

This chapter walks through creating a role from scratch — from scaffolding the directory structure with `ansible-galaxy init` to writing tasks, handlers, templates, defaults, and wiring it all together in a playbook.

---

## Scaffolding with ansible-galaxy init

```bash
ansible-galaxy init roles/webserver
```

```
  roles/webserver/
  ├── defaults/
  │   └── main.yml
  ├── files/
  ├── handlers/
  │   └── main.yml
  ├── meta/
  │   └── main.yml
  ├── README.md
  ├── tasks/
  │   └── main.yml
  ├── templates/
  ├── tests/
  │   ├── inventory
  │   └── test.yml
  └── vars/
      └── main.yml
```

---

## Building a Complete Role: webserver

### 1. defaults/main.yml

```yaml
# roles/webserver/defaults/main.yml
---
http_port: 80
server_name: localhost
document_root: /var/www/html
max_clients: 256
worker_processes: auto
enable_ssl: false
ssl_port: 443
```

### 2. vars/main.yml

```yaml
# roles/webserver/vars/main.yml
---
nginx_packages:
  - nginx
  - nginx-common
nginx_service: nginx
nginx_conf_dir: /etc/nginx
nginx_user: www-data
```

### 3. tasks/main.yml

```yaml
# roles/webserver/tasks/main.yml
---
- name: Include OS-specific variables
  include_vars: "{{ ansible_os_family }}.yml"
  ignore_errors: yes

- name: Install Nginx packages
  apt:
    name: "{{ nginx_packages }}"
    state: present
    update_cache: yes
  when: ansible_os_family == "Debian"

- name: Install Nginx packages (RedHat)
  yum:
    name: nginx
    state: present
  when: ansible_os_family == "RedHat"

- name: Deploy Nginx configuration
  template:
    src: nginx.conf.j2
    dest: "{{ nginx_conf_dir }}/nginx.conf"
    owner: root
    group: root
    mode: '0644'
    validate: nginx -t -c %s
  notify: restart nginx

- name: Deploy site configuration
  template:
    src: site.conf.j2
    dest: "{{ nginx_conf_dir }}/sites-available/{{ server_name }}.conf"
    owner: root
    group: root
    mode: '0644'
  notify: reload nginx

- name: Enable site
  file:
    src: "{{ nginx_conf_dir }}/sites-available/{{ server_name }}.conf"
    dest: "{{ nginx_conf_dir }}/sites-enabled/{{ server_name }}.conf"
    state: link
  notify: reload nginx

- name: Create document root
  file:
    path: "{{ document_root }}"
    state: directory
    owner: "{{ nginx_user }}"
    group: "{{ nginx_user }}"
    mode: '0755'

- name: Deploy index page
  copy:
    src: index.html
    dest: "{{ document_root }}/index.html"
    owner: "{{ nginx_user }}"
    mode: '0644'

- name: Ensure Nginx is started and enabled
  service:
    name: "{{ nginx_service }}"
    state: started
    enabled: yes
```

### 4. handlers/main.yml

```yaml
# roles/webserver/handlers/main.yml
---
- name: restart nginx
  service:
    name: "{{ nginx_service }}"
    state: restarted

- name: reload nginx
  service:
    name: "{{ nginx_service }}"
    state: reloaded
```

### 5. templates/nginx.conf.j2

```jinja2
# {{ ansible_managed }}
user {{ nginx_user }};
worker_processes {{ worker_processes }};
pid /run/nginx.pid;

events {
    worker_connections {{ max_clients }};
}

http {
    sendfile on;
    tcp_nopush on;
    types_hash_max_size 2048;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

### 6. templates/site.conf.j2

```jinja2
# {{ ansible_managed }}
server {
    listen {{ http_port }};
    server_name {{ server_name }};
    root {{ document_root }};
    index index.html;

{% if enable_ssl %}
    listen {{ ssl_port }} ssl;
    ssl_certificate /etc/ssl/certs/{{ server_name }}.crt;
    ssl_certificate_key /etc/ssl/private/{{ server_name }}.key;
{% endif %}

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### 7. files/index.html

```html
<!DOCTYPE html>
<html>
<head><title>Welcome</title></head>
<body><h1>Server is running</h1></body>
</html>
```

### 8. meta/main.yml

```yaml
# roles/webserver/meta/main.yml
---
galaxy_info:
  author: your_name
  description: Installs and configures Nginx web server
  license: MIT
  min_ansible_version: "2.9"
  platforms:
    - name: Ubuntu
      versions:
        - focal
        - jammy
    - name: EL
      versions:
        - "8"
        - "9"
  galaxy_tags:
    - web
    - nginx

dependencies: []
```

---

## Splitting Tasks into Multiple Files

```yaml
# roles/webserver/tasks/main.yml
---
- import_tasks: install.yml
- import_tasks: configure.yml
- import_tasks: service.yml
- import_tasks: ssl.yml
  when: enable_ssl | bool
```

```
  tasks/
  ├── main.yml           ◄── Entry point — imports others
  ├── install.yml        ◄── Package installation
  ├── configure.yml      ◄── Config file deployment
  ├── service.yml        ◄── Service management
  └── ssl.yml            ◄── SSL-specific tasks
```

---

## OS-Specific Variables

```yaml
# roles/webserver/tasks/main.yml (first task)
- name: Include OS-specific vars
  include_vars: "{{ item }}"
  with_first_found:
    - "{{ ansible_distribution }}-{{ ansible_distribution_major_version }}.yml"
    - "{{ ansible_distribution }}.yml"
    - "{{ ansible_os_family }}.yml"
    - default.yml
```

```
  vars/
  ├── main.yml
  ├── Debian.yml          ◄── Debian family vars
  ├── RedHat.yml          ◄── RedHat family vars
  ├── Ubuntu-22.yml       ◄── Ubuntu 22 specific
  └── default.yml         ◄── Fallback
```

---

## Using the Role

```yaml
---
- name: Deploy web servers
  hosts: webservers
  become: yes
  roles:
    - role: webserver
      vars:
        server_name: app.example.com
        http_port: 80
        enable_ssl: true
        max_clients: 512
```

---

## Role Argument Validation (Ansible 2.11+)

```yaml
# roles/webserver/meta/argument_specs.yml
---
argument_specs:
  main:
    short_description: Configure Nginx web server
    options:
      http_port:
        type: int
        default: 80
        description: HTTP listen port
      server_name:
        type: str
        required: true
        description: The server hostname
      enable_ssl:
        type: bool
        default: false
        description: Enable SSL configuration
      document_root:
        type: path
        default: /var/www/html
        description: Web root directory
```

If a required argument is missing, Ansible raises an error before running tasks.

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| `ERROR! no action detected in task` | YAML indentation issue in `tasks/main.yml` |
| Template not found | Ensure template is in `templates/` directory, reference by filename only (no path prefix) |
| Variables not applied | Check you are using `defaults/` for overridable defaults |
| Handler not triggered | Verify `notify` name matches handler name exactly |
| Role tasks run but nothing happens | Check `when` conditions, ensure correct OS family |

---

## Summary Table

| Component | File | Purpose |
|-----------|------|---------|
| `defaults/main.yml` | Default vars (low precedence) | Intended to be overridden |
| `vars/main.yml` | Role vars (high precedence) | Fixed role internals |
| `tasks/main.yml` | Task entry point | Import/include sub-tasks |
| `handlers/main.yml` | Handlers | Service restarts/reloads |
| `templates/*.j2` | Jinja2 templates | Configuration files |
| `files/*` | Static files | Copied as-is |
| `meta/main.yml` | Metadata | Dependencies, Galaxy info |
| `meta/argument_specs.yml` | Arg validation | Input validation (2.11+) |

---

## Quick Revision Questions

1. **What command scaffolds a new role directory?**
   > `ansible-galaxy init <role_name>` creates the full standard directory structure.

2. **Where should templates be placed in a role?**
   > In the `templates/` directory. The `template` module automatically looks there when used in a role.

3. **How do you split a large `tasks/main.yml` into smaller files?**
   > Use `import_tasks: install.yml` or `include_tasks: install.yml` in `main.yml` to load sub-files.

4. **What is the purpose of `meta/argument_specs.yml`?**
   > It validates role arguments at runtime (Ansible 2.11+), ensuring required vars are provided and types are correct.

5. **How do you handle OS-specific differences in a role?**
   > Use `include_vars` with `with_first_found` to load OS-specific variable files from the `vars/` directory.

6. **What does `{{ ansible_managed }}` do in templates?**
   > It inserts a comment indicating the file is managed by Ansible, warning users not to edit it manually.

---

[← Previous: Role Structure and Concepts](01-role-structure.md) | [Next: Role Dependencies →](03-role-dependencies.md)
