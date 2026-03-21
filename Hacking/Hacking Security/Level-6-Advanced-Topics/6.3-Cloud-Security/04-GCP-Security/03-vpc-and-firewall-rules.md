# Unit 4: GCP Security — Topic 3: VPC and Firewall Rules

## Overview

**Google Cloud VPC (Virtual Private Cloud)** provides networking for GCP resources. VPC networks in GCP are global (unlike AWS/Azure), and firewall rules control traffic flow. Understanding VPC architecture and firewall configuration is critical for network segmentation, defense in depth, and security assessment in Google Cloud.

---

## 1. VPC Fundamentals

```
GCP VPC CHARACTERISTICS:

  → Global resource (spans all regions)
  → Subnets are regional (span all zones in region)
  → No broadcast domain
  → Software-defined networking
  → Supports IPv4 and IPv6
  → Two types: Auto mode and Custom mode

AUTO MODE VPC:
  → Created with default subnets in every region
  → /20 subnets (4096 IPs each)
  → Quick to set up
  → NOT recommended for production
  → Limited control over IP ranges

CUSTOM MODE VPC:
  → No default subnets
  → You create subnets with specific CIDRs
  → Full control over IP ranges
  → Recommended for production
  → Better for security segmentation

VPC ARCHITECTURE:

  ┌─────────────────────────────────────────┐
  │        VPC Network (Global)              │
  │                                         │
  │  ┌─────────────────┐ ┌────────────────┐ │
  │  │ us-central1      │ │ europe-west1   │ │
  │  │ Subnet: 10.0.1/24│ │ Subnet:10.0.2/24│
  │  │                  │ │                │ │
  │  │ ┌──┐ ┌──┐       │ │ ┌──┐ ┌──┐     │ │
  │  │ │VM│ │VM│       │ │ │VM│ │VM│     │ │
  │  │ └──┘ └──┘       │ │ └──┘ └──┘     │ │
  │  └─────────────────┘ └────────────────┘ │
  │                                         │
  │  Internal routing between all subnets    │
  │  (automatic, no additional config)       │
  └─────────────────────────────────────────┘

SHARED VPC:
  → Centralized network management
  → Host project owns VPC
  → Service projects use shared subnets
  → Centralized firewall rules
  → Network admins in host project
  → Resource creators in service projects

VPC PEERING:
  → Connect two VPC networks
  → Private IP communication
  → No transitive peering (A↔B, B↔C ≠ A↔C)
  → Non-overlapping CIDRs required
  → No single point of failure
```

---

## 2. Firewall Rules

```
GCP FIREWALL RULES:

CHARACTERISTICS:
  → Applied at VPC level (global)
  → Stateful (return traffic allowed)
  → Target: All instances, tagged, or service account
  → Direction: Ingress or Egress
  → Priority: 0-65535 (lower = higher priority)

DEFAULT RULES:
  → default-allow-internal: Allow all internal traffic
  → default-allow-ssh: Allow SSH (22) from anywhere
  → default-allow-rdp: Allow RDP (3389) from anywhere
  → default-allow-icmp: Allow ICMP from anywhere
  ⚠ Default rules are permissive — tighten them!

RULE COMPONENTS:
  → Direction: INGRESS or EGRESS
  → Priority: 0-65535
  → Action: ALLOW or DENY
  → Target: All, target tags, service accounts
  → Source (ingress): IP ranges, tags, service accounts
  → Destination (egress): IP ranges
  → Protocol/Ports: TCP:80, UDP:53, ICMP, all

EXAMPLE RULES:
  # Allow web traffic
  gcloud compute firewall-rules create allow-web \
    --network=my-vpc \
    --direction=INGRESS \
    --action=ALLOW \
    --rules=tcp:80,tcp:443 \
    --source-ranges=0.0.0.0/0 \
    --target-tags=web-server \
    --priority=1000

  # Allow SSH from specific IP
  gcloud compute firewall-rules create allow-ssh-admin \
    --network=my-vpc \
    --direction=INGRESS \
    --action=ALLOW \
    --rules=tcp:22 \
    --source-ranges=203.0.113.10/32 \
    --target-tags=ssh-allowed \
    --priority=900

  # Deny all ingress (catch-all)
  gcloud compute firewall-rules create deny-all-ingress \
    --network=my-vpc \
    --direction=INGRESS \
    --action=DENY \
    --rules=all \
    --source-ranges=0.0.0.0/0 \
    --priority=65534

FIREWALL PROCESSING:
  1. Check for DENY rules (highest priority first)
  2. Check for ALLOW rules (highest priority first)
  3. If no match → implied deny-all (ingress)
  4. If no match → implied allow-all (egress)
```

---

## 3. Cloud Armor

