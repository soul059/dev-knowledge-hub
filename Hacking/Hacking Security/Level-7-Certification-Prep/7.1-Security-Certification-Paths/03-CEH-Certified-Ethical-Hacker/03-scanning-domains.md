# Unit 3: CEH — Topic 3: Scanning Domains

## Overview

The **Scanning** domain covers network discovery, port scanning, service enumeration, and vulnerability identification. This is a core CEH skill area that bridges reconnaissance and exploitation.

---

## 1. Network Discovery

```
HOST DISCOVERY TECHNIQUES:

ARP SCANNING (Layer 2):
  → Most reliable on local network
  → Cannot be blocked by firewall
  → Tools: arp-scan, Nmap -PR
  
  arp-scan 192.168.1.0/24
  nmap -PR -sn 192.168.1.0/24

ICMP SCANNING (Layer 3):
  → Ping sweep: ICMP echo request
  → May be blocked by firewall
  → Types: Echo (8), Reply (0)
  
  nmap -sn -PE 192.168.1.0/24
  fping -a -g 192.168.1.0/24

TCP SCANNING (Layer 4):
  → SYN ping: Send SYN, expect SYN/ACK
  → ACK ping: Bypasses some firewalls
  → Works when ICMP is blocked
  
  nmap -sn -PS22,80,443 192.168.1.0/24
  nmap -sn -PA80 192.168.1.0/24

UDP SCANNING:
  → Send UDP to common ports
  → Open: No response or response
  → Closed: ICMP port unreachable
  → Slow and unreliable
  
  nmap -sn -PU53,161 192.168.1.0/24

NETWORK MAPPING:
  → Traceroute: Map network path
  → Identify routers, firewalls, segments
  → Draw network topology
  → Zenmap: GUI for Nmap
```

---

## 2. Port Scanning Techniques

```
PORT STATES:

  State        | Meaning
  Open         | Service accepting connections
  Closed       | Port reachable, no service
  Filtered     | Firewall blocking probe
  Unfiltered   | ACK scan — port reachable
  Open|Filtered| Cannot determine state
  Closed|Filtered| Cannot determine state

SCAN TYPES DEEP DIVE:

TCP CONNECT (-sT):
  Client → SYN → Target
  Target → SYN/ACK → Client (Open)
  Client → ACK → Target
  Client → RST → Target (Close)
  → Full handshake → logged by target
  → Default for non-root users

SYN SCAN (-sS) "Half-Open/Stealth":
  Client → SYN → Target
  Target → SYN/ACK → Client (Open)
  Client → RST → Target (Abort)
  → No complete connection → less logging
  → Default for root/admin users
  → Fastest TCP scan

FIN SCAN (-sF):
  Client → FIN → Target
  No response = Open|Filtered
  RST = Closed
  → Bypasses some firewalls

XMAS SCAN (-sX):
  Client → FIN+PSH+URG → Target
  No response = Open|Filtered
  RST = Closed
  → "Christmas tree" — all flags lit

NULL SCAN (-sN):
  Client → (no flags) → Target
  No response = Open|Filtered
  RST = Closed
  → Minimal footprint

IDLE/ZOMBIE SCAN (-sI):
  → Uses a third-party "zombie" host
  → Completely anonymous scanning
  → Monitors IP ID increments
  → Complex but powerful
  → Requires predictable IP ID sequence
```

---

## 3. Service and OS Detection

```
SERVICE VERSION DETECTION:

  nmap -sV target.com
  → Connects to open ports
  → Sends probes to identify service
  → Returns: service name, version, info

  nmap -sV --version-intensity 5 target
  → Intensity 0 (light) to 9 (heavy)
  → Default is 7

BANNER GRABBING:
  → Connect to service, read banner
  → Identifies software and version
  
  # Netcat banner grab
  nc -v target 80
  HEAD / HTTP/1.1
  Host: target.com
  
  # Telnet banner grab
  telnet target 22
  → Shows SSH version
  
  # Nmap banner script
  nmap --script banner target

OS DETECTION:
  nmap -O target.com
  → TCP/IP fingerprinting
  → Analyzes: TTL, window size, flags
  → Identifies OS family and version
  
  nmap -A target.com
  → Aggressive: OS + version + scripts + traceroute
  
  OS FINGERPRINTING TECHNIQUES:
  → Active: Send packets, analyze responses
  → Passive: Sniff traffic (p0f)
  → TTL values: Windows=128, Linux=64, Cisco=255
```

