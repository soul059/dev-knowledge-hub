# Chapter 9.4: Custom Modules

[← Previous: Callback Plugins](03-callback-plugins.md) | [Next: Ansible Tower / AWX →](05-ansible-tower-awx.md)

---

## Overview

When built-in modules don't cover your needs, Ansible lets you write **custom modules** in any language — though **Python** is the most common and best-supported. Custom modules run on managed nodes and return JSON results.

---

## How Modules Work

```
  CONTROL NODE                          MANAGED NODE
  ══════════════════════════════════════════════════════════

  ┌──────────────┐    1. Copy     ┌─────────────────────┐
  │  Playbook    │──────────────▶│  /tmp/.ansible/      │
  │              │               │  ┌─────────────────┐ │
  │  - my_module │  2. Execute   │  │  my_module.py   │ │
  │    param: v  │──────────────▶│  │  (+ args JSON)  │ │
  │              │               │  └────────┬────────┘ │
  │              │  3. JSON      │           │          │
  │              │◀──────────────│  {"changed": true}   │
  └──────────────┘               └─────────────────────┘
                                  4. Cleanup temp files
```

---

## Module Locations

```
  Module Search Order:
  ══════════════════════════════════════════════════════════

  1. ./library/                   ◄── Project-level (recommended)
  2. ~/.ansible/plugins/modules/  ◄── User-level
  3. /usr/share/ansible/plugins/  ◄── System-level
  4. Configured in ansible.cfg:
     [defaults]
     library = /path/to/modules
```

---

## Basic Custom Module (Python)

### Minimal Module Structure

```python
#!/usr/bin/python
# library/my_module.py

from ansible.module_utils.basic import AnsibleModule

def run_module():
    # Define accepted arguments
    module_args = dict(
        name=dict(type='str', required=True),
        state=dict(type='str', default='present',
                   choices=['present', 'absent']),
        force=dict(type='bool', default=False),
    )

    # Initialise the module
    module = AnsibleModule(
        argument_spec=module_args,
        supports_check_mode=True,
    )

    # Collect inputs
    name = module.params['name']
    state = module.params['state']
    force = module.params['force']

    # Result dictionary
    result = dict(
        changed=False,
        original_message='',
        message='',
    )

    # Check mode — report but don't change
    if module.check_mode:
        result['changed'] = True
        module.exit_json(**result)

    # ─── Business Logic ───
    # (Check current state, make changes, set changed=True if modified)

    if state == 'present':
        result['changed'] = True
        result['message'] = f'Resource {name} created'
    else:
        result['changed'] = True
        result['message'] = f'Resource {name} removed'

    # Return success
    module.exit_json(**result)

def main():
    run_module()

if __name__ == '__main__':
    main()
```

---

## Module Argument Types

| Type | Python Type | Example |
|------|-------------|---------|
| `str` | string | `name: "webserver"` |
| `int` | integer | `port: 8080` |
| `float` | float | `ratio: 0.75` |
| `bool` | boolean | `force: true` |
| `list` | list | `tags: [a, b, c]` |
| `dict` | dictionary | `options: {k: v}` |
| `path` | string (expanded) | `dest: ~/file.txt` |
| `raw` | any | No type coercion |

### Advanced Argument Spec

```python
module_args = dict(
    name=dict(
        type='str',
        required=True,
        aliases=['service_name'],
    ),
    state=dict(
        type='str',
        default='present',
        choices=['present', 'absent', 'updated'],
    ),
    port=dict(
        type='int',
        default=80,
    ),
    config=dict(
        type='dict',
        default=None,
    ),
    tags=dict(
        type='list',
        elements='str',
        default=[],
    ),
    password=dict(
        type='str',
        no_log=True,         # Hide from logs
    ),
    dest=dict(
        type='path',         # Expands ~ and env vars
        required=True,
    ),
)

# Mutual exclusivity and dependencies
module = AnsibleModule(
    argument_spec=module_args,
    supports_check_mode=True,
    mutually_exclusive=[
        ['state', 'force'],
    ],
    required_if=[
        ['state', 'present', ['port']],
    ],
    required_together=[
        ['username', 'password'],
    ],
)
```

---

## Return Values

```python
# Success — module.exit_json()
module.exit_json(
    changed=True,
    msg="Operation completed",
    ansible_facts={
        'my_app_version': '2.1.0',  # Registers as a fact
    },
    result_data={
        'id': 12345,
        'status': 'active',
    },
)

# Failure — module.fail_json()
module.fail_json(
    msg="Failed to connect to API",
    rc=1,
    stderr="Connection refused",
)
```

```
  RETURN VALUE FLOW
  ══════════════════════════════════════════════════════════

  module.exit_json()          module.fail_json()
       │                           │
       ▼                           ▼
  {"changed": true,           {"failed": true,
   "msg": "...",               "msg": "...",
   "ansible_facts": {...}}     "rc": 1}
       │                           │
       ▼                           ▼
  Task OK / Changed           Task FAILED
  register: result            (rescue block or stop)
```

---

## Practical Example: Application Config Module

