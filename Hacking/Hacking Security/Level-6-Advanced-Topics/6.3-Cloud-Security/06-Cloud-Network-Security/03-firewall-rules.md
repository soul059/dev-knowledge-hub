# Unit 6: Cloud Network Security — Topic 3: Firewall Rules

## Overview

**Cloud firewall rules** control network traffic to and from cloud resources. Each cloud provider implements firewalls differently — from stateful security groups to centralized managed firewalls with Layer 7 inspection. Understanding how to configure, audit, and assess cloud firewall rules is essential for securing cloud workloads and conducting security assessments.

---

## 1. Cloud Firewall Comparison

```
FIREWALL TYPES ACROSS CLOUDS:

  ┌────────────────────────────────────────────────────┐
  │              CLOUD FIREWALLS                       │
  ├──────────┬──────────────┬─────────────┬────────────┤
  │          │    AWS        │   Azure      │   GCP     │
  ├──────────┼──────────────┼─────────────┼────────────┤
  │ Instance │ Security     │ NSG         │ VPC        │
  │ Level    │ Groups       │             │ Firewall   │
  ├──────────┼──────────────┼─────────────┼────────────┤
  │ Subnet   │ Network      │ NSG (subnet)│ N/A        │
  │ Level    │ ACLs         │             │ (global)   │
  ├──────────┼──────────────┼─────────────┼────────────┤
  │ Managed  │ AWS Network  │ Azure       │ Cloud      │
  │ Firewall │ Firewall     │ Firewall    │ Armor      │
  ├──────────┼──────────────┼─────────────┼────────────┤
  │ WAF      │ AWS WAF      │ Azure WAF   │ Cloud      │
  │          │              │             │ Armor WAF  │
  └──────────┴──────────────┴─────────────┴────────────┘

DETAILED COMPARISON:
  Feature         | AWS SG     | AWS NACL   | Azure NSG  | GCP FW
  Stateful        | Yes        | No         | Yes        | Yes
  Scope           | Instance   | Subnet     | Subnet/NIC | VPC
  Allow rules     | Yes        | Yes        | Yes        | Yes
  Deny rules      | No         | Yes        | Yes        | Yes
  Priority        | All eval   | Ordered    | Priority   | Priority
  Default         | Deny all in| Allow all  | Deny all in| Deny in
                  | Allow all out          | Allow out  | Allow out
```

---

## 2. AWS Firewall Configuration

```
AWS SECURITY GROUPS:

  # Create security group
  aws ec2 create-security-group \
    --group-name web-sg \
    --description "Web server SG" \
    --vpc-id vpc-123

  # Allow HTTPS from anywhere
  aws ec2 authorize-security-group-ingress \
    --group-id sg-123 \
    --protocol tcp \
    --port 443 \
    --cidr 0.0.0.0/0

  # Allow app tier from web SG
  aws ec2 authorize-security-group-ingress \
    --group-id sg-app \
    --protocol tcp \
    --port 8080 \
    --source-group sg-web

  # Describe rules
  aws ec2 describe-security-groups \
    --group-ids sg-123

AWS NETWORK FIREWALL:
  → Centralized, managed firewall
  → Stateful and stateless rules
  → Suricata-compatible IPS rules
  → Domain name filtering
  → TLS inspection
  → Deployed in dedicated subnets
  → Integrated with Firewall Manager

  # Managed by Firewall Manager for multi-account
  aws network-firewall describe-firewall \
    --firewall-name prod-firewall

AWS WAF:
  → Layer 7 protection
  → Web ACLs with rules
  → Managed rule groups (OWASP, bot control)
  → Rate limiting
  → IP reputation
  → Geographic blocking
  → Applied to ALB, CloudFront, API Gateway
```

---

## 3. Azure and GCP Firewall Configuration

```
AZURE NSG RULES:

  # Create NSG
  az network nsg create -g myRG -n web-nsg

  # Add allow HTTPS rule
  az network nsg rule create \
    -g myRG --nsg-name web-nsg \
    -n AllowHTTPS \
    --priority 100 \
    --direction Inbound \
    --access Allow \
    --protocol TCP \
    --destination-port-ranges 443

  # Add deny all
  az network nsg rule create \
    -g myRG --nsg-name web-nsg \
    -n DenyAll \
    --priority 4096 \
    --direction Inbound \
    --access Deny \
    --protocol '*' \
    --source-address-prefixes '*' \
    --destination-port-ranges '*'

AZURE FIREWALL:
  → Fully managed, centralized
  → Application rules (FQDN)
  → Network rules (IP/port)
  → NAT rules (DNAT)
  → Threat intelligence
  → TLS inspection (Premium)
  → IDPS (Premium)
  → Web categories
  → Force-tunnel all traffic through

GCP FIREWALL RULES:

  # Create allow rule
  gcloud compute firewall-rules create allow-https \
    --network=my-vpc \
    --direction=INGRESS \
    --action=ALLOW \
    --rules=tcp:443 \
    --source-ranges=0.0.0.0/0 \
    --target-tags=web-server \
    --priority=1000

  # Create deny rule
  gcloud compute firewall-rules create deny-all \
    --network=my-vpc \
    --direction=INGRESS \
    --action=DENY \
    --rules=all \
    --source-ranges=0.0.0.0/0 \
    --priority=65534

GCP HIERARCHICAL FIREWALL POLICIES:
  → Organization or folder level
  → Override project-level rules
  → Centralized security governance
  → "goto_next" for delegation
```

