# BloodHound Basics

## Unit 2 - Topic 4 | AD Enumeration

---

## Overview

**BloodHound** is a graph-based Active Directory analysis tool that maps relationships between AD objects and visualizes attack paths to high-value targets. It reveals hidden privilege escalation paths that would be nearly impossible to find manually.

---

## 1. BloodHound Architecture

```
BLOODHOUND WORKFLOW:

1. DATA COLLECTION (SharpHound/BloodHound.py)
   ├── Queries AD via LDAP
   ├── Enumerates sessions, ACLs, trusts
   ├── Collects group memberships
   └── Outputs JSON/ZIP files

2. DATA IMPORT (BloodHound GUI)
   ├── Import JSON/ZIP into Neo4j database
   └── Creates graph of all relationships

3. ANALYSIS (Graph Queries)
   ├── Pre-built queries for common attacks
   ├── Custom Cypher queries
   ├── Visual path finding
   └── Identify shortest path to Domain Admin

ARCHITECTURE:
┌───────────────┐     ┌─────────────┐     ┌──────────────┐
│  SharpHound   │ ──→ │   Neo4j     │ ←── │  BloodHound  │
│  (Collector)  │     │  (Database) │     │    (GUI)     │
│               │     │             │     │              │
│ Runs on       │     │ Stores      │     │ Visualizes   │
│ target domain │     │ graph data  │     │ attack paths │
└───────────────┘     └─────────────┘     └──────────────┘
```

---

## 2. Data Collection

```bash
# === SHARPHOUND (Windows — C# Collector) ===
# All collection methods:
.\SharpHound.exe -c all
# Collects: sessions, group memberships, ACLs, trusts, etc.

# Specific collection:
.\SharpHound.exe -c DCOnly        # DC data only (stealthier)
.\SharpHound.exe -c Session       # Session data only
.\SharpHound.exe -c Group         # Group memberships
.\SharpHound.exe -c ACL           # ACL data
.\SharpHound.exe -c ObjectProps   # Object properties

# With stealth:
.\SharpHound.exe -c all --stealth
# Only queries DCs, not every workstation

# Loop collection (gather sessions over time):
.\SharpHound.exe -c Session --loop --loopduration 02:00:00
# Collects sessions every few minutes for 2 hours
# More sessions = better attack path coverage

# Output: ZIP file (e.g., 20240115120000_BloodHound.zip)

# === BLOODHOUND.PY (Linux — Python Collector) ===
bloodhound-python -u user -p password -d corp.local \
  -ns dc01_ip -c all
# -c all: All collection methods
# -ns: Nameserver (DC for DNS)

# With NTLM hash:
bloodhound-python -u user --hashes :NTLM_HASH \
  -d corp.local -ns dc01_ip -c all
```

---

## 3. BloodHound GUI Setup

```bash
# === INSTALLATION ===
# Install Neo4j:
sudo apt install neo4j
sudo neo4j start
# Access: http://localhost:7474
# Default creds: neo4j/neo4j (change on first login)

# Install BloodHound:
sudo apt install bloodhound
# Or download from GitHub releases

# === LAUNCH ===
bloodhound
# Login with Neo4j credentials

# === IMPORT DATA ===
# Drag and drop ZIP file into BloodHound GUI
# Or: Upload Data button → Select ZIP file
# Wait for import to complete

# === BLOODHOUND CE (Community Edition) ===
# Newer version with Docker:
curl -L https://ghst.ly/getbhce | docker compose -f - up
# Access: http://localhost:8080
```

---

## 4. Key BloodHound Queries

```
PRE-BUILT QUERIES (Most Important):

SHORTEST PATHS:
├── "Find Shortest Paths to Domain Admins"
│   → Shows how to reach DA from any user
├── "Shortest Paths to Domain Admins from Owned Principals"
│   → From YOUR compromised accounts to DA
└── "Shortest Paths to High Value Targets"

KERBEROS:
├── "List all Kerberoastable Accounts"
├── "Find AS-REP Roastable Users"
└── "Find Principals with DCSync Rights"

DELEGATION:
├── "Find Computers with Unconstrained Delegation"
├── "Find Users with Constrained Delegation"
└── "Shortest Paths from Unconstrained Delegation Systems"

GROUP:
├── "Find all Domain Admins"
├── "Map Domain Trusts"
└── "Find Groups with Most Local Admin"

CUSTOM CYPHER QUERIES:
# Users with path to DA:
MATCH p=shortestPath((u:User)-[*1..]->(g:Group {name:"DOMAIN ADMINS@CORP.LOCAL"}))
RETURN p

# Kerberoastable users with path to DA:
MATCH (u:User {hasspn:true})
MATCH p=shortestPath((u)-[*1..]->(g:Group {name:"DOMAIN ADMINS@CORP.LOCAL"}))
RETURN p

# Find all computers where DA has a session:
MATCH (c:Computer)-[:HasSession]->(u:User)-[:MemberOf*1..]->(g:Group {name:"DOMAIN ADMINS@CORP.LOCAL"})
RETURN c.name, u.name
```

---

## 5. Interpreting BloodHound Results

```
EDGE TYPES (Relationships):

MEMBERSHIP:
├── MemberOf      → User is member of group
└── HasSession    → User has active session on computer

ADMIN:
├── AdminTo       → Has admin rights on computer
├── CanRDP        → Can RDP to computer
└── CanPSRemote   → Can PowerShell remote

ACL ABUSE:
├── GenericAll    → Full control over object
├── GenericWrite  → Can modify attributes
├── WriteDacl     → Can modify permissions
├── WriteOwner   → Can change owner
├── ForceChangePassword → Can reset password
├── AddMember     → Can add members to group
└── AllExtendedRights → Multiple dangerous rights

SPECIAL:
├── DCSync        → Can replicate DC (dump all hashes!)
├── GPLink        → GPO linked to OU
├── Contains      → OU contains object
├── TrustedBy     → Trust relationship
└── AllowedToDelegate → Delegation target

MARKING OWNED NODES:
├── Right-click node → "Mark as Owned"
├── Shows paths FROM your owned accounts
├── Track your progress through the domain
└── Update as you compromise more accounts
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **BloodHound** | Graph-based AD attack path analysis |
| **SharpHound** | Windows data collector (C#) |
| **bloodhound-python** | Linux data collector |
| **Neo4j** | Graph database backend |
| **Key query** | "Shortest Path to Domain Admins" |
| **Edge types** | GenericAll, DCSync, AdminTo = critical |

---

## Quick Revision Questions

1. **What does BloodHound use to store its data?**
2. **What is the difference between SharpHound and bloodhound-python?**
3. **What does the "Mark as Owned" feature do?**
4. **What edge type indicates the ability to dump all domain hashes?**
5. **Why should you run session collection in a loop?**

---

[← Previous: PowerShell Enumeration](03-powershell-enumeration.md) | [Next: User and Group Enumeration →](05-user-and-group-enumeration.md)

---

*Unit 2 - Topic 4 of 6 | [Back to README](../README.md)*
