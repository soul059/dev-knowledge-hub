# Service Enumeration

## Unit 6 - Topic 3 | Scanning and Enumeration

---

## Overview

**Service enumeration** goes beyond port scanning to extract detailed information about each running service — version numbers, configuration details, supported features, and user accounts. This information identifies specific attack vectors and exploitable weaknesses.

---

## 1. Enumeration by Service

### SMB Enumeration (Ports 139, 445)

```bash
# Share enumeration
smbclient -L //10.0.0.5 -N            # List shares (null session)
smbmap -H 10.0.0.5                     # Map shares and permissions
smbmap -H 10.0.0.5 -u '' -p ''        # Null session

# User and group enumeration
enum4linux -a 10.0.0.5                 # Comprehensive SMB enum
rpcclient -U "" -N 10.0.0.5           # RPC null session
> enumdomusers                         # List domain users
> enumdomgroups                        # List domain groups
> lookupnames administrator            # SID lookup

# Vulnerability checking
nmap --script smb-vuln* -p 445 10.0.0.5
# Checks: EternalBlue (MS17-010), SMBv1, etc.

# CrackMapExec
crackmapexec smb 10.0.0.0/24          # SMB scan entire subnet
crackmapexec smb 10.0.0.5 --shares    # List shares
crackmapexec smb 10.0.0.5 --users     # List users
```

### DNS Enumeration (Port 53)

```bash
dig @10.0.0.5 target.com ANY          # All records
dig @10.0.0.5 target.com axfr         # Zone transfer attempt
dnsrecon -d target.com -t std         # Standard enum
dnsenum target.com                    # Comprehensive DNS enum
```

### SMTP Enumeration (Port 25)

```bash
# User enumeration via VRFY
nmap --script smtp-enum-users -p 25 10.0.0.5

# Manual SMTP enumeration
telnet 10.0.0.5 25
> VRFY admin                           # Verify user exists
> EXPN admin                           # Expand mailing list
> RCPT TO: admin@target.com            # Check if recipient valid

# smtp-user-enum tool
smtp-user-enum -M VRFY -U users.txt -t 10.0.0.5
```

### SNMP Enumeration (Port 161)

```bash
# Community string brute force
onesixtyone -c /usr/share/wordlists/community.txt 10.0.0.5

# Walk SNMP MIB tree
snmpwalk -v 2c -c public 10.0.0.5
snmpwalk -v 2c -c public 10.0.0.5 1.3.6.1.2.1.25.1  # System info
snmpwalk -v 2c -c public 10.0.0.5 1.3.6.1.4.1.77.1.2.25  # Users

# Nmap SNMP scripts
nmap -sU -p 161 --script snmp-brute 10.0.0.5
nmap -sU -p 161 --script snmp-info 10.0.0.5
```

---

## 2. Web Service Enumeration (Ports 80, 443)

```bash
# Technology detection
whatweb https://target.com
curl -I https://target.com              # HTTP headers

# Directory enumeration
gobuster dir -u https://target.com -w /usr/share/seclists/Discovery/Web-Content/common.txt
ffuf -u https://target.com/FUZZ -w wordlist.txt

# API enumeration
gobuster dir -u https://target.com/api -w api_wordlist.txt
# Look for: /api/v1/, /api/users, /api/admin, /swagger, /graphql

# CMS detection and enumeration
wpscan --url https://target.com        # WordPress
droopescan scan drupal -u https://target.com  # Drupal
joomscan --url https://target.com      # Joomla
```

---

## 3. Database Enumeration

```bash
# MySQL (Port 3306)
nmap -sV -p 3306 --script mysql-info 10.0.0.5
nmap --script mysql-enum -p 3306 10.0.0.5
mysql -h 10.0.0.5 -u root -p          # Try default creds

# MSSQL (Port 1433)
nmap --script ms-sql-info -p 1433 10.0.0.5
nmap --script ms-sql-brute -p 1433 10.0.0.5

# PostgreSQL (Port 5432)
nmap --script pgsql-brute -p 5432 10.0.0.5

# Redis (Port 6379)
redis-cli -h 10.0.0.5
> INFO                                 # Server information
> KEYS *                               # List all keys
```

---

## 4. LDAP Enumeration (Ports 389, 636)

```bash
# LDAP enumeration (Active Directory)
ldapsearch -x -H ldap://10.0.0.5 -b "dc=target,dc=com"
nmap --script ldap-search -p 389 10.0.0.5

# Extract users, groups, OUs
ldapsearch -x -H ldap://10.0.0.5 -b "dc=target,dc=com" "(objectClass=user)"
```

---

## 5. Enumeration Checklist

| Service | Port | Key Enumeration |
|---------|:----:|----------------|
| **SMB** | 445 | Shares, users, groups, vulns |
| **DNS** | 53 | Zone transfer, subdomains |
| **SMTP** | 25 | User verification (VRFY) |
| **SNMP** | 161 | Community strings, MIB walk |
| **HTTP** | 80/443 | Directories, CMS, APIs |
| **LDAP** | 389 | Users, groups, AD structure |
| **MySQL** | 3306 | Default creds, databases |
| **RDP** | 3389 | NLA, version, screenshot |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Enumeration** | Extract detailed info from discovered services |
| **SMB** | Shares, users via null sessions |
| **SNMP** | Community strings reveal everything |
| **SMTP** | VRFY command enumerates users |
| **Web** | Directories, APIs, CMS detection |
| **LDAP** | Active Directory structure |

---

## Quick Revision Questions

1. **What is the difference between scanning and enumeration?**
2. **How do you enumerate SMB shares without credentials?**
3. **What is an SNMP community string?**
4. **How can SMTP be used for user enumeration?**
5. **Name 3 tools for web directory enumeration.**

---

[← Previous: Port Scanning](02-port-scanning.md) | [Next: Vulnerability Scanning →](04-vulnerability-scanning.md)

---

*Unit 6 - Topic 3 of 6 | [Back to README](../README.md)*
