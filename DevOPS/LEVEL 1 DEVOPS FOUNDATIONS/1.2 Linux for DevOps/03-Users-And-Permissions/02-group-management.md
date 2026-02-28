# Chapter 2: Group Management

[← Previous: User Management](01-user-management.md) | [Back to README](../README.md) | [Next: File Permissions →](03-file-permissions.md)

---

## Overview

Groups are the primary mechanism for organizing users and controlling shared access to resources. Properly using groups reduces the need for complex per-user permissions and enables manageable access control for teams, applications, and services.

---

## 2.1 How Groups Work

```
┌──────────────────────────────────────────────────────────────┐
│                    GROUP ARCHITECTURE                         │
│                                                               │
│  User: devops                                                │
│  ├── Primary Group: devops (GID 1001)                       │
│  │   └── New files created with this group                   │
│  │                                                           │
│  └── Supplementary Groups:                                   │
│      ├── docker (GID 998)   → Access Docker daemon          │
│      ├── sudo   (GID 27)   → Admin privileges               │
│      ├── www-data (GID 33) → Web server files               │
│      └── developers (GID 1010) → Shared project files       │
│                                                               │
│  Files:                                                       │
│  /etc/group    ──→ Group definitions                         │
│  /etc/gshadow  ──→ Group passwords (rarely used)             │
└──────────────────────────────────────────────────────────────┘
```

### Understanding `/etc/group`

```
docker:x:998:devops,jenkins
│      │ │   │
│      │ │   └── Member list (supplementary members)
│      │ └── GID (Group ID)
│      └── Password placeholder
└── Group name
```

---

## 2.2 Group Management Commands

```bash
# Create a group
sudo groupadd developers
sudo groupadd -g 2000 projectx       # Specific GID

# Delete a group
sudo groupdel developers

# Modify a group
sudo groupmod -n dev-team developers  # Rename
sudo groupmod -g 2001 developers      # Change GID

# List all groups
cat /etc/group
getent group

# View a specific group
getent group docker

# List members of a group
getent group docker | cut -d: -f4
# Or
grep "^docker:" /etc/group

# View current user's groups
groups
groups devops              # Groups for specific user
id -Gn devops             # Same info

# Add user to group
sudo usermod -aG docker devops       # Supplementary (KEEP -a!)
sudo gpasswd -a devops docker        # Alternative method

# Remove user from group
sudo gpasswd -d devops docker

# Change user's primary group
sudo usermod -g developers devops

# Temporarily switch primary group (current session)
newgrp docker
```

---

## 2.3 Common DevOps Groups

```
┌──────────────────────────────────────────────────────────┐
│            COMMON GROUPS IN DEVOPS                        │
├──────────────┬───────────────────────────────────────────┤
│ Group        │ Purpose                                   │
├──────────────┼───────────────────────────────────────────┤
│ sudo/wheel   │ Administrative privileges                 │
│ docker       │ Run Docker commands without sudo          │
│ www-data     │ Web server (Nginx/Apache) files          │
│ adm          │ Read system logs                          │
│ ssh/sshd     │ SSH access control                        │
│ developers   │ Shared project files (custom)             │
│ dba          │ Database administration (custom)          │
│ monitoring   │ Monitoring tool access (custom)           │
└──────────────┴───────────────────────────────────────────┘
```

---

## 2.4 Real-World Scenario: Team Access Management

```bash
#!/bin/bash
# setup-team-access.sh

# Create project groups
sudo groupadd -f frontend
sudo groupadd -f backend
sudo groupadd -f devops
sudo groupadd -f qa

# Create shared project directory
sudo mkdir -p /opt/project/{frontend,backend,shared,deployments}

# Set group ownership
sudo chgrp frontend /opt/project/frontend
sudo chgrp backend /opt/project/backend
sudo chgrp devops /opt/project/deployments

# Set permissions with SGID (new files inherit group)
sudo chmod 2775 /opt/project/{frontend,backend,shared,deployments}

# Add users to their teams
sudo usermod -aG frontend alice
sudo usermod -aG frontend,backend bob    # Full-stack
sudo usermod -aG devops charlie
sudo usermod -aG qa,frontend,backend dave  # QA all access

echo "Team access configured. Users must log out/in for group changes."
```

---

## Summary Table

| Command | Purpose | Key Flags |
|---------|---------|-----------|
| `groupadd` | Create group | `-g` (GID) |
| `groupdel` | Delete group | — |
| `groupmod` | Modify group | `-n` (rename), `-g` (GID) |
| `usermod -aG` | Add user to group | `-a` (append!) |
| `gpasswd -a` | Add user to group | Alternative method |
| `gpasswd -d` | Remove user from group | — |
| `groups` | List user's groups | — |
| `newgrp` | Switch active group | — |
| `getent group` | Query group database | — |

---

## Quick Revision Questions

1. **What is the difference between a primary group and supplementary groups?**
2. **What file stores group information?** Explain its format.
3. **How do you add a user to the `docker` group without removing them from other groups?**
4. **Why must a user log out and back in after being added to a group?**
5. **What is the `newgrp` command used for?**
6. **How would you set up shared directories so that new files automatically inherit the group?** (Hint: SGID)

---

[← Previous: User Management](01-user-management.md) | [Back to README](../README.md) | [Next: File Permissions →](03-file-permissions.md)
