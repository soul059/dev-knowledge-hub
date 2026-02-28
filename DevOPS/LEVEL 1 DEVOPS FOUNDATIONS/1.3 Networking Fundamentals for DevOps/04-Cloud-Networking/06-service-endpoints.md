# Chapter 6: VPC Endpoints and Service Endpoints

## Overview

**VPC Endpoints** allow you to privately connect your VPC to supported AWS services without traversing the public internet. This eliminates the need for a NAT Gateway, internet gateway, or VPN connection for accessing services like S3, DynamoDB, or ECR. VPC Endpoints improve security, reduce costs, and lower latency.

---

## 6.1 Why VPC Endpoints?

```
┌──────────────────────────────────────────────────────────────┐
│       WITHOUT VPC ENDPOINT (via Internet)                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────┐     ┌─────────┐     ┌──────────┐     ┌─────┐  │
│  │ EC2      │ ──▶ │ NAT     │ ──▶ │ Internet │ ──▶ │ S3  │  │
│  │ (Private │     │ Gateway │     │ Gateway  │     │     │  │
│  │  Subnet) │     │ ($$$)   │     │          │     │     │  │
│  └──────────┘     └─────────┘     └──────────┘     └─────┘  │
│                                                              │
│  Problems: ✗ NAT Gateway costs  ✗ Data leaves VPC           │
│            ✗ Higher latency     ✗ Internet dependency        │
├──────────────────────────────────────────────────────────────┤
│       WITH VPC ENDPOINT (Private)                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────┐     ┌──────────────┐     ┌─────┐              │
│  │ EC2      │ ──▶ │ VPC Endpoint │ ──▶ │ S3  │              │
│  │ (Private │     │ (Free/Low $) │     │     │              │
│  │  Subnet) │     │              │     │     │              │
│  └──────────┘     └──────────────┘     └─────┘              │
│                                                              │
│  Benefits: ✓ No NAT cost  ✓ Traffic stays in AWS network    │
│            ✓ Lower latency ✓ More secure                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 6.2 Types of VPC Endpoints

```
┌──────────────────────────────────────────────────────────────┐
│           VPC ENDPOINT TYPES                                 │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. GATEWAY ENDPOINT                                         │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  - For: S3 and DynamoDB ONLY                           │  │
│  │  - How: Route table entry pointing to endpoint         │  │
│  │  - Cost: FREE                                          │  │
│  │  - Creates: Prefix list in route table                 │  │
│  │                                                        │  │
│  │  Route Table:                                          │  │
│  │  pl-xxxxx (S3 prefix list) → vpce-gateway              │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  2. INTERFACE ENDPOINT (PrivateLink)                         │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  - For: Most other AWS services (ECR, SQS, SSM, etc.) │  │
│  │  - How: ENI placed in your subnet with private IP      │  │
│  │  - Cost: ~$0.01/hr + data processing                   │  │
│  │  - DNS: Creates private DNS name                       │  │
│  │                                                        │  │
│  │  Subnet:                                               │  │
│  │  ┌──────────────────────┐                              │  │
│  │  │ ENI: 10.0.1.200      │ ← Endpoint Network Interface │  │
│  │  │ DNS: ecr.us-east-1   │                              │  │
│  │  │      .amazonaws.com  │                              │  │
│  │  └──────────────────────┘                              │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  3. GATEWAY LOAD BALANCER ENDPOINT                           │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  - For: Third-party virtual appliances                 │  │
│  │  - How: Intercepts traffic for inspection              │  │
│  │  - Use case: Firewalls, IDS/IPS                        │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 6.3 Gateway Endpoint vs Interface Endpoint

| Feature | Gateway Endpoint | Interface Endpoint |
|---------|-----------------|-------------------|
| Services | S3, DynamoDB | 100+ AWS services |
| Cost | Free | ~$0.01/hr per AZ |
| Implementation | Route table entry | ENI in subnet |
| Security | Endpoint policy | SG + Endpoint policy |
| DNS | No change | Private DNS override |
| Cross-region | No | No |
| On-premise access | Via route propagation | Via DNS + Direct Connect/VPN |

---

## 6.4 Terraform: Gateway Endpoint (S3)

```hcl
# Gateway Endpoint for S3 (FREE)
resource "aws_vpc_endpoint" "s3" {
  vpc_id       = aws_vpc.main.id
  service_name = "com.amazonaws.us-east-1.s3"

  # Associate with private route tables
  route_table_ids = [
    aws_route_table.private[0].id,
    aws_route_table.private[1].id,
  ]

  # Endpoint policy: restrict to specific bucket
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid       = "AllowSpecificBucket"
        Effect    = "Allow"
        Principal = "*"
        Action    = ["s3:GetObject", "s3:PutObject"]
        Resource  = [
          "arn:aws:s3:::my-app-bucket",
          "arn:aws:s3:::my-app-bucket/*"
        ]
      }
    ]
  })

  tags = { Name = "s3-gateway-endpoint" }
}
```

---

## 6.5 Terraform: Interface Endpoint (ECR, SSM)

