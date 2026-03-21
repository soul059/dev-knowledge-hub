# Protocol-Specific Enumeration

## Unit 2 - Topic 4 | Service Enumeration

---

## Overview

**Protocol-specific enumeration** uses targeted techniques for each service type to extract maximum information. Each protocol has unique enumeration methods, tools, and data points that generic scanning misses.

---

## 1. SMB Enumeration (Port 445/139)

```bash
# === ENUM4LINUX (All-in-One SMB Enum) ===
enum4linux -a target.com
# Returns: users, shares, groups, OS, password policy

# === SMBCLIENT ===
smbclient -L //target.com -N          # List shares (null session)
smbclient //target.com/share -N       # Connect to share

# === SMBMAP ===
smbmap -H target.com                  # List shares and permissions
smbmap -H target.com -R              # Recursive listing
smbmap -H target.com -u "" -p ""     # Null session

# === CRACKMAPEXEC (CME) ===
crackmapexec smb target.com           # OS, hostname, domain
crackmapexec smb target.com --shares  # Enumerate shares
crackmapexec smb target.com --users   # Enumerate users

# === NMAP SMB SCRIPTS ===
nmap -p 445 --script smb-enum-shares,smb-enum-users target.com
nmap -p 445 --script smb-os-discovery target.com
```

---

## 2. DNS Enumeration (Port 53)

```bash
# === DIG ===
dig target.com ANY                    # All record types
dig target.com AXFR @ns.target.com   # Zone transfer (if allowed!)
dig -x 203.0.113.10                  # Reverse lookup

# === DNSRECON ===
dnsrecon -d target.com -t std        # Standard enumeration
dnsrecon -d target.com -t brt        # Brute-force subdomains
dnsrecon -d target.com -t axfr       # Zone transfer attempt

# === DNSENUM ===
dnsenum target.com                    # Full DNS enumeration

# === NMAP DNS ===
nmap --script dns-brute target.com    # Subdomain brute-force
nmap --script dns-zone-transfer -p 53 ns.target.com
```

---

## 3. SNMP Enumeration (Port 161)

```bash
# === SNMPWALK ===
snmpwalk -v2c -c public target.com
# Walks entire MIB tree — reveals EVERYTHING:
# System info, interfaces, routes, processes, installed software

# Specific OIDs:
snmpwalk -v2c -c public target.com 1.3.6.1.2.1.1     # System info
snmpwalk -v2c -c public target.com 1.3.6.1.2.1.25.4   # Running processes
snmpwalk -v2c -c public target.com 1.3.6.1.2.1.25.6   # Installed software

# === SNMP-CHECK ===
snmp-check target.com -c public
# Formatted output: system info, processes, network

# === ONESIXTYONE (Community String Brute-Force) ===
onesixtyone -c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt target.com
# Tries common community strings: public, private, manager, etc.

# === NMAP SNMP ===
nmap -sU -p 161 --script snmp-info,snmp-processes,snmp-interfaces target.com
```

---

## 4. LDAP Enumeration (Port 389/636)

```bash
# === LDAPSEARCH ===
ldapsearch -x -H ldap://target.com -b "dc=target,dc=com"
# Anonymous bind — lists directory entries

# Specific queries:
ldapsearch -x -H ldap://target.com -b "dc=target,dc=com" "(objectClass=user)"
ldapsearch -x -H ldap://target.com -b "dc=target,dc=com" "(objectClass=computer)"

# === NMAP LDAP ===
nmap -p 389 --script ldap-rootdse target.com         # Root DSE info
nmap -p 389 --script ldap-search target.com          # Search directory

# === WINDAPSEARCH ===
windapsearch -d target.com --dc dc01.target.com -U   # Enumerate users
windapsearch -d target.com --dc dc01.target.com -G   # Enumerate groups
```

---

## 5. Other Protocol Enumeration

```bash
# === NFS (Port 2049) ===
showmount -e target.com               # List exported shares
nmap -p 111 --script nfs-ls,nfs-showmount target.com
mount -t nfs target.com:/share /mnt   # Mount if accessible

# === RPC (Port 111) ===
rpcinfo -p target.com                 # List RPC services

# === SMTP (Port 25) ===
nmap -p 25 --script smtp-enum-users target.com
smtp-user-enum -M VRFY -U users.txt -t target.com

# === MYSQL (Port 3306) ===
nmap -p 3306 --script mysql-info,mysql-enum target.com
mysql -h target.com -u root           # Try default credentials

# === RDP (Port 3389) ===
nmap -p 3389 --script rdp-enum-encryption target.com
nmap -p 3389 --script rdp-ntlm-info target.com
```

---

## Protocol Enumeration Quick Reference

| Protocol | Port | Key Tool | Key Info |
|----------|:----:|----------|----------|
| **SMB** | 445 | enum4linux, CME | Users, shares, OS |
| **DNS** | 53 | dig, dnsrecon | Subdomains, zone transfers |
| **SNMP** | 161 | snmpwalk | System info, processes |
| **LDAP** | 389 | ldapsearch | Users, groups, computers |
| **NFS** | 2049 | showmount | Exported shares |
| **SMTP** | 25 | smtp-user-enum | Valid email addresses |
| **MySQL** | 3306 | nmap scripts | Version, databases |
| **RDP** | 3389 | nmap scripts | Encryption, NTLM info |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **SMB** | enum4linux for all-in-one enumeration |
| **DNS** | Zone transfers = full domain listing |
| **SNMP** | Community strings reveal everything |
| **LDAP** | Anonymous bind may list all users |
| **NFS** | showmount to find mountable shares |
| **Approach** | Each protocol needs specific tools |

---

## Quick Revision Questions

1. **What tool provides all-in-one SMB enumeration?**
2. **What is a DNS zone transfer and why is it dangerous?**
3. **What SNMP community string should you test first?**
4. **How do you enumerate users via LDAP?**
5. **What information does NFS showmount reveal?**

---

[← Previous: Script Scanning (NSE)](03-script-scanning-nse.md) | [Next: Service Fingerprinting →](05-service-fingerprinting.md)

---

*Unit 2 - Topic 4 of 6 | [Back to README](../README.md)*
