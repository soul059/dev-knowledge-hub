# Unit 6: Cloud Network Security — Topic 4: Private Connectivity

## Overview

**Private connectivity** in cloud environments ensures that communication between services, between cloud and on-premises, and between cloud providers occurs over private networks without traversing the public internet. Private connectivity reduces exposure to network-based attacks, improves latency, and is required for many compliance frameworks.

---

## 1. Private Access to Cloud Services

```
PRIVATE ENDPOINTS / SERVICE ENDPOINTS:

THE PROBLEM:
  Application → Internet → Cloud PaaS Service
  (traffic traverses public internet — risky!)

THE SOLUTION:
  Application → Private Endpoint → Cloud PaaS Service
  (traffic stays on cloud backbone — secure!)

AWS VPC ENDPOINTS:

  Gateway Endpoints:
  → S3 and DynamoDB only
  → Free
  → Route table entry
  → Regional
  
  Interface Endpoints (PrivateLink):
  → Most AWS services
  → Creates ENI in your VPC
  → Private IP address
  → DNS resolution to private IP
  → Cost: ~$7.50/month + data processing
  
  # Create S3 Gateway Endpoint
  aws ec2 create-vpc-endpoint \
    --vpc-id vpc-123 \
    --service-name com.amazonaws.us-east-1.s3 \
    --route-table-ids rtb-123

  # Create Interface Endpoint (SQS)
  aws ec2 create-vpc-endpoint \
    --vpc-id vpc-123 \
    --vpc-endpoint-type Interface \
    --service-name com.amazonaws.us-east-1.sqs \
    --subnet-ids subnet-123 \
    --security-group-ids sg-123

  Endpoint Policies:
  → Restrict which resources can be accessed
  → Prevent data exfiltration
  {
    "Statement": [{
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ]
    }]
  }

AZURE PRIVATE ENDPOINTS:

  → Creates private IP in your VNet
  → Supported for Storage, SQL, Key Vault, etc.
  → DNS integration required
  → Approval workflow available
  → Network policies (NSG, UDR) supported
  
  # Create private endpoint
  az network private-endpoint create \
    --name storage-pe \
    --resource-group myRG \
    --vnet-name myVNet \
    --subnet private-subnet \
    --private-connection-resource-id /subscriptions/.../Microsoft.Storage/storageAccounts/mystorage \
    --group-id blob \
    --connection-name storage-connection

  # Create private DNS zone
  az network private-dns zone create \
    -g myRG -n privatelink.blob.core.windows.net
  
  Azure Service Endpoints (older):
  → Route traffic over Azure backbone
  → No private IP (uses service public IP)
  → Service endpoint policies
  → Simpler, less isolation than Private Endpoints

GCP PRIVATE GOOGLE ACCESS:

  → Access Google APIs without external IP
  → Enabled per subnet
  → Private.googleapis.com domain
  → Restricted.googleapis.com (VPC-SC enforced)
  
  # Enable Private Google Access
  gcloud compute networks subnets update my-subnet \
    --region=us-central1 \
    --enable-private-google-access

  Private Service Connect:
  → Consumer-side endpoints
  → Private IP for Google services
  → Service attachments for own services
  → Network-level isolation
```

---

## 2. Hybrid Connectivity

```
VPN CONNECTIONS:

AWS SITE-TO-SITE VPN:
  → IPsec tunnels (2 per connection)
  → Virtual Private Gateway or Transit Gateway
  → Up to 1.25 Gbps per tunnel
  → Customer Gateway on-premises
  → BGP or static routing

AZURE VPN GATEWAY:
  → IPsec/IKEv2
  → Site-to-Site, Point-to-Site, VNet-to-VNet
  → BGP support
  → Active-Active for HA
  → Multiple SKUs (Basic to VpnGw5)

GCP CLOUD VPN:
  → HA VPN (recommended) or Classic VPN
  → IPsec tunnels
  → 99.99% SLA (HA VPN)
  → BGP routing
  → Up to 3 Gbps per tunnel

DEDICATED CONNECTIONS:

  AWS DIRECT CONNECT:
  → Dedicated fiber (1/10/100 Gbps)
  → Hosted connection (50 Mbps - 10 Gbps)
  → Private VIF (VPC access)
  → Public VIF (AWS public services)
  → Transit VIF (Transit Gateway)
  → Consistent latency
  → No internet traversal

  AZURE EXPRESSROUTE:
  → Private peering (VNet access)
  → Microsoft peering (M365, Dynamics)
  → 50 Mbps to 100 Gbps
  → Global Reach (connect circuits)
  → FastPath for performance
  → Metered or unlimited

  GCP CLOUD INTERCONNECT:
  → Dedicated Interconnect (10/100 Gbps)
  → Partner Interconnect (50 Mbps - 50 Gbps)
  → Private access to GCP
  → 99.99% SLA (with redundancy)

SECURITY CONSIDERATIONS:
  → Encrypt traffic even on dedicated connections
  → IPsec over Direct Connect/ExpressRoute
  → MACsec for L2 encryption (where available)
  → Monitor connection health
  → Redundant connections for availability
  → Restrict BGP route announcements
  → Network segmentation extends to hybrid
```

