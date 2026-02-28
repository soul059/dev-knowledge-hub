# Chapter 3: File Permissions

[← Previous: Group Management](02-group-management.md) | [Back to README](../README.md) | [Next: Special Permissions →](04-special-permissions.md)

---

## Overview

Linux file permissions are the first line of defense in system security. Every file and directory has three permission sets (owner, group, others), each controlling read, write, and execute access. Misconfigured permissions are one of the most common causes of security vulnerabilities and application failures.

---

## 3.1 Permission Model

```
┌─────────────────────────────────────────────────────────────────┐
│                   LINUX PERMISSION MODEL                        │
│                                                                  │
│  -rwxr-xr-- 1 devops developers 4096 Feb 20 10:30 deploy.sh   │
│   │││ │││ │││                                                    │
│   │││ │││ ││└── Others: Read only                               │
│   │││ │││ │└─── Others: No write                                │
│   │││ │││ └──── Others: No execute                              │
│   │││ ││└────── Group: Read                                     │
│   │││ │└─────── Group: No write                                 │
│   │││ └──────── Group: Execute                                  │
│   ││└────────── Owner: Read                                     │
│   │└─────────── Owner: Write                                    │
│   └──────────── Owner: Execute                                  │
│                                                                  │
│   Owner (u)    Group (g)    Others (o)                          │
│   rwx          r-x          r--                                  │
│   7            5            4        = 754                       │
└─────────────────────────────────────────────────────────────────┘
```

### Permission Values

```
┌──────────────────────────────────────────────┐
│  PERMISSION BITS                              │
├────────┬────────┬────────┬───────────────────┤
│ Symbol │ Octal  │ Binary │ Meaning           │
├────────┼────────┼────────┼───────────────────┤
│  r     │   4    │  100   │ Read              │
│  w     │   2    │  010   │ Write             │
│  x     │   1    │  001   │ Execute           │
│  -     │   0    │  000   │ No permission     │
├────────┼────────┼────────┼───────────────────┤
│  rwx   │   7    │  111   │ Full access       │
│  rw-   │   6    │  110   │ Read + Write      │
│  r-x   │   5    │  101   │ Read + Execute    │
│  r--   │   4    │  100   │ Read only         │
│  -wx   │   3    │  011   │ Write + Execute   │
│  -w-   │   2    │  010   │ Write only        │
│  --x   │   1    │  001   │ Execute only      │
│  ---   │   0    │  000   │ No access         │
└────────┴────────┴────────┴───────────────────┘
```

### Permissions on Files vs Directories

```
┌──────────────────────────────────────────────────────────┐
│  Permission   │  On Files           │  On Directories    │
├───────────────┼─────────────────────┼────────────────────┤
│  r (read)     │  Read file content  │  List directory    │
│               │                     │  contents (ls)     │
├───────────────┼─────────────────────┼────────────────────┤
│  w (write)    │  Modify content     │  Create/delete     │
│               │                     │  files inside      │
├───────────────┼─────────────────────┼────────────────────┤
│  x (execute)  │  Run as program     │  Enter directory   │
│               │                     │  (cd into it)      │
└───────────────┴─────────────────────┴────────────────────┘
```

---

## 3.2 `chmod` — Change Mode (Permissions)

### Symbolic Mode

```bash
# Syntax: chmod [who][operator][permission] file
# who:      u (user/owner), g (group), o (others), a (all)
# operator: + (add), - (remove), = (set exactly)

# Add execute for owner
chmod u+x script.sh

# Remove write for others
chmod o-w config.yaml

# Set exact permissions for group
chmod g=rx deploy.sh

# Add read for all
chmod a+r README.md

# Multiple changes at once
chmod u+x,g+r,o-w file.txt

# Make file executable by owner only
chmod u+x,go-x script.sh
```

### Octal (Numeric) Mode

```bash
# Common permission sets:
chmod 755 script.sh          # rwxr-xr-x (executable scripts)
chmod 644 config.yaml        # rw-r--r-- (config files)
chmod 600 id_rsa             # rw------- (SSH private key)
chmod 700 .ssh/              # rwx------ (SSH directory)
chmod 666 /dev/null          # rw-rw-rw- (device file)
chmod 777 /tmp/shared        # rwxrwxrwx (shared temp - avoid!)
chmod 640 /etc/shadow        # rw-r----- (shadow file)
chmod 444 important.txt      # r--r--r-- (read-only for all)
```

### Recursive Permissions

```bash
# Change permissions recursively
chmod -R 755 /var/www/html/

# Better: Separate files and directories
find /var/www -type d -exec chmod 755 {} +   # Dirs: 755
find /var/www -type f -exec chmod 644 {} +   # Files: 644
```

---

## 3.3 `chown` — Change Ownership