---

## 4. Network Scanning Tools

```
SCANNING TOOLS FOR CEH:

NMAP (Network Mapper):
  → Industry standard port scanner
  → Supports all scan types
  → Nmap Scripting Engine (NSE)
  → GUI: Zenmap
  
  # Common exam scenarios:
  nmap -sS -sV -O -A target     # Full scan
  nmap -p- target                 # All 65535 ports
  nmap -sU -p 53,161 target      # UDP scan
  nmap --script vuln target       # Vuln scan
  nmap -sS -D RND:5 target       # Decoy scan

HPING3:
  → Packet crafting tool
  → Custom TCP/UDP/ICMP packets
  → Firewall testing
  → OS fingerprinting
  
  hping3 -S -p 80 target         # SYN to port 80
  hping3 -1 target               # ICMP ping
  hping3 -F -P -U -p 80 target   # XMAS scan
  hping3 --flood target           # DoS test

MASSCAN:
  → Fastest port scanner
  → Scans entire internet in 6 min
  → Rate-limited scanning
  
  masscan -p80,443 10.0.0.0/8 --rate=1000

ANGRY IP SCANNER:
  → GUI-based scanner
  → Lightweight
  → Cross-platform
  → Good for quick discovery

SUPERSCAN / ADVANCED PORT SCANNER:
  → Windows-based scanners
  → GUI for easy use
  → Less powerful than Nmap
```

---

## 5. Scan Analysis and Reporting

```
ANALYZING SCAN RESULTS:

IDENTIFYING KEY SERVICES:
  Port  | Service      | Risk
  21    | FTP          | Clear text, anonymous
  22    | SSH          | Brute force target
  23    | Telnet       | Clear text → replace SSH
  25    | SMTP         | Relay, enumeration
  53    | DNS          | Zone transfer
  80    | HTTP         | Web attacks
  110   | POP3         | Clear text
  135   | RPC          | Windows exploits
  139   | NetBIOS      | SMB attacks
  443   | HTTPS        | SSL/TLS vulnerabilities
  445   | SMB          | EternalBlue, etc.
  1433  | MSSQL        | Database attacks
  3306  | MySQL        | Database attacks
  3389  | RDP          | Brute force, BlueKeep
  8080  | HTTP Proxy   | Web attacks

SCAN DOCUMENTATION:
  → Record all scan parameters
  → Note scan time and duration
  → Document open ports per host
  → Identify risky services
  → Map network topology
  → Screenshot key findings
  → Create vulnerability matrix

EXAM SCENARIO TYPES:
  → "Given this Nmap output, what ports are open?"
  → "Which scan type bypasses firewall?"
  → "What does SYN/ACK response indicate?"
  → "Identify the OS from TTL value"
  → "Which tool performs X scanning?"
```

---

## Summary Table

| Scan Type | Flag | Stealth | Speed | Notes |
|-----------|------|---------|-------|-------|
| TCP Connect | -sT | Low | Slow | Full handshake, logged |
| SYN Scan | -sS | High | Fast | Half-open, default root |
| FIN Scan | -sF | High | Med | Bypasses some firewalls |
| XMAS Scan | -sX | High | Med | FIN+PSH+URG flags |
| NULL Scan | -sN | High | Med | No flags set |
| UDP Scan | -sU | N/A | Very Slow | Unreliable results |
| Idle Scan | -sI | Very High | Slow | Uses zombie host |

---

## Revision Questions

1. What is the difference between SYN scan and TCP Connect scan?
2. How does a XMAS scan work and what flags are set?
3. What are the six port states reported by Nmap?
4. How does OS fingerprinting work using TTL values?
5. What is an idle/zombie scan and why is it considered stealthy?

---

*Previous: [02-reconnaissance-domains.md](02-reconnaissance-domains.md) | Next: [04-exploitation-domains.md](04-exploitation-domains.md)*

---

*[Back to README](../README.md)*
