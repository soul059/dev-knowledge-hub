# Chapter 1.2: Push vs Pull Model

[← Previous: What is Configuration Management?](01-what-is-configuration-management.md) | [Next: Idempotency →](03-idempotency.md)

---

## Overview

Configuration management tools use one of two fundamental models to apply configurations to target machines: **Push** and **Pull**. Understanding these models is critical to choosing the right tool and architecture for your infrastructure.

---

## Push Model

In the **push model**, a central control node **pushes** configurations to managed nodes on demand.

```
  PUSH MODEL
  ══════════════════════════════════════════════════════════

  ┌────────────────────┐
  │   Control Node     │
  │   (Admin Machine)  │
  │                    │
  │  ┌──────────────┐  │
  │  │  Playbooks   │  │
  │  │  Inventory   │  │─────────── Admin triggers push
  │  │  Configs     │  │           (ansible-playbook site.yml)
  │  └──────────────┘  │
  └────────┬───────────┘
           │
           │  SSH / WinRM (No agent needed!)
           │
     ┌─────┼─────────────────┐
     │     │                 │
     ▼     ▼                 ▼
  ┌──────┐ ┌──────┐    ┌──────┐
  │Node 1│ │Node 2│    │Node N│
  │      │ │      │    │      │
  │ No   │ │ No   │    │ No   │
  │Agent │ │Agent │    │Agent │
  └──────┘ └──────┘    └──────┘

  ✅ Simple architecture
  ✅ No agent to install/maintain
  ✅ Immediate execution
  ✅ Full control over timing

  Tools: Ansible, SaltStack (can do both)
```

### How Push Works (Step by Step)

```
  Step 1: Admin writes playbook
  ┌──────────────────────────┐
  │ site.yml                 │
  │ - hosts: webservers      │
  │   tasks:                 │
  │     - apt: name=nginx    │
  └──────────────────────────┘

  Step 2: Admin runs command
  $ ansible-playbook site.yml
            │
  Step 3: Control node connects via SSH
            │
            ├──► SSH to 192.168.1.10  ──► Execute tasks
            ├──► SSH to 192.168.1.11  ──► Execute tasks
            └──► SSH to 192.168.1.12  ──► Execute tasks
                                              │
  Step 4: Results returned                    │
            ◄─────────────────────────────────┘
  
  Step 5: Summary displayed
  PLAY RECAP ═══════════════════════════════
  192.168.1.10 : ok=3  changed=1  failed=0
  192.168.1.11 : ok=3  changed=1  failed=0
  192.168.1.12 : ok=3  changed=0  failed=0
```

---

## Pull Model

In the **pull model**, each managed node has an **agent** that periodically **pulls** its configuration from a central server.

```
  PULL MODEL
  ══════════════════════════════════════════════════════════

  ┌────────────────────┐
  │   Central Server   │
  │   (Config Master)  │
  │                    │
  │  ┌──────────────┐  │
  │  │  Configs     │  │
  │  │  Catalogs    │  │◄──── Stores desired state
  │  │  Manifests   │  │
  │  └──────────────┘  │
  └────────────────────┘
           ▲     ▲     ▲
           │     │     │    Agents pull every 30 min
           │     │     │    (configurable interval)
     ┌─────┘     │     └──────┐
     │           │            │
  ┌──────┐  ┌──────┐    ┌──────┐
  │Node 1│  │Node 2│    │Node N│
  │      │  │      │    │      │
  │Agent │  │Agent │    │Agent │
  │ ┌──┐ │  │ ┌──┐ │    │ ┌──┐ │
  │ │🔄│ │  │ │🔄│ │    │ │🔄│ │
  │ └──┘ │  │ └──┘ │    │ └──┘ │
  └──────┘  └──────┘    └──────┘

  ✅ Continuous enforcement
  ✅ Self-healing (auto-corrects drift)
  ✅ Scales to thousands of nodes
  
  ❌ Agent required on every node
  ❌ More complex setup
  ❌ Certificate management needed

  Tools: Puppet, Chef, SaltStack (minion mode)
```

### How Pull Works (Step by Step)