```bash
# Change owner
sudo chown devops file.txt

# Change owner and group
sudo chown devops:developers file.txt

# Change group only (with colon)
sudo chown :developers file.txt

# Recursive ownership change
sudo chown -R devops:www-data /var/www/html/

# Change ownership to match another file
sudo chown --reference=reference.txt target.txt
```

---

## 3.4 `chgrp` — Change Group

```bash
# Change group ownership
sudo chgrp developers project/
sudo chgrp -R www-data /var/www/

# Reference-based
sudo chgrp --reference=ref.txt target.txt
```

---

## 3.5 `umask` — Default Permission Mask

```
┌───────────────────────────────────────────────────────────┐
│                    umask CALCULATION                       │
│                                                            │
│  Default permissions:                                      │
│    Files:       666 (rw-rw-rw-)                           │
│    Directories: 777 (rwxrwxrwx)                           │
│                                                            │
│  umask 022:                                                │
│    Files:       666 - 022 = 644 (rw-r--r--)               │
│    Directories: 777 - 022 = 755 (rwxr-xr-x)              │
│                                                            │
│  umask 077:                                                │
│    Files:       666 - 077 = 600 (rw-------)               │
│    Directories: 777 - 077 = 700 (rwx------)              │
└───────────────────────────────────────────────────────────┘
```

```bash
# View current umask
umask            # Octal: 0022
umask -S         # Symbolic: u=rwx,g=rx,o=rx

# Set umask for current session
umask 027        # Files: 640, Dirs: 750

# Set umask permanently
echo "umask 027" >> ~/.bashrc

# System-wide umask
# Edit /etc/login.defs or /etc/profile
```

---

## 3.6 Common Permission Patterns for DevOps

```
┌──────────────────────────────────────────────────────────────┐
│  DEVOPS PERMISSION REFERENCE                                  │
├────────────┬───────────┬─────────────────────────────────────┤
│ Permission │  Octal    │  Use Case                           │
├────────────┼───────────┼─────────────────────────────────────┤
│ rwx------  │   700     │ .ssh/ directory                     │
│ rw-------  │   600     │ SSH private keys, .env files        │
│ rw-r--r--  │   644     │ Config files, HTML, CSS             │
│ rwxr-xr-x  │   755     │ Scripts, directories, binaries      │
│ rw-r-----  │   640     │ Sensitive configs (shadow, DB)      │
│ rwxr-x---  │   750     │ Service directories                 │
│ rw-rw-r--  │   664     │ Shared project files                │
│ rwxrwxr-x  │   775     │ Shared project directories          │
└────────────┴───────────┴─────────────────────────────────────┘
```

---

## 3.7 Real-World Scenario: Web Application Permissions

```bash
#!/bin/bash
# fix-web-permissions.sh

WEB_ROOT="/var/www/myapp"
WEB_USER="www-data"
DEPLOY_USER="deploy"

# Set ownership: deploy user, www-data group
sudo chown -R $DEPLOY_USER:$WEB_USER $WEB_ROOT

# Directories: rwxrwxr-x (775) — deploy + web server can write
find $WEB_ROOT -type d -exec chmod 775 {} +

# Files: rw-rw-r-- (664) — deploy + web server can write
find $WEB_ROOT -type f -exec chmod 664 {} +

# Scripts: executable
find $WEB_ROOT/scripts -type f -name "*.sh" -exec chmod 755 {} +

# Sensitive configs: restricted
chmod 640 $WEB_ROOT/.env
chmod 640 $WEB_ROOT/config/database.yml

# Upload directory: writable by web server
chmod 2775 $WEB_ROOT/storage/uploads   # SGID for group inheritance

echo "Permissions fixed for $WEB_ROOT"
```

---

## Summary Table

| Command | Purpose | Key Flags |
|---------|---------|-----------|
| `chmod` | Change permissions | `-R`, octal (755), symbolic (u+x) |
| `chown` | Change owner/group | `-R`, `user:group` |
| `chgrp` | Change group | `-R` |
| `umask` | Set default permissions | `-S` (symbolic display) |

---

## Quick Revision Questions

1. **Convert `rwxr-x---` to octal.** Convert `641` to symbolic.
2. **What is the difference between permissions on files vs directories?** Explain `x` on a directory.
3. **If `umask` is `027`, what are the default permissions for new files and directories?**
4. **Write a command to recursively set 755 for directories and 644 for files** under `/var/www`.
5. **What permissions should SSH private keys have?** What happens if they're too open?
6. **Why should you avoid `chmod 777`?** What's a better alternative for shared access?

---

[← Previous: Group Management](02-group-management.md) | [Back to README](../README.md) | [Next: Special Permissions →](04-special-permissions.md)
