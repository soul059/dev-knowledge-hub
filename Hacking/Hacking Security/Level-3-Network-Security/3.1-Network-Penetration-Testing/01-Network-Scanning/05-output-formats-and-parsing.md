# Output Formats and Parsing

## Unit 1 - Topic 5 | Network Scanning

---

## Overview

**Nmap output formats** store scan results in different structures optimized for human reading, scripting, or import into other tools. Effective parsing transforms raw scan data into actionable intelligence for the next phases of penetration testing.

---

## 1. Nmap Output Formats

```bash
# === SAVE IN ALL FORMATS ===
nmap -oA scan_results target.com
# Creates: scan_results.nmap (normal)
#          scan_results.xml  (XML)
#          scan_results.gnmap (grepable)

# === INDIVIDUAL FORMATS ===
nmap -oN scan.txt target.com       # Normal (human-readable)
nmap -oX scan.xml target.com       # XML (tool import)
nmap -oG scan.gnmap target.com     # Grepable (one host per line)
nmap -oS scan.skid target.com      # Script kiddie (just for fun)

# === APPEND TO FILE ===
nmap --append-output -oN scan.txt target2.com
# Adds to existing file instead of overwriting
```

---

## 2. Format Examples

```bash
# === NORMAL OUTPUT (-oN) ===
# Nmap scan report for target.com (203.0.113.10)
# Host is up (0.025s latency).
# PORT     STATE SERVICE    VERSION
# 22/tcp   open  ssh        OpenSSH 8.9p1
# 80/tcp   open  http       Apache httpd 2.4.49
# 443/tcp  open  ssl/http   nginx 1.24.0
# 3306/tcp open  mysql      MySQL 5.7.40

# === GREPABLE OUTPUT (-oG) ===
# Host: 203.0.113.10 (target.com) Status: Up
# Host: 203.0.113.10 (target.com) Ports: 22/open/tcp//ssh//OpenSSH 8.9p1/, 80/open/tcp//http//Apache httpd 2.4.49/, 443/open/tcp//ssl|http//nginx 1.24.0/, 3306/open/tcp//mysql//MySQL 5.7.40/

# === XML OUTPUT (-oX) ===
# <host>
#   <address addr="203.0.113.10" addrtype="ipv4"/>
#   <hostnames><hostname name="target.com"/></hostnames>
#   <ports>
#     <port protocol="tcp" portid="22">
#       <state state="open"/>
#       <service name="ssh" product="OpenSSH" version="8.9p1"/>
#     </port>
#     ...
#   </ports>
# </host>
```

---

## 3. Parsing Nmap Output

```bash
# === PARSING GREPABLE OUTPUT ===
# Extract open ports for a host:
grep "203.0.113.10" scan.gnmap | grep -oP '\d+/open'
# 22/open
# 80/open
# 443/open

# Extract all alive hosts:
grep "Status: Up" scan.gnmap | cut -d" " -f2
# 203.0.113.10
# 203.0.113.20

# Extract hosts with specific port open:
grep "80/open" scan.gnmap | cut -d" " -f2
# All hosts with port 80 open

# === PARSING XML OUTPUT ===
# Using xmlstarlet:
xmlstarlet sel -t -m "//port[@portid='80']" \
  -v "../../../address/@addr" -n scan.xml

# Using Python:
python3 -c "
import xml.etree.ElementTree as ET
tree = ET.parse('scan.xml')
for host in tree.findall('.//host'):
    ip = host.find('address').get('addr')
    for port in host.findall('.//port'):
        state = port.find('state').get('state')
        portid = port.get('portid')
        service = port.find('service')
        name = service.get('name') if service is not None else 'unknown'
        print(f'{ip}:{portid} ({name}) - {state}')
"

# === PARSING NORMAL OUTPUT ===
# Quick port list:
grep "open" scan.txt | grep -v "#" | awk '{print $1}'
# 22/tcp
# 80/tcp
# 443/tcp
```

---

## 4. Converting and Importing

```bash
# === CONVERT XML TO HTML REPORT ===
xsltproc scan.xml -o report.html
# Uses Nmap's built-in stylesheet

# === IMPORT INTO METASPLOIT ===
# In msfconsole:
msf> db_import scan.xml
msf> hosts
msf> services
msf> vulns

# === IMPORT INTO SEARCHSPLOIT ===
searchsploit --nmap scan.xml
# Automatically searches exploits for detected services

# === CONVERT TO CSV ===
python3 -c "
import xml.etree.ElementTree as ET
import csv
tree = ET.parse('scan.xml')
with open('scan.csv', 'w', newline='') as f:
    writer = csv.writer(f)
    writer.writerow(['IP', 'Port', 'Protocol', 'State', 'Service', 'Version'])
    for host in tree.findall('.//host'):
        ip = host.find('address').get('addr')
        for port in host.findall('.//port'):
            portid = port.get('portid')
            proto = port.get('protocol')
            state = port.find('state').get('state')
            svc = port.find('service')
            name = svc.get('name', '') if svc is not None else ''
            ver = svc.get('version', '') if svc is not None else ''
            writer.writerow([ip, portid, proto, state, name, ver])
"
```

---

## 5. Quick Reference

| Format | Flag | Best For | Extension |
|--------|:----:|---------|:---------:|
| Normal | `-oN` | Human reading | .nmap |
| XML | `-oX` | Tool import (MSF, etc.) | .xml |
| Grepable | `-oG` | Quick grep/awk parsing | .gnmap |
| All | `-oA` | Save everything | .nmap/.xml/.gnmap |

| Tool | Import Command |
|------|---------------|
| **Metasploit** | `db_import scan.xml` |
| **Searchsploit** | `searchsploit --nmap scan.xml` |
| **Nmap to HTML** | `xsltproc scan.xml -o report.html` |
| **Custom scripts** | Parse XML with Python/Ruby |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **-oA** | Save in all formats simultaneously |
| **-oN** | Human-readable normal output |
| **-oX** | XML for tool import |
| **-oG** | Grepable for quick parsing |
| **Metasploit** | `db_import scan.xml` |
| **searchsploit** | `--nmap scan.xml` for auto exploit search |

---

## Quick Revision Questions

1. **What does `-oA` do and what files does it create?**
2. **Which output format is best for importing into Metasploit?**
3. **How do you extract all alive hosts from grepable output?**
4. **How does `searchsploit --nmap` work?**
5. **How do you convert Nmap XML to an HTML report?**

---

[← Previous: Scan Timing and Evasion](04-scan-timing-and-evasion.md) | [Next: Masscan for Large Networks →](06-masscan-for-large-networks.md)

---

*Unit 1 - Topic 5 of 6 | [Back to README](../README.md)*