```hcl
# Security Group for Interface Endpoints
resource "aws_security_group" "vpc_endpoints" {
  name        = "vpc-endpoints-sg"
  description = "Allow HTTPS from VPC"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = [aws_vpc.main.cidr_block]
  }

  tags = { Name = "vpc-endpoints-sg" }
}

# Interface Endpoint for ECR (Docker image pulls)
resource "aws_vpc_endpoint" "ecr_api" {
  vpc_id              = aws_vpc.main.id
  service_name        = "com.amazonaws.us-east-1.ecr.api"
  vpc_endpoint_type   = "Interface"
  private_dns_enabled = true

  subnet_ids         = aws_subnet.private[*].id
  security_group_ids = [aws_security_group.vpc_endpoints.id]

  tags = { Name = "ecr-api-endpoint" }
}

resource "aws_vpc_endpoint" "ecr_dkr" {
  vpc_id              = aws_vpc.main.id
  service_name        = "com.amazonaws.us-east-1.ecr.dkr"
  vpc_endpoint_type   = "Interface"
  private_dns_enabled = true

  subnet_ids         = aws_subnet.private[*].id
  security_group_ids = [aws_security_group.vpc_endpoints.id]

  tags = { Name = "ecr-dkr-endpoint" }
}

# Interface Endpoint for SSM (Session Manager — no SSH needed)
resource "aws_vpc_endpoint" "ssm" {
  vpc_id              = aws_vpc.main.id
  service_name        = "com.amazonaws.us-east-1.ssm"
  vpc_endpoint_type   = "Interface"
  private_dns_enabled = true

  subnet_ids         = aws_subnet.private[*].id
  security_group_ids = [aws_security_group.vpc_endpoints.id]

  tags = { Name = "ssm-endpoint" }
}

resource "aws_vpc_endpoint" "ssm_messages" {
  vpc_id              = aws_vpc.main.id
  service_name        = "com.amazonaws.us-east-1.ssmmessages"
  vpc_endpoint_type   = "Interface"
  private_dns_enabled = true

  subnet_ids         = aws_subnet.private[*].id
  security_group_ids = [aws_security_group.vpc_endpoints.id]

  tags = { Name = "ssm-messages-endpoint" }
}
```

---

## 6.6 Common VPC Endpoints for DevOps

| Service | Endpoint Type | Why You Need It |
|---------|--------------|-----------------|
| S3 | Gateway | Artifact storage, logs, backups |
| DynamoDB | Gateway | Terraform state locking |
| ECR (api + dkr) | Interface | Pull Docker images privately |
| SSM / SSM Messages | Interface | Session Manager (replaces SSH) |
| CloudWatch Logs | Interface | Ship logs without internet |
| STS | Interface | Assume IAM roles |
| Secrets Manager | Interface | Fetch secrets at runtime |
| SQS | Interface | Message queue access |
| KMS | Interface | Encryption key operations |
| EC2 Messages | Interface | SSM Agent communication |

---

## 6.7 AWS PrivateLink (Sharing Services)

```
┌──────────────────────────────────────────────────────────────┐
│             AWS PRIVATELINK                                  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Service Provider Account        Service Consumer Account    │
│  ┌──────────────────────┐        ┌──────────────────────┐    │
│  │  VPC: 10.1.0.0/16    │        │  VPC: 10.2.0.0/16    │    │
│  │                      │        │                      │    │
│  │  ┌────────────────┐  │        │  ┌────────────────┐  │    │
│  │  │ NLB            │  │ Private│  │ VPC Endpoint   │  │    │
│  │  │ (Network LB)   │◀═══Link═══▶│ (Interface)     │  │    │
│  │  └───────┬────────┘  │        │  └───────┬────────┘  │    │
│  │          │           │        │          │           │    │
│  │  ┌───────▼────────┐  │        │  ┌───────▼────────┐  │    │
│  │  │ Your Service   │  │        │  │ Consumer App   │  │    │
│  │  │ (API/App)      │  │        │  │                │  │    │
│  │  └────────────────┘  │        │  └────────────────┘  │    │
│  └──────────────────────┘        └──────────────────────┘    │
│                                                              │
│  Use Cases:                                                  │
│  - SaaS providers exposing services to customers             │
│  - Internal platform teams sharing microservices             │
│  - Cross-account service access without peering              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 6.8 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| S3 access not working via endpoint | Route table not associated | Associate RT with gateway endpoint |
| Interface endpoint DNS not resolving | Private DNS not enabled | Enable `private_dns_enabled` |
| ECR pull fails in private subnet | Missing ecr.api, ecr.dkr, S3 endpoints | Create all three endpoints |
| Endpoint security group blocking | SG doesn't allow HTTPS (443) | Allow 443 from VPC CIDR |
| SSM Session Manager not connecting | Missing ssm, ssmmessages, ec2messages | Create all three endpoints |
| High NAT costs for AWS API calls | Using NAT instead of endpoints | Create VPC endpoints for services |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| Gateway Endpoint | S3/DynamoDB only, free, route table-based |
| Interface Endpoint | Most services, ~$0.01/hr, ENI-based |
| PrivateLink | Share services across accounts privately |
| Endpoint Policy | Controls which resources are accessible |
| Private DNS | Interface endpoints override public DNS |
| ECR needs 3 endpoints | ecr.api + ecr.dkr + S3 gateway |
| SSM needs 3 endpoints | ssm + ssmmessages + ec2messages |
| Cost saving | Replace NAT Gateway with VPC endpoints |

---

## Quick Revision Questions

1. **What's the difference between a Gateway Endpoint and an Interface Endpoint?**
2. **Why is a Gateway Endpoint for S3 free while Interface Endpoints cost money?**
3. **What three VPC endpoints do you need for ECS/EKS to pull images from ECR in a private subnet?**
4. **How does AWS PrivateLink differ from VPC Peering?**
5. **Your EC2 instances in a private subnet need to send logs to CloudWatch without internet access. What do you set up?**
6. **How can VPC endpoints reduce your NAT Gateway costs?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← NACLs vs Security Groups](05-nacls-vs-security-groups.md) | [README](../README.md) | [Unit 5: DNS Hierarchy →](../05-DNS-Deep-Dive/01-dns-hierarchy.md) |
