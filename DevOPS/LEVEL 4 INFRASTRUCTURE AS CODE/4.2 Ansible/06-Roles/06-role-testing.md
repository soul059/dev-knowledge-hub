# Chapter 6.6: Role Testing

[← Previous: Collections](05-collections.md) | [Next: Jinja2 Templates →](../07-Templates-and-Files/01-jinja2-templates.md)

---

## Overview

Testing roles ensures they work correctly across target platforms before deploying to production. This chapter covers testing strategies from simple syntax checks to full integration tests with **Molecule**, the standard Ansible role testing framework.

---

## Testing Pyramid for Ansible

```
            ┌─────────┐
            │ Integr- │    Full infrastructure test
            │  ation  │    (Molecule + cloud/containers)
           ┌┴─────────┴┐
           │  Functional │   Run against real/virtual hosts
          ┌┴─────────────┴┐
          │    Linting     │  ansible-lint, yamllint
         ┌┴───────────────┴┐
         │   Syntax Check   │  ansible-playbook --syntax-check
        ┌┴─────────────────┴┐
        │     Unit Tests     │  Check mode, assert module
        └───────────────────┘
```

---

## Level 1: Syntax Checking

```bash
# Check playbook syntax
ansible-playbook site.yml --syntax-check

# Check a role via a test playbook
cat > test.yml << 'EOF'
- hosts: localhost
  roles:
    - my_role
EOF

ansible-playbook test.yml --syntax-check
```

---

## Level 2: Linting

### ansible-lint

```bash
# Install
pip install ansible-lint

# Run
ansible-lint roles/webserver/

# With specific rules
ansible-lint -R -r ~/.ansible-lint-rules roles/webserver/
```

```yaml
# .ansible-lint (project config)
---
skip_list:
  - yaml[line-length]
  - name[missing]
exclude_paths:
  - .cache/
  - .github/
warn_list:
  - experimental
```

### yamllint

```bash
pip install yamllint
yamllint roles/webserver/
```

```yaml
# .yamllint
---
extends: default
rules:
  line-length:
    max: 120
  truthy:
    allowed-values: ['true', 'false', 'yes', 'no']
```

---

## Level 3: Dry Run (Check Mode)

```bash
# Run in check mode — no changes made
ansible-playbook site.yml --check --diff

# Against specific hosts
ansible-playbook site.yml --check --diff --limit webserver01
```

```yaml
# Tasks that support check mode
- name: Install package
  apt:
    name: nginx
    state: present
  # Automatically works in check mode

# Force a task to always run (even in check mode)
- name: Gather info
  command: uptime
  check_mode: no        # or: always runs
```

---

## Level 4: Molecule (Integration Testing)

### What is Molecule?

```
  MOLECULE WORKFLOW
  ══════════════════════════════════════════════════════════

  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌────────┐
  │ Create  │───▶│Converge │───▶│ Verify  │───▶│Destroy │
  │ (spin   │    │ (run    │    │ (test   │    │ (clean │
  │  up VM) │    │  role)  │    │  result)│    │  up)   │
  └─────────┘    └─────────┘    └─────────┘    └────────┘

  Drivers: Docker, Podman, Vagrant, EC2, Azure, GCP, etc.
```

### Installing Molecule

```bash
# Core + Docker driver
pip install molecule molecule-plugins[docker]

# Or with Podman
pip install molecule molecule-plugins[podman]

# Or with Vagrant
pip install molecule molecule-plugins[vagrant]
```

### Initialising Molecule in a Role

```bash
cd roles/webserver
molecule init scenario --driver-name docker
```

```
  roles/webserver/
  └── molecule/
      └── default/               ◄── Scenario name
          ├── converge.yml       ◄── Playbook to test the role
          ├── molecule.yml       ◄── Molecule configuration
          ├── verify.yml         ◄── Verification tests
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
    image: geerlingguy/docker-ubuntu2204-ansible
    pre_build_image: true
    privileged: true
    command: /lib/systemd/systemd
    volumes:
      - /sys/fs/cgroup:/sys/fs/cgroup:rw

  - name: rocky-test
    image: geerlingguy/docker-rockylinux9-ansible
    pre_build_image: true
    privileged: true
    command: /lib/systemd/systemd

provisioner:
  name: ansible
  playbooks:
    converge: converge.yml
    verify: verify.yml
  inventory:
    group_vars:
      all:
        http_port: 8080
        server_name: test.local

verifier:
  name: ansible
```

### converge.yml

```yaml
---
- name: Converge
  hosts: all
  become: yes
  roles:
    - role: webserver
      vars:
        http_port: 8080
        server_name: test.local
```

### verify.yml

