# Chapter 4.4: Variables in Playbooks

[← Previous: Handlers and Notify](03-handlers-and-notify.md) | [Next: Conditionals (when) →](05-conditionals-when.md)

---

## Overview

Variables in Ansible make playbooks flexible and reusable. Instead of hardcoding values, you can define variables at multiple levels and override them as needed. Variables use **Jinja2** templating with `{{ variable_name }}` syntax.

---

## Variable Definition Locations

```
  WHERE TO DEFINE VARIABLES (Precedence: low → high)
  ══════════════════════════════════════════════════════════

  LOWEST PRIORITY
  ┌─────────────────────────────────────────────────┐
  │  1. Role defaults        (roles/x/defaults/)    │
  │  2. Group vars (inventory) (group_vars/)        │
  │  3. Host vars (inventory)  (host_vars/)         │
  │  4. Play vars              (vars:)              │
  │  5. Play vars_files        (vars_files:)        │
  │  6. Play vars_prompt       (vars_prompt:)       │
  │  7. Task vars              (vars: on task)      │
  │  8. Block vars             (vars: on block)     │
  │  9. Role vars              (roles/x/vars/)      │
  │  10. set_fact / register                        │
  │  11. include_vars                               │
  │  12. Extra vars (-e)       ALWAYS WINS          │
  └─────────────────────────────────────────────────┘
  HIGHEST PRIORITY
```

---

## Defining Variables in Plays

### Inline vars

```yaml
---
- name: Configure web server
  hosts: webservers
  vars:
    http_port: 80
    https_port: 443
    document_root: /var/www/html
    server_name: example.com
    packages:
      - nginx
      - certbot
      - python3-certbot-nginx
  tasks:
    - name: Install packages
      apt:
        name: "{{ packages }}"
        state: present

    - name: Set up document root
      file:
        path: "{{ document_root }}"
        state: directory
        mode: '0755'
```

### vars_files

```yaml
---
- name: Deploy application
  hosts: appservers
  vars_files:
    - vars/common.yml
    - vars/{{ env }}.yml       # Dynamic file name
    - vars/secrets.yml          # Vault-encrypted file
  tasks:
    - name: Configure app
      template:
        src: app.conf.j2
        dest: /etc/myapp/config.yml
```

```yaml
# vars/common.yml
app_name: myapp
app_user: appuser
app_group: appgroup
log_level: info

# vars/production.yml
env: production
debug: false
replicas: 3
db_host: db-prod.internal

# vars/staging.yml
env: staging
debug: true
replicas: 1
db_host: db-staging.internal
```

### vars_prompt

```yaml
---
- name: Deploy with user input
  hosts: webservers
  vars_prompt:
    - name: app_version
      prompt: "Which version to deploy?"
      default: "latest"
      private: no              # Show typed input

    - name: db_password
      prompt: "Enter database password"
      private: yes             # Hide typed input (default)
      encrypt: sha512_crypt    # Optional encryption
      confirm: yes             # Ask twice to confirm
  tasks:
    - name: Deploy version
      debug:
        msg: "Deploying {{ app_version }}"
```

---

## Using Variables

### Basic Substitution

```yaml
tasks:
  - name: Create user
    user:
      name: "{{ username }}"
      shell: "{{ user_shell | default('/bin/bash') }}"
      groups: "{{ user_groups }}"

  # IMPORTANT: Quotes required when variable starts a value
  - name: This works
    debug:
      msg: "{{ my_var }}"      # ✅ Quoted

  - name: This also works
    debug:
      msg: The value is {{ my_var }}  # ✅ Not at start

  # - name: This FAILS
  #   debug:
  #     msg: {{ my_var }}      # ❌ Unquoted at start = YAML error
```

### Variable Data Types

