# Chapter 10.4: Testing Strategies

[← Previous: Security Best Practices](03-security-best-practices.md) | [Next: CI/CD Integration →](05-ci-cd-integration.md)

---

## Overview

Testing Ansible code is essential to catch errors before they reach production. This chapter covers the full **testing pyramid** — from fast static analysis through integration testing with Molecule to production validation.

---

## The Ansible Testing Pyramid

```
  ══════════════════════════════════════════════════════════

                    ▲
                   / \
                  / P \        Production Validation
                 / R O \       (smoke tests, monitoring)
                / D     \
               /─────────\
              / INTEGRAT- \    Molecule / Vagrant
             /   ION       \   (full convergence)
            /───────────────\
           /   UNIT TESTS    \  Custom module unit tests
          /───────────────────\
         /   ANSIBLE-LINT      \  Linting & style
        /───────────────────────\
       /    YAML / SYNTAX CHECK  \  Basic validation
      /───────────────────────────\

      ◄── Fast & cheap        Slow & thorough ──▶
```

---

## Level 1: YAML Syntax Check

### ansible-playbook --syntax-check

```bash
# Verify YAML and Ansible syntax
ansible-playbook playbooks/site.yml --syntax-check
```

```
  playbook: playbooks/site.yml    ◄── No errors = valid syntax
```

### yamllint

```bash
pip install yamllint
yamllint .
```

```yaml
# .yamllint (config file)
---
extends: default
rules:
  line-length:
    max: 120
  truthy:
    allowed-values: ['true', 'false']
  comments:
    min-spaces-from-content: 1
  indentation:
    spaces: 2
    indent-sequences: consistent
```

---

## Level 2: ansible-lint

```bash
pip install ansible-lint
ansible-lint playbooks/site.yml
```

### Common Lint Rules

| Rule ID | Description | Fix |
|---------|-------------|-----|
| `yaml` | YAML syntax issues | Fix formatting |
| `name[missing]` | Tasks without names | Add descriptive name |
| `no-changed-when` | command/shell without changed_when | Add `changed_when` |
| `command-instead-of-module` | Using shell when module exists | Replace with module |
| `risky-file-permissions` | File tasks without explicit mode | Add `mode:` |
| `fqcn[action-core]` | Not using FQCN | Use `ansible.builtin.apt` |

### Configuration

```yaml
# .ansible-lint
---
profile: production          # strict rule set

skip_list:
  - yaml[line-length]       # Skip specific rules

warn_list:
  - experimental             # Warn instead of fail

exclude_paths:
  - .cache/
  - .github/
  - molecule/

enable_list:
  - no-log-password
  - no-same-owner
```

```
  LINT OUTPUT EXAMPLE
  ══════════════════════════════════════════════════════════

  WARNING  Listing 3 violation(s):
  playbooks/deploy.yml:15: name[missing] All tasks should be named.
  playbooks/deploy.yml:22: no-changed-when Commands should not
    change things if nothing needs doing.
  roles/web/tasks/main.yml:8: risky-file-permissions File
    permissions unset or incorrect.
```

---

## Level 3: Unit Testing Custom Modules

```python
# tests/unit/test_app_config.py
import pytest
import json
import os
import sys

# Add library path
sys.path.insert(0, os.path.join(os.path.dirname(__file__), '../../library'))
from app_config import run_module

from ansible.module_utils import basic
from ansible.module_utils._text import to_bytes

def set_module_args(args):
    """Prepare module args for testing."""
    args = json.dumps({'ANSIBLE_MODULE_ARGS': args})
    basic._ANSIBLE_ARGS = to_bytes(args)

class TestAppConfig:
    def test_create_new_config(self, tmp_path):
        config_path = str(tmp_path / "config.json")
        set_module_args({
            'path': config_path,
            'settings': {'key': 'value'},
            'backup': False,
        })
        with pytest.raises(SystemExit) as exc:
            run_module()
        assert exc.value.code == 0

        with open(config_path) as f:
            data = json.load(f)
        assert data['key'] == 'value'

    def test_no_change_when_same(self, tmp_path):
        config_path = str(tmp_path / "config.json")
        with open(config_path, 'w') as f:
            json.dump({'key': 'value'}, f)

        set_module_args({
            'path': config_path,
            'settings': {'key': 'value'},
        })
        with pytest.raises(SystemExit):
            run_module()
```

```bash
# Run unit tests
pytest tests/unit/ -v
```

---

## Level 4: Integration Testing with Molecule

### Install Molecule

```bash
pip install molecule molecule-docker ansible-lint
```

### Initialise

```bash
cd roles/webserver
molecule init scenario --driver-name docker
```

```
  roles/webserver/
  └── molecule/
      └── default/
          ├── molecule.yml       ◄── Test configuration
          ├── converge.yml       ◄── Playbook to test role
          ├── verify.yml         ◄── Assertions
          └── prepare.yml        ◄── Pre-test setup (optional)
```

### molecule.yml