```
  Step 1: Admin defines configs on central server
  ┌──────────────────────────┐
  │ manifest.pp (Puppet)     │
  │ package { 'nginx':       │
  │   ensure => present,     │
  │ }                        │
  └──────────────────────────┘

  Step 2: Agent on node checks in (every 30 min)
  ┌──────┐                    ┌────────────┐
  │Agent │────── "Any new ───►│  Central   │
  │      │      config for    │  Server    │
  │      │◄──── me?" ────────│            │
  │      │   "Yes, install    │            │
  │      │    nginx"          │            │
  └──────┘                    └────────────┘

  Step 3: Agent applies configuration locally
  Step 4: Agent reports back to central server
```

---

## Push vs Pull: Side-by-Side Comparison

```
  ┌─────────────────┬──────────────────┬──────────────────┐
  │    Aspect        │   PUSH Model     │   PULL Model     │
  ├─────────────────┼──────────────────┼──────────────────┤
  │                 │                  │                  │
  │ Agent Required  │   ❌ No          │   ✅ Yes         │
  │                 │                  │                  │
  │ Trigger         │   Admin-initiated│   Agent-initiated│
  │                 │   (on-demand)    │   (scheduled)    │
  │                 │                  │                  │
  │ Timing          │   Immediate      │   Periodic       │
  │                 │   when run       │   (30 min default)│
  │                 │                  │                  │
  │ Drift Handling  │   Only when run  │   Continuous     │
  │                 │                  │   self-healing   │
  │                 │                  │                  │
  │ Complexity      │   Simple         │   Complex        │
  │                 │                  │   (PKI, agents)  │
  │                 │                  │                  │
  │ Scalability     │   Good           │   Excellent      │
  │                 │   (with tuning)  │   (distributed)  │
  │                 │                  │                  │
  │ Network         │   SSH outbound   │   HTTPS inbound  │
  │ Requirements    │   from control   │   from agents    │
  │                 │                  │                  │
  │ Setup Time      │   Minutes        │   Hours/Days     │
  │                 │                  │                  │
  │ Example Tools   │   Ansible        │   Puppet, Chef   │
  └─────────────────┴──────────────────┴──────────────────┘
```

---

## Ansible's Push Model in Detail

Ansible uses the **push model** exclusively (though `ansible-pull` exists for special cases):

```
  ANSIBLE PUSH ARCHITECTURE
  ══════════════════════════════════════════════════════════

  ┌─────────────────────────────────────────────┐
  │            CONTROL NODE                      │
  │                                              │
  │  ansible-playbook  ──► Parses playbook       │
  │         │               │                    │
  │         ▼               ▼                    │
  │  ┌───────────┐   ┌─────────────┐            │
  │  │ Inventory │   │   Module    │            │
  │  │ Parser    │   │   Loader    │            │
  │  └─────┬─────┘   └──────┬──────┘            │
  │        │                │                    │
  │        ▼                ▼                    │
  │  ┌───────────────────────────┐              │
  │  │    Connection Plugin      │              │
  │  │    (SSH / WinRM / Local)  │              │
  │  └─────────────┬─────────────┘              │
  └────────────────│─────────────────────────────┘
                   │
        ┌──────────┼──────────┐
        │          │          │
        ▼          ▼          ▼
   ┌────────┐ ┌────────┐ ┌────────┐
   │ Node A │ │ Node B │ │ Node C │
   │        │ │        │ │        │
   │ 1.Copy │ │ 1.Copy │ │ 1.Copy │
   │  module│ │  module│ │  module│
   │ 2.Run  │ │ 2.Run  │ │ 2.Run  │
   │ 3.Clean│ │ 3.Clean│ │ 3.Clean│
   │  up    │ │  up    │ │  up    │
   └────────┘ └────────┘ └────────┘

  Detailed execution per node:
  ┌──────────────────────────────────────┐
  │ 1. SSH connection established        │
  │ 2. Temporary directory created       │
  │ 3. Module code copied to node        │
  │ 4. Module executed with arguments    │
  │ 5. Results returned as JSON          │
  │ 6. Temporary files cleaned up        │
  │ 7. SSH connection closed             │
  └──────────────────────────────────────┘
```

