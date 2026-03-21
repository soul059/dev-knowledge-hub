# Default Credentials

## Unit 3 - Topic 2 | Network Vulnerabilities

---

## Overview

**Default credentials** are factory-set usernames and passwords that ship with devices, software, and services. Many organizations fail to change these defaults, creating easy entry points for attackers. This is consistently one of the most exploited vulnerability classes.

---

## 1. Common Default Credentials

| Device/Service | Username | Password |
|---------------|----------|----------|
| **Cisco Router** | admin | admin, cisco, password |
| **Netgear Router** | admin | password |
| **Tomcat Manager** | tomcat | tomcat, s3cret |
| **Jenkins** | admin | admin (or no password) |
| **phpMyAdmin** | root | (blank), root |
| **MySQL** | root | (blank), mysql |
| **PostgreSQL** | postgres | postgres |
| **MongoDB** | (none) | (no auth by default!) |
| **Redis** | (none) | (no auth by default!) |
| **SSH (IoT)** | root | root, toor, admin |
| **SNMP** | (community) | public, private |
| **VNC** | (none) | password |
| **Elasticsearch** | (none) | (no auth by default!) |

---

## 2. Finding Default Credentials

```bash
# === ONLINE DATABASES ===
# https://defaultcredentials.com/
# https://cirt.net/passwords
# https://datarecovery.com/rd/default-passwords/
# https://www.routerpasswords.com/
# https://open-sez.me/

# === NMAP DEFAULT CREDENTIAL CHECK ===
nmap -p 80 --script http-default-accounts target.com
nmap -p 21 --script ftp-anon target.com          # Anonymous FTP
nmap -p 1433 --script ms-sql-empty-password target.com
nmap -p 3306 --script mysql-empty-password target.com

# === METASPLOIT SCANNERS ===
msf> use auxiliary/scanner/http/tomcat_mgr_login
msf> set RHOSTS target.com
msf> run
# Tests common Tomcat default credentials

msf> use auxiliary/scanner/ssh/ssh_login
msf> set RHOSTS target.com
msf> set USERNAME root
msf> set PASS_FILE /usr/share/wordlists/default_passwords.txt
msf> run

# === MANUAL TESTING ===
# Test service with common defaults:
ssh root@target.com          # Try: root/toor, admin/admin
mysql -h target.com -u root  # Try blank password
redis-cli -h target.com      # No auth needed by default
curl http://target.com:8080/manager/html  # Tomcat default
```

---

## 3. Commonly Overlooked Defaults

```
FREQUENTLY MISSED DEFAULT CREDENTIALS:

NETWORK DEVICES:
├── Switches, routers: admin/admin
├── Firewalls: admin/password
├── WiFi APs: admin/admin
├── IPMI/iDRAC/iLO: admin/admin or ADMIN/ADMIN
└── UPS systems: apc/apc

WEB APPLICATIONS:
├── CMS admin panels: admin/admin
├── WordPress: check wp-login.php
├── phpMyAdmin: root/(blank)
├── Grafana: admin/admin
└── Kibana: no auth by default

DATABASES:
├── MongoDB: no auth required by default!
├── Redis: no auth by default!
├── Elasticsearch: no auth by default!
├── CouchDB: admin party (all admin by default)
└── Memcached: no auth

MANAGEMENT INTERFACES:
├── IPMI: admin/admin (port 623)
├── iDRAC: root/calvin
├── iLO: Administrator/(on sticker)
├── VMware ESXi: root/(set during install)
└── Proxmox: root/(set during install)

IoT DEVICES:
├── IP cameras: admin/admin, admin/12345
├── Printers: admin/(blank)
├── Smart TVs: varies by manufacturer
└── Industrial control: varies widely
```

---

## 4. Exploitation Impact

```
DEFAULT CREDENTIAL ATTACK CHAIN:

1. DISCOVERY
   └── Nmap finds Tomcat on port 8080

2. DEFAULT LOGIN
   └── admin:tomcat works on /manager/html

3. WAR FILE UPLOAD
   └── Deploy malicious .war file via manager

4. REVERSE SHELL
   └── Msfvenom WAR payload → reverse shell

5. PRIVILEGE ESCALATION
   └── Tomcat runs as root → game over

RESULT: Full server compromise from default credentials!
```

---

## 5. Defensive Countermeasures

| Countermeasure | Implementation |
|---------------|----------------|
| **Change defaults** | Immediately on deployment |
| **Strong passwords** | Enforce complexity requirements |
| **Remove unused accounts** | Delete default/test accounts |
| **Audit regularly** | Scan for default credentials |
| **Disable default services** | Turn off what isn't needed |
| **Network segmentation** | Limit access to management interfaces |
| **MFA** | Add multi-factor for admin access |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Default creds** | Factory passwords never changed |
| **Databases** | MongoDB, Redis, Elasticsearch — no auth! |
| **IPMI/iDRAC** | Server management often has defaults |
| **Nmap scripts** | `http-default-accounts`, `*-empty-password` |
| **Impact** | Often leads to full system compromise |
| **Defense** | Change on deployment, audit regularly |

---

## Quick Revision Questions

1. **Why are default credentials a critical vulnerability?**
2. **Name 3 services that have no authentication by default.**
3. **Where can you find default credential databases?**
4. **What Nmap scripts test for default credentials?**
5. **What attack chain can follow from default Tomcat credentials?**

---

[← Previous: Common Network Vulnerabilities](01-common-network-vulnerabilities.md) | [Next: Unpatched Services →](03-unpatched-services.md)

---

*Unit 3 - Topic 2 of 6 | [Back to README](../README.md)*