---

## 3. Cross-Cloud Connectivity

```
MULTI-CLOUD NETWORKING:

VPN BETWEEN CLOUDS:
  AWS ←→ Azure:
  → AWS VPN Gateway + Azure VPN Gateway
  → IPsec tunnel between them
  → BGP for dynamic routing
  
  AWS ←→ GCP:
  → AWS VPN + GCP Cloud VPN
  → HA VPN for reliability
  
  Azure ←→ GCP:
  → Azure VPN + GCP Cloud VPN

THIRD-PARTY SOLUTIONS:
  → Aviatrix (multi-cloud networking)
  → Alkira (cloud networking as-a-service)
  → Megaport (interconnection platform)
  → Equinix Cloud Exchange
  
SECURITY:
  → Encrypt all cross-cloud traffic
  → Consistent firewall policies across clouds
  → Centralized monitoring
  → Identity federation across clouds
  → Unified logging and alerting
```

---

## 4. DNS Security for Private Access

```
PRIVATE DNS:

WHY IT MATTERS:
  → Private endpoints need DNS resolution
  → Must resolve to private IPs (not public)
  → Split-horizon DNS (different answers internally)
  → Prevent DNS-based data exfiltration

AWS:
  → Route 53 Resolver
  → Private Hosted Zones
  → Resolver Rules (forwarding)
  → DNS Firewall (block malicious domains)
  
  # Create private hosted zone
  aws route53 create-hosted-zone \
    --name internal.example.com \
    --vpc VPCRegion=us-east-1,VPCId=vpc-123 \
    --caller-reference $(date +%s)

AZURE:
  → Private DNS Zones
  → VNet links
  → Auto-registration
  → Private DNS Resolver
  
  # Link DNS zone to VNet
  az network private-dns link vnet create \
    -g myRG -n myLink \
    -z privatelink.blob.core.windows.net \
    -v myVNet -e true

GCP:
  → Cloud DNS private zones
  → DNS policies
  → DNS peering
  → Response policies (DNS firewall)
  
  # Create private DNS zone
  gcloud dns managed-zones create internal-zone \
    --dns-name=internal.example.com \
    --visibility=private \
    --networks=my-vpc

DNS SECURITY BEST PRACTICES:
  [ ] Private DNS zones for all private endpoints
  [ ] DNS logging enabled
  [ ] DNS firewall to block malicious domains
  [ ] Split-horizon DNS for hybrid environments
  [ ] DNSSEC where supported
  [ ] Monitor for DNS tunneling
  [ ] Restrict DNS resolution to authorized resolvers
```

---

## 5. Assessment

```
PRIVATE CONNECTIVITY ASSESSMENT:

CHECKLIST:
  [ ] PaaS services accessed via private endpoints
  [ ] No unnecessary public endpoints
  [ ] Hybrid connections encrypted
  [ ] Dedicated connections for sensitive workloads
  [ ] DNS properly configured for private access
  [ ] Endpoint policies restrict resource access
  [ ] Cross-cloud traffic encrypted
  [ ] Connection health monitored
  [ ] Redundant connectivity for critical paths
  [ ] Private connectivity documented

COMMON FINDINGS:
  Finding                            | Risk
  PaaS accessed via public endpoint  | High
  VPN without encryption             | Critical
  No endpoint policies               | Medium
  DNS resolves to public IPs         | Medium
  Single point of failure connection | High
  No monitoring on VPN tunnels       | Medium

TESTING:
  → Verify private endpoint resolves to private IP
    nslookup mystorageaccount.blob.core.windows.net
    → Should return 10.x.x.x (not public IP)
  
  → Test connectivity through private endpoint
    curl https://mystorageaccount.blob.core.windows.net
  
  → Verify public access is blocked
    → From external: Should fail
    → From VNet: Should succeed via private IP
  
  → Check VPN tunnel status
    → Both tunnels active
    → BGP routes propagated
    → Latency within SLA
```

---

## Summary Table

| Service | AWS | Azure | GCP |
|---------|-----|-------|-----|
| Private Endpoint | PrivateLink | Private Endpoint | Private Service Connect |
| VPN | Site-to-Site VPN | VPN Gateway | Cloud VPN |
| Dedicated | Direct Connect | ExpressRoute | Cloud Interconnect |
| Private DNS | Route 53 Private | Private DNS Zone | Cloud DNS Private |
| DNS Firewall | Route 53 DNS FW | DNS Private Resolver | Response Policies |

---

## Revision Questions

1. Why is private connectivity important for PaaS services?
2. How do VPC endpoints prevent data exfiltration?
3. What are the security considerations for dedicated connections?
4. Why is DNS configuration critical for private endpoints?
5. How should private connectivity be tested during a security assessment?

---

*Previous: [03-firewall-rules.md](03-firewall-rules.md) | Next: [05-ddos-protection.md](05-ddos-protection.md)*

---

*[Back to README](../README.md)*
