# Unit 2: AWS Security — Topic 4: VPC Security (Security Groups, NACLs)

## Overview

**Amazon Virtual Private Cloud (VPC)** provides network isolation for AWS resources. VPC security is implemented through Security Groups (instance-level firewalls) and Network Access Control Lists (NACLs — subnet-level firewalls). Proper VPC configuration is essential for defense in depth, network segmentation, and preventing unauthorized access to cloud resources.

---

## 1. VPC Security Architecture

```
VPC COMPONENTS:

  ┌───────────────────────────────────────────────┐
  │                     VPC                        │
  │               (10.0.0.0/16)                    │
  │                                                │
  │  ┌──────────────────────┐                      │
  │  │  Internet Gateway    │ ← Public traffic     │
  │  └──────────┬───────────┘                      │
  │             │                                  │
  │  ┌──────────▼───────────┐  ← NACL (subnet)    │
  │  │  Public Subnet       │                      │
  │  │  10.0.1.0/24         │                      │
  │  │  ┌─────────────────┐ │                      │
  │  │  │ EC2 Instance    │ │ ← Security Group     │
  │  │  │ (Web Server)    │ │   (instance)         │
  │  │  └─────────────────┘ │                      │
  │  └──────────┬───────────┘                      │
  │             │                                  │
  │  ┌──────────▼───────────┐  ← NACL (subnet)    │
  │  │  Private Subnet      │                      │
  │  │  10.0.2.0/24         │                      │
  │  │  ┌─────────────────┐ │                      │
  │  │  │ EC2 Instance    │ │ ← Security Group     │
  │  │  │ (App Server)    │ │   (instance)         │
  │  │  └─────────────────┘ │                      │
  │  └──────────┬───────────┘                      │
  │             │                                  │
  │  ┌──────────▼───────────┐  ← NACL (subnet)    │
  │  │  Data Subnet         │                      │
  │  │  10.0.3.0/24         │                      │
  │  │  ┌─────────────────┐ │                      │
  │  │  │ RDS Instance    │ │ ← Security Group     │
  │  │  │ (Database)      │ │   (instance)         │
  │  │  └─────────────────┘ │                      │
  │  └──────────────────────┘                      │
  │                                                │
  │  Route Tables | NAT Gateway | VPC Endpoints    │
  └────────────────────────────────────────────────┘
```

---

## 2. Security Groups

```
SECURITY GROUPS — INSTANCE FIREWALL:

CHARACTERISTICS:
  → Stateful (return traffic automatically allowed)
  → Allow rules only (no deny rules)
  → Applied at instance/ENI level
  → Up to 5 SGs per instance
  → Default: deny all inbound, allow all outbound
  → Changes take effect immediately

INBOUND RULES:
  Type          | Protocol | Port | Source
  SSH           | TCP      | 22   | 10.0.0.0/8     ✓ Good
  SSH           | TCP      | 22   | 0.0.0.0/0      ✗ BAD!
  HTTP          | TCP      | 80   | 0.0.0.0/0      ✓ Public web
  HTTPS         | TCP      | 443  | 0.0.0.0/0      ✓ Public web
  RDP           | TCP      | 3389 | 10.0.1.0/24    ✓ Internal only
  MySQL         | TCP      | 3306 | sg-app-servers  ✓ SG reference
  Custom        | TCP      | 8080 | pl-xxxxxxxx    ✓ Prefix list

OUTBOUND RULES:
  Type          | Protocol | Port | Destination
  All traffic   | All      | All  | 0.0.0.0/0      Default
  HTTPS         | TCP      | 443  | 0.0.0.0/0      Restricted
  DNS           | UDP      | 53   | 0.0.0.0/0      DNS resolution

SECURITY GROUP REFERENCES:
  → Reference other security groups as source/destination
  → Powerful for dynamic environments
  → Example: App SG allows DB port FROM App SG
  → Any instance in App SG can reach DB
  → Works across subnets within VPC
  
  Web SG → allows HTTP/HTTPS from internet
  App SG → allows port 8080 from Web SG
  DB SG  → allows port 3306 from App SG only

ANTI-PATTERNS (Avoid):
  ✗ 0.0.0.0/0 on SSH (port 22)
  ✗ 0.0.0.0/0 on RDP (port 3389)
  ✗ 0.0.0.0/0 on database ports
  ✗ All traffic allowed inbound
  ✗ Overly broad CIDR ranges
  ✗ Unused security groups with wide rules

CLI COMMANDS:
  # List security groups
  aws ec2 describe-security-groups
  
  # Find open SSH
  aws ec2 describe-security-groups \
    --filters "Name=ip-permission.from-port,Values=22" \
              "Name=ip-permission.cidr,Values=0.0.0.0/0"
  
  # Create security group
  aws ec2 create-security-group \
    --group-name WebSG \
    --description "Web server SG" \
    --vpc-id vpc-123456
  
  # Add inbound rule
  aws ec2 authorize-security-group-ingress \
    --group-id sg-123456 \
    --protocol tcp \
    --port 443 \
    --cidr 0.0.0.0/0
```