```yaml
vars:
  # String
  hostname: web01.example.com

  # Integer
  http_port: 80

  # Boolean
  enable_ssl: true

  # List
  dns_servers:
    - 8.8.8.8
    - 8.8.4.4

  # Dictionary
  user_config:
    name: appuser
    uid: 1001
    shell: /bin/bash
    groups:
      - web
      - deploy

  # Multi-line string
  welcome_message: |
    Welcome to {{ hostname }}
    Managed by Ansible
    Environment: {{ env }}
```

### Accessing Complex Variables

```yaml
vars:
  users:
    alice:
      uid: 1001
      role: admin
    bob:
      uid: 1002
      role: developer

  servers:
    - name: web01
      ip: 10.0.1.1
    - name: web02
      ip: 10.0.1.2

tasks:
  # Dictionary access - dot notation
  - debug:
      msg: "Alice UID: {{ users.alice.uid }}"

  # Dictionary access - bracket notation (safer)
  - debug:
      msg: "Alice UID: {{ users['alice']['uid'] }}"

  # List access by index
  - debug:
      msg: "First server: {{ servers[0].name }}"

  # Bracket notation avoids conflicts with Python methods
  - debug:
      msg: "{{ mydict['keys'] }}"     # Access key named 'keys'
  # vs
  # msg: "{{ mydict.keys }}"          # Could be Python .keys() method!
```

---

## Register Variables

Capture task output for use in later tasks:

```yaml
tasks:
  - name: Check if file exists
    stat:
      path: /etc/myapp/config.yml
    register: config_file

  - name: Create config if missing
    template:
      src: config.yml.j2
      dest: /etc/myapp/config.yml
    when: not config_file.stat.exists

  - name: Get running services
    command: systemctl list-units --type=service --state=running
    register: running_services
    changed_when: false

  - name: Show running services
    debug:
      msg: "{{ running_services.stdout_lines }}"
```

---

## set_fact Module

Create or modify variables during play execution:

```yaml
tasks:
  - name: Set simple fact
    set_fact:
      app_path: /opt/myapp
      app_version: "2.1.0"

  - name: Set computed fact
    set_fact:
      full_path: "{{ app_path }}/releases/{{ app_version }}"

  - name: Set fact from command output
    command: hostname -f
    register: fqdn_result
    changed_when: false

  - name: Store FQDN as fact
    set_fact:
      server_fqdn: "{{ fqdn_result.stdout }}"
      cacheable: yes    # Persist across plays if fact caching enabled
```

---

## Extra Variables (CLI)

Extra vars have the **highest precedence** and override everything:

```bash
# Single variable
ansible-playbook deploy.yml -e "version=2.1.0"

# Multiple variables
ansible-playbook deploy.yml -e "version=2.1.0 env=production"

# JSON format
ansible-playbook deploy.yml -e '{"version": "2.1.0", "env": "production"}'

# From a file
ansible-playbook deploy.yml -e @vars/deploy_vars.yml

# Override boolean
ansible-playbook deploy.yml -e "debug=true"
```

---

## Jinja2 Filters

Filters transform variable values:

```yaml
vars:
  my_string: "Hello World"
  my_list: [3, 1, 4, 1, 5, 9]
  my_path: "/etc/nginx/nginx.conf"

tasks:
  # String filters
  - debug:
      msg: "{{ my_string | upper }}"          # HELLO WORLD
  - debug:
      msg: "{{ my_string | lower }}"          # hello world
  - debug:
      msg: "{{ my_string | replace('World', 'Ansible') }}"

  # Default values
  - debug:
      msg: "{{ undefined_var | default('fallback') }}"

  # List filters
  - debug:
      msg: "{{ my_list | sort }}"             # [1, 1, 3, 4, 5, 9]
  - debug:
      msg: "{{ my_list | unique }}"           # [3, 1, 4, 5, 9]
  - debug:
      msg: "{{ my_list | min }}"              # 1
  - debug:
      msg: "{{ my_list | max }}"              # 9
  - debug:
      msg: "{{ my_list | length }}"           # 6

  # Path filters
  - debug:
      msg: "{{ my_path | basename }}"         # nginx.conf
  - debug:
      msg: "{{ my_path | dirname }}"          # /etc/nginx

  # Type conversion
  - debug:
      msg: "{{ '8080' | int }}"               # 8080 as integer
  - debug:
      msg: "{{ my_list | to_json }}"          # JSON string
  - debug:
      msg: "{{ my_list | to_yaml }}"          # YAML string

  # Regex
  - debug:
      msg: "{{ 'server123' | regex_search('\\d+') }}"  # 123
```