```yaml
---
dependency:
  name: galaxy

driver:
  name: docker

platforms:
  - name: ubuntu-test
    image: ubuntu:22.04
    pre_build_image: true
    command: /sbin/init
    privileged: true

  - name: rocky-test
    image: rockylinux:9
    pre_build_image: true
    command: /sbin/init
    privileged: true

provisioner:
  name: ansible
  lint:
    name: ansible-lint

verifier:
  name: ansible
```

### converge.yml

```yaml
---
- name: Converge
  hosts: all
  roles:
    - role: webserver
      vars:
        webserver_port: 8080
```

### verify.yml

```yaml
---
- name: Verify
  hosts: all
  gather_facts: false
  tasks:
    - name: Check nginx is installed
      command: nginx -v
      register: nginx_version
      changed_when: false

    - name: Check nginx is running
      service:
        name: nginx
        state: started
      check_mode: true
      register: nginx_service

    - name: Assert nginx is running
      assert:
        that:
          - nginx_service is not changed

    - name: Check port is listening
      wait_for:
        port: 8080
        timeout: 5
```

### Run Molecule

```bash
# Full test sequence
molecule test

# Interactive development
molecule create      # Create container
molecule converge    # Run playbook
molecule verify      # Run assertions
molecule idempotence # Run again, check no changes
molecule destroy     # Remove container
```

```
  MOLECULE TEST SEQUENCE
  ══════════════════════════════════════════════════════════

  molecule test
  │
  ├── dependency     Install Galaxy dependencies
  ├── lint           Run yamllint + ansible-lint
  ├── cleanup        Remove previous instances
  ├── destroy        Ensure clean state
  ├── syntax         Syntax check playbooks
  ├── create         Spin up test instances
  ├── prepare        Pre-test setup
  ├── converge       Run the role            ◄── First run
  ├── idempotence    Run again (no changes?) ◄── Verify idempotency
  ├── verify         Run assertions
  ├── cleanup        Post-test cleanup
  └── destroy        Remove instances
```

---

## Level 5: Production Validation

```yaml
# playbooks/smoke-test.yml
---
- name: Post-deployment smoke tests
  hosts: webservers
  gather_facts: false
  tasks:
    - name: Check HTTP response
      uri:
        url: "http://{{ inventory_hostname }}:{{ http_port }}/health"
        status_code: 200
        return_content: true
      register: health_check
      retries: 3
      delay: 5

    - name: Verify response content
      assert:
        that:
          - "'healthy' in health_check.content"
        fail_msg: "Health check failed on {{ inventory_hostname }}"

    - name: Check SSL certificate validity
      uri:
        url: "https://{{ inventory_hostname }}"
        validate_certs: true
      when: ssl_enabled | default(false)
```

---

## Testing Tools Summary

```
  TOOL              LEVEL           SPEED     CONFIDENCE
  ══════════════════════════════════════════════════════════
  yamllint          Syntax          ████░░    ██░░░░
  --syntax-check    Syntax          ████░░    ██░░░░
  ansible-lint      Style/Best      ███░░░    ███░░░
  pytest            Unit            ███░░░    ███░░░
  Molecule          Integration     ██░░░░    █████░
  Smoke tests       Production      █░░░░░    ██████
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| yamllint too strict | Customise `.yamllint` config, relax rules |
| ansible-lint false positive | Add to `skip_list` in `.ansible-lint` |
| Molecule docker fails | Ensure Docker is running; use `privileged: true` for systemd |
| Idempotence test fails | Fix tasks that report changed on re-run |
| Molecule verify fails | Check assertions match actual state after converge |
| CI too slow | Parallelise tests; use pre-built Docker images |

---

## Summary Table

| Level | Tool | What It Tests |
|-------|------|---------------|
| 1 | `yamllint` | YAML formatting |
| 1 | `--syntax-check` | Ansible syntax validity |
| 2 | `ansible-lint` | Best practices and style |
| 3 | `pytest` | Custom module logic |
| 4 | Molecule | Full role convergence and idempotency |
| 5 | Smoke tests | Production service health |

---

## Quick Revision Questions

1. **What is the Ansible testing pyramid?**
   > A layered approach from fast, cheap tests (syntax, lint) at the base through integration tests (Molecule) to production validation (smoke tests) at the top.

2. **What does `ansible-lint` check for?**
   > Best practices, naming conventions, deprecated syntax, missing `no_log`, use of shell instead of modules, missing file permissions, and more.

3. **What is the Molecule idempotence test?**
   > Molecule runs the role a second time after converge and verifies that no tasks report `changed`, confirming the role is idempotent.

4. **How do you test a custom Ansible module?**
   > Write Python unit tests using pytest, setting module args with `_ANSIBLE_ARGS` and asserting the module's exit code and output.

5. **What are smoke tests in an Ansible context?**
   > Post-deployment checks (HTTP health endpoints, SSL validation, service status) that verify the infrastructure is working correctly in production.

---

[← Previous: Security Best Practices](03-security-best-practices.md) | [Next: CI/CD Integration →](05-ci-cd-integration.md)