---

## 4. Firewall Rule Auditing

```
SECURITY ASSESSMENT:

COMMON MISCONFIGURATIONS:
  → SSH (22) open to 0.0.0.0/0
  → RDP (3389) open to 0.0.0.0/0
  → All ports open (0-65535)
  → Database ports open to internet
  → ICMP from any source
  → Overly broad CIDR ranges
  → Egress not restricted
  → Unused rules not cleaned up

AWS AUDIT:
  # Find SGs with SSH open to world
  aws ec2 describe-security-groups \
    --filters "Name=ip-permission.from-port,Values=22" \
              "Name=ip-permission.cidr,Values=0.0.0.0/0" \
    --query "SecurityGroups[*].{ID:GroupId,Name:GroupName}"

  # Find SGs allowing all traffic
  aws ec2 describe-security-groups \
    --filters "Name=ip-permission.protocol,Values=-1" \
    --query "SecurityGroups[*].{ID:GroupId,Name:GroupName}"

AZURE AUDIT:
  # Find NSGs with SSH open
  az network nsg list --query "[].{Name:name, Rules:securityRules[?destinationPortRange=='22' && sourceAddressPrefix=='*' && access=='Allow']}" -o json

GCP AUDIT:
  # Find overly permissive rules
  gcloud compute firewall-rules list \
    --filter="sourceRanges:0.0.0.0/0 AND direction:INGRESS" \
    --format="table(name,allowed[].map().firewall_rule().list())"

AUTOMATED TOOLS:
  → Prowler (AWS)
  → ScoutSuite (multi-cloud)
  → Cloud Custodian (policy as code)
  → Prisma Cloud
  → Steampipe (SQL queries on cloud)
  → tfsec (Terraform security)

RULE REVIEW PROCESS:
  1. Export all firewall rules
  2. Identify rules with broad source (0.0.0.0/0, *)
  3. Check for sensitive ports (22, 3389, 3306, 5432)
  4. Verify each rule has justification
  5. Remove unused rules
  6. Tighten overly permissive rules
  7. Document approved exceptions
  8. Schedule regular reviews (monthly)
```

---

## 5. Best Practices

```
FIREWALL BEST PRACTICES:

DESIGN:
  [ ] Default deny (explicit allow)
  [ ] Least privilege per rule
  [ ] Use security group references (not IPs)
  [ ] Use service tags where available
  [ ] Separate rules per tier/function
  [ ] Consistent naming convention
  [ ] Description on every rule

MANAGEMENT:
  [ ] Infrastructure as code (Terraform)
  [ ] Change management process
  [ ] Peer review for rule changes
  [ ] Automated compliance checking
  [ ] Regular rule cleanup
  [ ] Document all exceptions

MONITORING:
  [ ] Flow logs enabled
  [ ] Alert on rule changes
  [ ] Track denied traffic patterns
  [ ] Baseline normal traffic
  [ ] Detect anomalous flows

NAMING CONVENTION:
  Format: [Direction]-[Source]-[Destination]-[Port]-[Action]
  Examples:
  → ingress-internet-web-443-allow
  → ingress-web-app-8080-allow
  → ingress-app-db-3306-allow
  → egress-app-internet-443-allow
  → ingress-any-mgmt-22-deny
```

---

## Summary Table

| Feature | AWS SG | AWS NACL | Azure NSG | GCP FW |
|---------|--------|----------|-----------|--------|
| State | Stateful | Stateless | Stateful | Stateful |
| Scope | Instance | Subnet | Subnet/NIC | VPC |
| Deny rules | No | Yes | Yes | Yes |
| Layer | 3/4 | 3/4 | 3/4 | 3/4 |

---

## Revision Questions

1. What is the difference between stateful and stateless firewalls?
2. How do AWS Security Groups differ from Azure NSGs?
3. What common firewall misconfigurations should be audited?
4. How can firewall rules be managed as infrastructure as code?
5. What tools can automate cloud firewall auditing?

---

*Previous: [02-network-segmentation.md](02-network-segmentation.md) | Next: [04-private-connectivity.md](04-private-connectivity.md)*

---

*[Back to README](../README.md)*