---

## 3. Network ACLs (NACLs)

```
NACLs — SUBNET FIREWALL:

CHARACTERISTICS:
  → Stateless (must define inbound AND outbound)
  → Allow AND deny rules
  → Applied at subnet level
  → Rules processed in order (lowest number first)
  → Default NACL: allows all traffic
  → Custom NACL: denies all by default

RULE EVALUATION:
  Rules evaluated in numerical order:
  Rule 100: Allow TCP 443 from 0.0.0.0/0   → Checked first
  Rule 200: Allow TCP 80 from 0.0.0.0/0    → Checked second
  Rule 300: Deny TCP 22 from 0.0.0.0/0     → Checked third
  Rule *:   Deny all                         → Default deny

EXAMPLE NACL — WEB SUBNET:

  INBOUND:
  Rule | Type   | Protocol | Port      | Source      | Action
  100  | HTTPS  | TCP      | 443       | 0.0.0.0/0   | ALLOW
  110  | HTTP   | TCP      | 80        | 0.0.0.0/0   | ALLOW
  120  | SSH    | TCP      | 22        | 10.0.0.0/8  | ALLOW
  130  | Custom | TCP      | 1024-65535| 0.0.0.0/0   | ALLOW (return)
  *    | All    | All      | All       | 0.0.0.0/0   | DENY

  OUTBOUND:
  Rule | Type   | Protocol | Port      | Destination | Action
  100  | HTTPS  | TCP      | 443       | 0.0.0.0/0   | ALLOW
  110  | HTTP   | TCP      | 80        | 0.0.0.0/0   | ALLOW
  120  | Custom | TCP      | 1024-65535| 0.0.0.0/0   | ALLOW (return)
  130  | MySQL  | TCP      | 3306      | 10.0.3.0/24 | ALLOW
  *    | All    | All      | All       | 0.0.0.0/0   | DENY

IMPORTANT: EPHEMERAL PORTS
  → Return traffic uses ephemeral ports (1024-65535)
  → Must allow these in NACLs for responses to work
  → This is because NACLs are stateless
  → Security groups handle this automatically (stateful)

SECURITY GROUPS vs NACLs:
  Feature        | Security Group  | NACL
  Level          | Instance/ENI    | Subnet
  State          | Stateful        | Stateless
  Rules          | Allow only      | Allow + Deny
  Rule Order     | All evaluated   | Numbered order
  Default        | Deny all in     | Allow all (default)
  Applied to     | Specific ENIs   | All in subnet
  Return traffic | Automatic       | Must allow ephemeral
```

---

## 4. VPC Flow Logs