```
CLOUD ARMOR:

PURPOSE:
  → DDoS protection (Layer 3/4/7)
  → Web Application Firewall (WAF)
  → IP allowlisting/denylisting
  → Geographic-based access control
  → Pre-configured WAF rules (OWASP Top 10)

COMPONENTS:
  → Security policies
  → Rules within policies
  → Backend service association
  → Works with external HTTP(S) Load Balancers

EXAMPLE RULES:
  # Create security policy
  gcloud compute security-policies create my-policy
  
  # Block specific country
  gcloud compute security-policies rules create 1000 \
    --security-policy=my-policy \
    --expression="origin.region_code == 'CN'" \
    --action=deny-403
  
  # Block SQL injection
  gcloud compute security-policies rules create 2000 \
    --security-policy=my-policy \
    --expression="evaluatePreconfiguredExpr('sqli-v33-stable')" \
    --action=deny-403
  
  # Rate limiting
  gcloud compute security-policies rules create 3000 \
    --security-policy=my-policy \
    --expression="true" \
    --action=rate-based-ban \
    --rate-limit-threshold-count=100 \
    --rate-limit-threshold-interval-sec=60 \
    --ban-duration-sec=300

PRE-CONFIGURED RULES:
  → SQL injection (sqli)
  → Cross-site scripting (xss)
  → Local file inclusion (lfi)
  → Remote file inclusion (rfi)
  → Remote code execution (rce)
  → Method enforcement
  → Scanner detection
  → Protocol attack
```

---

## 4. VPC Service Controls

```
VPC SERVICE CONTROLS:

PURPOSE:
  → Create security perimeters around GCP services
  → Prevent data exfiltration
  → Control API-level access
  → Independent from network-level controls

HOW IT WORKS:
  ┌──────────────────────────────────┐
  │      VPC Service Perimeter        │
  │                                  │
  │  ┌─────────┐  ┌──────────────┐  │
  │  │ Project1 │  │ Project2     │  │
  │  │ BigQuery │  │ Cloud Storage│  │
  │  │ GCS      │  │ Pub/Sub     │  │
  │  └─────────┘  └──────────────┘  │
  │                                  │
  │  Data can flow between projects  │
  │  inside perimeter                │
  │                                  │
  │  Data CANNOT leave perimeter     │
  │  (even if user has IAM access)   │
  └──────────────────────────────────┘
  
  Outside:
  → Cannot read/write to protected services
  → Even with valid credentials
  → Blocks exfiltration via API

ACCESS LEVELS:
  → Conditions for allowing access to perimeter
  → IP-based (corporate network)
  → Device-based (managed devices)
  → Identity-based (specific users)
  → Geographic-based

INGRESS/EGRESS RULES:
  → Fine-grained exceptions to perimeter
  → Allow specific identities from outside
  → Allow specific data flows out
  → Logged and auditable

USE CASES:
  → Prevent copying BigQuery data to external project
  → Restrict Cloud Storage access to corporate network
  → Protect sensitive data from insider threats
  → Compliance (data residency)
```

---

## 5. Security Assessment

```
NETWORK SECURITY ASSESSMENT:

# List all VPC networks
gcloud compute networks list

# Describe a VPC
gcloud compute networks describe my-vpc

# List subnets
gcloud compute networks subnets list

# List all firewall rules
gcloud compute firewall-rules list

# Find overly permissive rules
gcloud compute firewall-rules list \
  --filter="sourceRanges:0.0.0.0/0 AND direction:INGRESS" \
  --format="table(name, allowed[].map().firewall_rule().list():label=RULES)"

# Check VPC Flow Logs status
gcloud compute networks subnets describe SUBNET \
  --region=REGION --format="get(logConfig)"

COMMON MISCONFIGURATIONS:
  → SSH/RDP open to 0.0.0.0/0
  → Default VPC with permissive rules
  → No VPC Flow Logs enabled
  → Missing egress restrictions
  → No network segmentation
  → VPC Service Controls not used for sensitive data
  → Firewall rules using tags (can be self-assigned)
  → No Cloud Armor on public-facing apps

SECURITY CHECKLIST:
  [ ] Custom mode VPC (not auto mode)
  [ ] No management ports open to internet
  [ ] VPC Flow Logs enabled on all subnets
  [ ] Cloud Armor on external load balancers
  [ ] VPC Service Controls for sensitive projects
  [ ] Private Google Access enabled
  [ ] Shared VPC for centralized management
  [ ] Firewall rules use service accounts (not tags)
  [ ] Regular firewall rule review
  [ ] Egress rules restrict outbound
```

---

## Summary Table

| Feature | Purpose | Scope |
|---------|---------|-------|
| VPC | Network isolation | Global |
| Firewall Rules | Traffic filtering | VPC-level |
| Cloud Armor | DDoS/WAF | Load Balancer |
| VPC Service Controls | API perimeter | Organization |
| Shared VPC | Centralized networking | Cross-project |
| VPC Peering | VPC interconnection | VPC-to-VPC |

---

## Revision Questions

1. How do GCP VPCs differ from AWS VPCs in terms of scope?
2. What are the default firewall rules and why are they risky?
3. How does Cloud Armor provide Layer 7 protection?
4. What problem do VPC Service Controls solve?
5. What firewall misconfigurations should be checked during assessment?

---

*Previous: [02-iam-fundamentals.md](02-iam-fundamentals.md) | Next: [04-cloud-audit-logs.md](04-cloud-audit-logs.md)*

---

*[Back to README](../README.md)*