### Common Useful Filters

```
  ┌──────────────────────┬──────────────────────────────────┐
  │ Filter               │ Description                      │
  ├──────────────────────┼──────────────────────────────────┤
  │ default(val)         │ Provide fallback for undefined   │
  │ mandatory            │ Fail if variable is undefined    │
  │ bool                 │ Convert to boolean               │
  │ int / float          │ Type conversion                  │
  │ string               │ Convert to string                │
  │ to_json / to_yaml    │ Serialize                        │
  │ from_json / from_yaml│ Deserialize                      │
  │ join(', ')           │ Join list elements               │
  │ split(',')           │ Split string to list             │
  │ trim                 │ Remove whitespace                │
  │ hash('sha256')       │ Hash a string                    │
  │ b64encode/b64decode  │ Base64 encoding                  │
  │ combine(dict2)       │ Merge dictionaries               │
  │ selectattr('key',    │ Filter list of dicts             │
  │   'eq', 'value')     │                                  │
  │ map(attribute='key') │ Extract attribute from list      │
  │ flatten              │ Flatten nested lists             │
  │ ipaddr               │ Validate/manipulate IP addresses │
  └──────────────────────┴──────────────────────────────────┘
```

---

## Jinja2 Tests

Tests check conditions and return boolean:

```yaml
tasks:
  - name: Check if variable is defined
    debug:
      msg: "Variable exists"
    when: my_var is defined

  - name: Check if variable is undefined
    debug:
      msg: "Variable missing"
    when: my_var is undefined

  - name: Check result status
    debug:
      msg: "Task succeeded"
    when: result is success   # or: succeeded, failed, changed, skipped

  - name: Check if string matches
    debug:
      msg: "It's a web server"
    when: inventory_hostname is match("web.*")

  - name: Check if value is in list
    debug:
      msg: "Port is standard"
    when: http_port in [80, 443, 8080]

  - name: Check file existence
    stat:
      path: /etc/config.yml
    register: config
  - debug:
      msg: "Config exists and is a file"
    when: config.stat.exists and config.stat.isreg
```

---

## Variable Scope

```
  VARIABLE SCOPE
  ══════════════════════════════════════════════════════════

  ┌─────────────────────────────────────────┐
  │ GLOBAL SCOPE                            │
  │  • Extra vars (-e)                      │
  │  • Config settings                      │
  │                                         │
  │  ┌─────────────────────────────────┐    │
  │  │ PLAY SCOPE                      │    │
  │  │  • vars:                        │    │
  │  │  • vars_files:                  │    │
  │  │  • vars_prompt:                 │    │
  │  │  • group_vars / host_vars      │    │
  │  │                                 │    │
  │  │  ┌─────────────────────────┐    │    │
  │  │  │ HOST SCOPE              │    │    │
  │  │  │  • set_fact             │    │    │
  │  │  │  • register             │    │    │
  │  │  │  • host-specific vars   │    │    │
  │  │  │  (Persist across plays  │    │    │
  │  │  │   for the same host)    │    │    │
  │  │  └─────────────────────────┘    │    │
  │  └─────────────────────────────────┘    │
  └─────────────────────────────────────────┘
```

```yaml
---
# Play 1
- name: Gather info
  hosts: webservers
  tasks:
    - set_fact:
        app_state: running    # Host-scoped: survives to Play 2

# Play 2
- name: Use gathered info
  hosts: webservers
  tasks:
    - debug:
        msg: "App is {{ app_state }}"    # ✅ Available
```

---