```yaml
---
- name: Verify
  hosts: all
  become: yes
  tasks:
    - name: Check nginx is installed
      package:
        name: nginx
        state: present
      check_mode: yes
      register: pkg_check
      failed_when: pkg_check.changed

    - name: Check nginx is running
      service:
        name: nginx
        state: started
      check_mode: yes
      register: svc_check
      failed_when: svc_check.changed

    - name: Check nginx is listening
      wait_for:
        port: 8080
        timeout: 5

    - name: Verify config file exists
      stat:
        path: /etc/nginx/nginx.conf
      register: config_stat
      failed_when: not config_stat.stat.exists

    - name: Check HTTP response
      uri:
        url: "http://localhost:8080"
        status_code: 200
```

### Running Molecule

```bash
# Full test sequence
molecule test

# Individual steps
molecule create       # Create instances
molecule converge     # Run the role
molecule verify       # Run verification
molecule destroy      # Tear down instances
molecule login        # SSH into test instance

# List instances
molecule list

# Run with debug
molecule --debug test
```

---

## Molecule Test Sequence

```
  DEFAULT TEST SEQUENCE
  ══════════════════════════════════════════════════════════

  1. dependency         Install Galaxy dependencies
  2. lint               Run linters (if configured)
  3. cleanup            Pre-test cleanup
  4. destroy            Remove any existing instances
  5. syntax             Syntax check
  6. create             Create test instances
  7. prepare            Run preparation playbook
  8. converge           Run the role (first time)
  9. idempotence        Run the role again (should show 0 changes)
  10. side_effect       Run side-effect playbook (optional)
  11. verify            Run verification tests
  12. cleanup           Post-test cleanup
  13. destroy           Remove instances
```

---

## Idempotence Testing

Molecule's idempotence step runs the role twice. The second run should report **zero changes**:

```
  IDEMPOTENCE CHECK
  ══════════════════════════════════════════════════════════

  Run 1 (converge):  changed=5   ok=8    ◄── Expected
  Run 2 (idempotence): changed=0  ok=13   ◄── PASS ✓

  If Run 2 shows changed>0:                ◄── FAIL ✗
    → Tasks are not idempotent
    → Fix: use creates/removes, changed_when, state checks
```

---

## Multiple Test Scenarios

```
  molecule/
  ├── default/          ◄── Default scenario (Docker)
  │   ├── molecule.yml
  │   ├── converge.yml
  │   └── verify.yml
  ├── ec2/              ◄── AWS EC2 scenario
  │   ├── molecule.yml
  │   ├── converge.yml
  │   └── verify.yml
  └── vagrant/          ◄── Vagrant scenario
      ├── molecule.yml
      ├── converge.yml
      └── verify.yml
```

```bash
# Run specific scenario
molecule test -s ec2
molecule converge -s vagrant
```

---

## Real-World Scenario: CI/CD Pipeline Testing

```yaml
# .github/workflows/test-role.yml
---
name: Role Tests
on: [push, pull_request]

jobs:
  molecule:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        distro:
          - ubuntu2204
          - rockylinux9
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          pip install ansible molecule molecule-plugins[docker]

      - name: Run Molecule
        run: molecule test
        env:
          MOLECULE_DISTRO: ${{ matrix.distro }}
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Docker permission denied | Add user to docker group or use `sudo` |
| Systemd not available in container | Use `geerlingguy/docker-*-ansible` images with `privileged: true` |
| Idempotence failure | Check for tasks using `command`/`shell` without `changed_when` |
| Molecule can't find role | Ensure you `cd` into the role directory before running |
| Slow tests | Use `pre_build_image: true` to skip image building |

---

## Summary Table

| Tool | Purpose |
|------|---------|
| `--syntax-check` | Parse YAML, find basic errors |
| `ansible-lint` | Style and best-practice checks |
| `--check --diff` | Dry run, simulate changes |
| Molecule | Full lifecycle integration testing |
| `molecule test` | Run complete test sequence |
| `molecule converge` | Apply role to test instances |
| `molecule verify` | Run verification playbook |
| Idempotence check | Ensure second run has zero changes |

---

## Quick Revision Questions

1. **What does `ansible-playbook --syntax-check` do?**
   > It parses the playbook and role YAML for syntax errors without connecting to any hosts.

2. **What is the purpose of the Molecule idempotence step?**
   > It runs the role a second time and verifies zero changes, confirming the role is idempotent.

3. **What is the default Molecule test sequence?**
   > dependency → lint → cleanup → destroy → syntax → create → prepare → converge → idempotence → verify → cleanup → destroy.

4. **How do you test a role against multiple OS distributions?**
   > Define multiple platforms in `molecule.yml` or use CI matrix strategies to test across distributions.

5. **What command lets you SSH into a Molecule test instance?**
   > `molecule login` — connects to the running test container/VM for debugging.

---

[← Previous: Collections](05-collections.md) | [Next: Jinja2 Templates →](../07-Templates-and-Files/01-jinja2-templates.md)