---

## `ansible-pull` (Pull Mode Exception)

Ansible also supports a pull mode via `ansible-pull` for special use cases:

```bash
# On the managed node itself
ansible-pull -U https://github.com/myorg/ansible-configs.git site.yml

# Typically run via cron job
# */15 * * * * ansible-pull -U https://github.com/myorg/configs.git site.yml
```

```
  ANSIBLE-PULL WORKFLOW
  ══════════════════════════════════════════════

  ┌────────────┐         ┌──────────────────┐
  │  Git Repo  │◄────────│  Managed Node    │
  │  (GitHub,  │  Clone  │                  │
  │  GitLab)   │  or     │  1. cron runs    │
  │            │  Pull   │     ansible-pull  │
  │  site.yml  │────────►│  2. pulls repo   │
  │  roles/    │         │  3. runs locally  │
  │  inventory │         │  4. applies config│
  └────────────┘         └──────────────────┘

  Use cases:
  • Auto-scaling groups (no persistent control node)
  • Bootstrapping new servers
  • Edge/remote locations with intermittent connectivity
```

---

## When to Use Each Model

```
  Decision Tree:
  ══════════════════════════════════════════════

  Do you need continuous drift correction?
  │
  ├── YES ──► Do you mind managing agents?
  │           ├── NO  ──► PULL MODEL (Puppet/Chef)
  │           └── YES ──► Push + Cron/Scheduler
  │
  └── NO  ──► Do you want simplicity?
              ├── YES ──► PUSH MODEL (Ansible) ✅
              └── NO  ──► Either model works
```

---

## Real-World Scenarios

### Scenario 1: Small Team, 20 Servers
**Best choice**: Push (Ansible)
- Simple setup, no agents to maintain
- Team manually triggers deployments
- Low overhead

### Scenario 2: Large Enterprise, 5000 Servers
**Best choice**: Pull (Puppet) or Push (Ansible + AWX/Tower)
- Continuous compliance enforcement needed
- Or use Ansible Tower for scheduled push runs

### Scenario 3: Auto-Scaling Cloud
**Best choice**: `ansible-pull` or cloud-init + push
- New instances configure themselves
- No need to update inventory for ephemeral instances

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Push fails to connect | Check SSH keys, firewall rules, and network connectivity |
| Push is slow for many hosts | Increase `forks` in ansible.cfg (default: 5) |
| Pull mode not picking up changes | Check cron job and Git repository accessibility |
| Agent not checking in (pull) | Verify agent service status and certificate validity |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| Push Model | Control node pushes configs to targets on demand |
| Pull Model | Agents on targets pull configs from central server |
| Agentless | No software to install on managed nodes (push advantage) |
| SSH | Transport protocol used by Ansible for push |
| ansible-pull | Ansible's pull mode for special use cases |
| Forks | Number of parallel connections Ansible makes |
| On-demand | Push runs when admin triggers it |
| Periodic | Pull agents check in at configurable intervals |

---

## Quick Revision Questions

1. **What is the fundamental difference between push and pull models?**
   > Push: Control node sends configs to targets. Pull: Targets fetch configs from a central server.

2. **Why is Ansible considered agentless?**
   > Ansible uses SSH to connect to managed nodes and doesn't require any agent software to be installed on them.

3. **What is `ansible-pull` and when would you use it?**
   > `ansible-pull` inverts Ansible's push model by having nodes pull and apply configs from a Git repo. Useful for auto-scaling groups and bootstrapping.

4. **What are two advantages of the push model over pull?**
   > Simpler setup (no agents) and immediate execution (no waiting for agent check-in interval).

5. **What are two advantages of the pull model over push?**
   > Continuous drift correction (self-healing) and better scalability for very large infrastructures.

6. **How does Ansible execute modules on remote nodes?**
   > It establishes an SSH connection, copies the module code to a temp directory on the remote node, executes it, collects the JSON output, and cleans up.

---

[← Previous: What is Configuration Management?](01-what-is-configuration-management.md) | [Next: Idempotency →](03-idempotency.md)