```
FLOW LOGS — NETWORK MONITORING:

WHAT THEY CAPTURE:
  → Source and destination IP
  → Source and destination port
  → Protocol
  → Packets and bytes
  → Start and end time
  → Action (ACCEPT or REJECT)
  → NOT packet contents (just metadata)

LOG FORMAT:
  version account-id interface-id srcaddr dstaddr srcport dstport protocol packets bytes start end action log-status
  
  Example:
  2 123456789012 eni-abc123 10.0.1.50 10.0.2.100 49152 3306 6 10 840 1616729292 1616729352 ACCEPT OK
  → Source: 10.0.1.50:49152
  → Dest: 10.0.2.100:3306 (MySQL)
  → Protocol: 6 (TCP)
  → Action: ACCEPT

LEVELS:
  → VPC Flow Logs: All traffic in VPC
  → Subnet Flow Logs: Traffic in specific subnet
  → ENI Flow Logs: Traffic for specific network interface

DESTINATION:
  → CloudWatch Logs
  → S3 bucket
  → Kinesis Data Firehose

ENABLE FLOW LOGS:
  aws ec2 create-flow-logs \
    --resource-type VPC \
    --resource-ids vpc-123456 \
    --traffic-type ALL \
    --log-destination-type s3 \
    --log-destination arn:aws:s3:::flow-log-bucket

SECURITY USE CASES:
  → Detect port scanning
  → Identify unauthorized connections
  → Monitor rejected traffic
  → Compliance evidence
  → Incident investigation
  → Network baseline analysis
```

---

## 5. Advanced VPC Security

```
VPC ENDPOINTS:
  → Private connectivity to AWS services
  → Traffic doesn't traverse internet
  → Interface endpoints (ENI-based)
  → Gateway endpoints (S3, DynamoDB)
  → Endpoint policies restrict access
  
  # Create S3 gateway endpoint
  aws ec2 create-vpc-endpoint \
    --vpc-id vpc-123456 \
    --service-name com.amazonaws.us-east-1.s3 \
    --route-table-ids rtb-123456

VPC PEERING:
  → Connect two VPCs privately
  → No transitive peering
  → Cross-account and cross-region
  → Security group references across peers

TRANSIT GATEWAY:
  → Hub-and-spoke network
  → Connect multiple VPCs and on-premises
  → Centralized network management
  → Route table-based traffic control

AWS NETWORK FIREWALL:
  → Managed stateful firewall
  → IPS/IDS capabilities
  → Domain filtering
  → Suricata-compatible rules
  → Deep packet inspection

SECURITY BEST PRACTICES:
  [ ] Use private subnets for non-public resources
  [ ] Security groups with least privilege
  [ ] NACLs as additional defense layer
  [ ] VPC Flow Logs enabled (all traffic)
  [ ] VPC endpoints for AWS service access
  [ ] No public IPs on internal resources
  [ ] NAT Gateway for outbound-only internet
  [ ] Network segmentation per tier
  [ ] Regular security group audit
  [ ] DNS logging enabled
```

---

## Summary Table

| Control | Level | Type | Key Feature |
|---------|-------|------|-------------|
| Security Groups | Instance | Stateful, Allow only | SG references |
| NACLs | Subnet | Stateless, Allow/Deny | Rule ordering |
| Flow Logs | VPC/Subnet/ENI | Monitoring | Traffic metadata |
| VPC Endpoints | Service access | Private path | No internet needed |
| Network Firewall | VPC | IPS/IDS | Deep inspection |

---

## Revision Questions

1. What are the key differences between Security Groups and NACLs?
2. Why are Security Group references considered best practice?
3. What is the significance of ephemeral ports in NACL configuration?
4. How do VPC Flow Logs support security monitoring?
5. What is a VPC Endpoint and why is it important for security?

---

*Previous: [03-s3-security.md](03-s3-security.md) | Next: [05-cloudtrail-and-cloudwatch.md](05-cloudtrail-and-cloudwatch.md)*

---

*[Back to README](../README.md)*