```python
#!/usr/bin/python
# library/app_config.py

from ansible.module_utils.basic import AnsibleModule
import os
import json

DOCUMENTATION = r'''
---
module: app_config
short_description: Manage application configuration files
description:
    - Creates or updates JSON configuration files
    - Supports check mode
options:
    path:
        description: Path to configuration file
        required: true
        type: path
    settings:
        description: Dictionary of settings to apply
        required: true
        type: dict
    backup:
        description: Create backup before modifying
        default: true
        type: bool
'''

EXAMPLES = r'''
- name: Set application config
  app_config:
    path: /etc/myapp/config.json
    settings:
      database_host: db.example.com
      database_port: 5432
      debug: false
'''

RETURN = r'''
path:
    description: Path to the config file
    type: str
    returned: always
backup_file:
    description: Path to backup file if created
    type: str
    returned: when backup is created
'''

def run_module():
    module_args = dict(
        path=dict(type='path', required=True),
        settings=dict(type='dict', required=True),
        backup=dict(type='bool', default=True),
    )

    module = AnsibleModule(
        argument_spec=module_args,
        supports_check_mode=True,
    )

    path = module.params['path']
    settings = module.params['settings']
    backup = module.params['backup']

    result = dict(changed=False, path=path)

    # Read existing config
    current_config = {}
    if os.path.exists(path):
        try:
            with open(path, 'r') as f:
                current_config = json.load(f)
        except (json.JSONDecodeError, IOError) as e:
            module.fail_json(msg=f"Error reading {path}: {e}")

    # Merge settings
    new_config = {**current_config, **settings}

    # Detect changes
    if new_config != current_config:
        result['changed'] = True
        result['diff'] = {
            'before': json.dumps(current_config, indent=2),
            'after': json.dumps(new_config, indent=2),
        }

        if not module.check_mode:
            # Create backup
            if backup and os.path.exists(path):
                backup_file = module.backup_local(path)
                result['backup_file'] = backup_file

            # Write new config
            directory = os.path.dirname(path)
            if not os.path.exists(directory):
                os.makedirs(directory, mode=0o755)

            with open(path, 'w') as f:
                json.dump(new_config, f, indent=2)

    module.exit_json(**result)

def main():
    run_module()

if __name__ == '__main__':
    main()
```

### Using the Custom Module

```yaml
- name: Configure application
  hosts: app_servers
  tasks:
    - name: Set database config
      app_config:
        path: /etc/myapp/config.json
        settings:
          database_host: "{{ db_host }}"
          database_port: 5432
          log_level: info
      register: config_result

    - name: Restart app if config changed
      service:
        name: myapp
        state: restarted
      when: config_result.changed
```

---

## Testing Custom Modules

### Local Testing

```bash
# Test module directly with JSON args
python library/app_config.py /tmp/args.json

# /tmp/args.json contains:
# {"ANSIBLE_MODULE_ARGS": {
#     "path": "/tmp/test.json",
#     "settings": {"key": "value"}
# }}
```

### Playbook Test

```yaml
- name: Test custom module
  hosts: localhost
  connection: local
  tasks:
    - name: Test with check mode
      app_config:
        path: /tmp/test_config.json
        settings:
          test_key: test_value
      check_mode: true
      register: check_result

    - name: Verify check mode didn't change
      assert:
        that:
          - check_result.changed == true
          - not (lookup('file', '/tmp/test_config.json', errors='ignore'))
```

---

## Module Utils (Shared Code)

```
  project/
  ├── library/
  │   ├── my_module_a.py
  │   └── my_module_b.py
  └── module_utils/
      └── my_common.py       ◄── Shared utilities
```

```python
# module_utils/my_common.py
def validate_name(name):
    if not name or len(name) > 64:
        return False, "Name must be 1-64 characters"
    return True, ""

# library/my_module_a.py
from ansible.module_utils.my_common import validate_name

def run_module():
    module = AnsibleModule(...)
    valid, error = validate_name(module.params['name'])
    if not valid:
        module.fail_json(msg=error)
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Module not found | Place in `library/` dir or configure `library` path in ansible.cfg |
| JSON decode error on return | Ensure module only prints valid JSON to stdout |
| Module hangs | Check for blocking I/O; add timeout handling |
| `check_mode` makes changes | Guard all write operations with `if not module.check_mode` |
| Import errors on managed node | Module utils must be shipped; use `module_utils/` for shared code |
| Debugging module logic | Use `module.debug()` or test locally with JSON args |

---

## Summary Table

| Concept | Details |
|---------|---------|
| Module language | Any, but Python preferred |
| Primary import | `from ansible.module_utils.basic import AnsibleModule` |
| Project location | `./library/` directory |
| Argument definition | `argument_spec` dict with types, defaults, choices |
| Success return | `module.exit_json(changed=True, ...)` |
| Failure return | `module.fail_json(msg="error")` |
| Check mode | `supports_check_mode=True` + guard writes |
| Documentation | `DOCUMENTATION`, `EXAMPLES`, `RETURN` docstrings |
| Shared code | `module_utils/` directory |
| Testing | JSON args file or playbook with `assert` |

---

## Quick Revision Questions

1. **Where should you place custom modules for a project?**
   > In a `library/` directory at the project root (next to your playbooks).

2. **How do you define accepted parameters in a custom module?**
   > Using the `argument_spec` dictionary passed to `AnsibleModule()`, specifying type, required, default, choices, etc.

3. **What is the difference between `exit_json()` and `fail_json()`?**
   > `exit_json()` indicates success and optionally reports `changed=True`. `fail_json()` indicates failure and stops the play (unless `ignore_errors` is set).

4. **How do you support check mode in a custom module?**
   > Set `supports_check_mode=True` in `AnsibleModule()` and wrap all state-changing code in `if not module.check_mode` blocks.

5. **How can you share common code between multiple custom modules?**
   > Place shared functions in the `module_utils/` directory and import them in your modules.

6. **How do you test a custom module locally?**
   > Run `python library/my_module.py /tmp/args.json` where the JSON file contains `{"ANSIBLE_MODULE_ARGS": {...}}`.

---

[← Previous: Callback Plugins](03-callback-plugins.md) | [Next: Ansible Tower / AWX →](05-ansible-tower-awx.md)