## Special Variables (Magic Variables)

```yaml
# Variables Ansible sets automatically
- debug:
    msg: |
      Host: {{ inventory_hostname }}
      FQDN: {{ inventory_hostname_short }}
      Groups: {{ group_names }}
      All hosts: {{ groups['all'] }}
      Play hosts: {{ ansible_play_hosts }}
      Hostvars: {{ hostvars['other_host']['some_var'] }}
      Role path: {{ role_path }}
      Playbook dir: {{ playbook_dir }}
      Environment: {{ ansible_env.HOME }}
```

---

## Real-World Scenario: Environment-Specific Configuration

```yaml
# deploy.yml
---
- name: Deploy application
  hosts: appservers
  vars:
    app_name: myapp
    default_port: 8080
  vars_files:
    - "vars/{{ env | default('staging') }}.yml"
  tasks:
    - name: Set deployment timestamp
      set_fact:
        deploy_timestamp: "{{ ansible_date_time.iso8601 }}"

    - name: Create config from template
      template:
        src: app.conf.j2
        dest: "/etc/{{ app_name }}/config.yml"
      vars:
        config_port: "{{ app_port | default(default_port) }}"
        config_debug: "{{ debug | default(false) | bool }}"
        config_workers: "{{ workers | default(ansible_processor_vcpus) }}"

    - name: Show deployment info
      debug:
        msg: >
          Deployed {{ app_name }} to {{ env }}
          at {{ deploy_timestamp }}
          on port {{ app_port | default(default_port) }}
```

```bash
# Usage
ansible-playbook deploy.yml -e "env=production"
ansible-playbook deploy.yml -e "env=staging app_port=9090"
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| `undefined variable` error | Use `default()` filter or check with `is defined` |
| YAML error when variable starts a value | Wrap in quotes: `"{{ var }}"` |
| Wrong variable value (unexpected override) | Check precedence; use `-e` for highest priority |
| Variable not available in other play | Use `set_fact` (host-scoped) or `hostvars` |
| Dictionary key conflicts with Python methods | Use bracket notation: `dict['keys']` instead of `dict.keys` |
| `vars_files` file not found | Use `include_vars` with `ignore_errors` or check path |

---

## Summary Table

| Feature | Description |
|---------|-------------|
| `vars:` | Define variables inline in a play |
| `vars_files:` | Load variables from YAML files |
| `vars_prompt:` | Prompt user for input at runtime |
| `register:` | Capture task output as a variable |
| `set_fact:` | Create/modify variables at runtime |
| `-e` / `--extra-vars` | CLI; highest precedence |
| Jinja2 filters | Transform variables (`default`, `upper`, `int`, etc.) |
| Jinja2 tests | Boolean checks (`is defined`, `is match`, etc.) |
| Bracket notation | Safe dictionary access: `dict['key']` |
| Magic variables | Auto-set by Ansible (`inventory_hostname`, etc.) |

---

## Quick Revision Questions

1. **Which variable source has the highest precedence?**
   > Extra vars (`-e` on the CLI) always win and override all other sources.

2. **Why must you quote `"{{ variable }}"` when it starts a YAML value?**
   > YAML interprets `{` as the start of a dictionary. Quotes tell YAML it's a string to be parsed by Jinja2.

3. **What is the difference between `register` and `set_fact`?**
   > `register` captures the full return value of a task. `set_fact` creates or sets custom variables with arbitrary values.

4. **How do you provide a default value for an undefined variable?**
   > Use the `default()` filter: `{{ my_var | default('fallback_value') }}`.

5. **What is the scope of a `set_fact` variable?**
   > It's host-scoped and persists across plays within the same playbook run for that specific host.

6. **How do you safely access a dictionary key that might conflict with a Python method?**
   > Use bracket notation: `my_dict['keys']` instead of `my_dict.keys`.

---

[← Previous: Handlers and Notify](03-handlers-and-notify.md) | [Next: Conditionals (when) →](05-conditionals-when.md)
