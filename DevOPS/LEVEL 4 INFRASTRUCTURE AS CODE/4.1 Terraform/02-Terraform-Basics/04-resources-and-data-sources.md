# Chapter 4: Resources and Data Sources

[← Previous: HCL Syntax Basics](03-hcl-syntax-basics.md) | [Back to README](../README.md) | [Next: Terraform Workflow →](05-terraform-workflow.md)

---

## Overview

**Resources** are the most important element in Terraform — they represent the infrastructure objects you create and manage. **Data sources** let you read information about existing infrastructure that Terraform doesn't manage. Understanding both is essential for writing effective configurations.

---

## 4.1 Resources — Creating Infrastructure

```
┌──────────────────────────────────────────────────────────────┐
│                    RESOURCE ANATOMY                          │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│   resource "aws_instance" "web_server" {                     │
│      ▲          ▲              ▲                              │
│      │          │              └── Local name (your label)   │
│      │          └── Resource type (provider_resource)         │
│      └── Block keyword                                       │
│                                                               │
│     ami           = "ami-0c55b159cbfafe1f0"  ← Arguments    │
│     instance_type = "t3.micro"                               │
│                                                               │
│     tags = {                                ← Nested block   │
│       Name = "web-server"                                    │
│     }                                                         │
│   }                                                           │
│                                                               │
│   Reference: aws_instance.web_server.public_ip               │
│              ─────────── ────────── ─────────                │
│              type        name       attribute                 │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### Resource Lifecycle

```
  ┌────────────┐    ┌────────────┐    ┌────────────┐    ┌────────────┐
  │  CREATE    │───▶│   READ     │───▶│  UPDATE    │───▶│  DELETE    │
  │ (apply)    │    │ (refresh)  │    │ (apply)    │    │ (destroy)  │
  └────────────┘    └────────────┘    └────────────┘    └────────────┘
       │                 │                 │                 │
       ▼                 ▼                 ▼                 ▼
   API: Create       API: Describe     API: Modify      API: Delete
   State: Add ID     State: Verify     State: Update    State: Remove
```

### Common AWS Resources

```hcl
# EC2 Instance
resource "aws_instance" "web" {
  ami                    = "ami-0c55b159cbfafe1f0"
  instance_type          = "t3.micro"
  subnet_id              = aws_subnet.public.id
  vpc_security_group_ids = [aws_security_group.web.id]
  key_name               = aws_key_pair.deployer.key_name

  root_block_device {
    volume_size = 20
    volume_type = "gp3"
  }

  tags = {
    Name = "web-server"
  }
}

# S3 Bucket
resource "aws_s3_bucket" "data" {
  bucket = "my-app-data-bucket-12345"

  tags = {
    Environment = "production"
  }
}

# Security Group
resource "aws_security_group" "web" {
  name        = "web-sg"
  description = "Allow HTTP and HTTPS"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# RDS Database
resource "aws_db_instance" "main" {
  identifier           = "app-database"
  engine               = "postgres"
  engine_version       = "15.4"
  instance_class       = "db.t3.micro"
  allocated_storage    = 20
  db_name              = "appdb"
  username             = var.db_username
  password             = var.db_password
  skip_final_snapshot  = true
}
```

---

## 4.2 Data Sources — Reading Existing Infrastructure

```
┌──────────────────────────────────────────────────────────────┐
│                    DATA SOURCE ANATOMY                       │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│   data "aws_ami" "ubuntu" {                                  │
│    ▲       ▲         ▲                                       │
│    │       │         └── Local name                          │
│    │       └── Data source type                              │
│    └── Block keyword (data, not resource)                    │
│                                                               │
│     most_recent = true              ← Filter / Query         │
│                                                               │
│     filter {                        ← Search criteria        │
│       name   = "name"                                        │
│       values = ["ubuntu/images/*"]                           │
│     }                                                         │
│   }                                                           │
│                                                               │
│   Reference: data.aws_ami.ubuntu.id                          │
│              ──── ─────── ────── ──                          │
│              data  type    name  attr                         │
│                                                               │
│   KEY DIFFERENCE:                                            │
│   resource = CREATE/MANAGE infrastructure                    │
│   data     = READ/QUERY existing infrastructure              │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### Common Data Sources

```hcl
# Look up the latest Ubuntu AMI
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]  # Canonical

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

# Use it in a resource
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id    # ← Dynamic AMI lookup
  instance_type = "t3.micro"
}

# Look up existing VPC
data "aws_vpc" "existing" {
  filter {
    name   = "tag:Name"
    values = ["production-vpc"]
  }
}

# Look up availability zones
data "aws_availability_zones" "available" {
  state = "available"
}

# Look up current AWS account info
data "aws_caller_identity" "current" {}

output "account_id" {
  value = data.aws_caller_identity.current.account_id
}

# Look up current region
data "aws_region" "current" {}

output "region" {
  value = data.aws_region.current.name
}

# Look up an IAM policy
data "aws_iam_policy" "admin" {
  name = "AdministratorAccess"
}
```

---

## 4.3 Resource vs Data Source

```
  ┌──────────────────────────────────────────────────────────┐
  │          RESOURCE vs DATA SOURCE COMPARISON              │
  ├──────────────────────────────────────────────────────────┤
  │                                                           │
  │   RESOURCE                      DATA SOURCE              │
  │   ┌──────────────────┐         ┌──────────────────┐     │
  │   │ resource "aws_   │         │ data "aws_       │     │
  │   │ instance" "web"  │         │ ami" "ubuntu"    │     │
  │   │ {                │         │ {                │     │
  │   │   ami = "..."    │         │   most_recent =  │     │
  │   │   type = "..."   │         │     true         │     │
  │   │ }                │         │ }                │     │
  │   └──────────────────┘         └──────────────────┘     │
  │                                                           │
  │   • CREATES resources          • READS existing data     │
  │   • Managed by Terraform       • Not managed by TF       │
  │   • In state file              • Queried each plan       │
  │   • CRUD operations            • Read-only               │
  │   • You control lifecycle      • External lifecycle      │
  │                                                           │
  │   Reference:                   Reference:                │
  │   aws_instance.web.id         data.aws_ami.ubuntu.id    │
  │                                                           │
  └──────────────────────────────────────────────────────────┘
```

---

## 4.4 Resource Dependencies

### Implicit Dependencies (Automatic)

```hcl
# Terraform detects dependencies from references

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id     # ← Implicit dependency on VPC
  cidr_block = "10.0.1.0/24"
}

resource "aws_instance" "web" {
  subnet_id = aws_subnet.public.id  # ← Implicit dependency on subnet
  ami       = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
}
```

```
  Dependency Graph (Auto-resolved):
  
  ┌─────────┐
  │   VPC   │  Created FIRST
  └────┬────┘
       │
       ▼
  ┌─────────┐
  │ Subnet  │  Created SECOND (needs VPC ID)
  └────┬────┘
       │
       ▼
  ┌─────────┐
  │   EC2   │  Created THIRD (needs Subnet ID)
  └─────────┘
```

### Explicit Dependencies (depends_on)

```hcl
# Use depends_on when Terraform can't detect the dependency

resource "aws_s3_bucket" "data" {
  bucket = "my-app-data"
}

resource "aws_instance" "app" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"

  # No reference to S3 bucket, but app needs it to exist
  depends_on = [aws_s3_bucket.data]
}

# Common use case: IAM policy must exist before resource that uses it
resource "aws_iam_role_policy" "app_policy" {
  name = "app-s3-access"
  role = aws_iam_role.app.id
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect   = "Allow"
      Action   = ["s3:GetObject"]
      Resource = ["${aws_s3_bucket.data.arn}/*"]
    }]
  })
}

resource "aws_instance" "app" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"

  depends_on = [aws_iam_role_policy.app_policy]
}
```

---

## 4.5 Resource Lifecycle Management

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"

  lifecycle {
    # Create the replacement BEFORE destroying the old one
    # Prevents downtime during updates
    create_before_destroy = true

    # Prevent accidental deletion (must be removed before destroy works)
    prevent_destroy = true

    # Ignore changes made outside Terraform
    # Useful when auto-scaling changes tags, etc.
    ignore_changes = [
      tags,
      ami,           # Don't replace when AMI updates
    ]

    # Replace when a different resource changes
    replace_triggered_by = [
      aws_security_group.web.id
    ]
  }
}
```

```
  create_before_destroy:
  
  WITHOUT (default):              WITH create_before_destroy:
  ┌──────┐                        ┌──────┐  ┌──────┐
  │ Old  │──destroy──▶ DOWNTIME   │ Old  │  │ New  │  ← Both exist
  │      │                        │      │  │      │     briefly
  └──────┘                        └──┬───┘  └──────┘
                                     │
     ┌──────┐                     destroy old
     │ New  │──create──▶ UP         
     │      │                     Result: Zero downtime!
     └──────┘
```

---

## 4.6 Real-World Example: Web Application Stack

```hcl
# Complete example: VPC + Subnet + Security Group + EC2

# 1. VPC
resource "aws_vpc" "app" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = { Name = "app-vpc" }
}

# 2. Internet Gateway
resource "aws_internet_gateway" "app" {
  vpc_id = aws_vpc.app.id
  tags   = { Name = "app-igw" }
}

# 3. Public Subnet
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.app.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = true
  availability_zone       = "us-east-1a"

  tags = { Name = "app-public-subnet" }
}

# 4. Route Table
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.app.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.app.id
  }

  tags = { Name = "app-public-rt" }
}

# 5. Route Table Association
resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}

# 6. Security Group
resource "aws_security_group" "web" {
  name   = "web-sg"
  vpc_id = aws_vpc.app.id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# 7. Data source: Latest AMI
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}

# 8. EC2 Instance
resource "aws_instance" "web" {
  ami                    = data.aws_ami.ubuntu.id
  instance_type          = "t3.micro"
  subnet_id              = aws_subnet.public.id
  vpc_security_group_ids = [aws_security_group.web.id]

  user_data = <<-EOF
    #!/bin/bash
    apt-get update
    apt-get install -y nginx
    systemctl start nginx
  EOF

  tags = { Name = "web-server" }
}

# 9. Outputs
output "public_ip" {
  value = aws_instance.web.public_ip
}
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| `Error: Resource not found` | Wrong resource type name | Check provider docs for correct type |
| `Error: Cycle detected` | Circular dependency between resources | Restructure to break the cycle, use `depends_on` carefully |
| Data source returns no results | Filter too restrictive | Widen filter criteria or check spelling |
| `Error: creating X: already exists` | Resource exists outside Terraform | Use `terraform import` to bring under management |
| Force replacement when not needed | Attribute triggers replacement | Use `ignore_changes` in lifecycle block |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Resource** | Declares infrastructure to create and manage |
| **Data Source** | Reads information about existing infrastructure |
| **Implicit Dependency** | Auto-detected from resource references |
| **Explicit Dependency** | Manual `depends_on` for hidden dependencies |
| **create_before_destroy** | Create new resource before destroying old (zero downtime) |
| **prevent_destroy** | Safety guard against accidental deletion |
| **ignore_changes** | Skip specific attributes during plan/apply |
| **Dependency Graph** | Auto-resolved order of creation/destruction |
| **Resource Reference** | `type.name.attribute` (e.g., `aws_vpc.main.id`) |
| **Data Source Reference** | `data.type.name.attribute` (e.g., `data.aws_ami.ubuntu.id`) |

---

## Quick Revision Questions

1. **What is the difference between a resource and a data source?**
   > A resource creates and manages infrastructure (CRUD). A data source only reads existing infrastructure (read-only).

2. **How does Terraform determine the order to create resources?**
   > Through implicit dependencies (references between resources) and explicit dependencies (`depends_on`).

3. **When should you use `depends_on` instead of implicit dependencies?**
   > When there's a dependency Terraform can't detect from references — e.g., an IAM policy must exist before a resource uses it, but the resource doesn't directly reference the policy.

4. **What does `create_before_destroy` do and when is it useful?**
   > It creates the replacement resource before destroying the old one, preventing downtime. Useful for load-balanced servers, DNS records, etc.

5. **How do you reference a data source attribute?**
   > Using `data.type.name.attribute`, e.g., `data.aws_ami.ubuntu.id`.

6. **What does `ignore_changes` do in a lifecycle block?**
   > It tells Terraform to ignore changes to specific attributes, so external modifications (auto-scaling, manual tags) don't trigger updates.

---

[← Previous: HCL Syntax Basics](03-hcl-syntax-basics.md) | [Back to README](../README.md) | [Next: Terraform Workflow →](05-terraform-workflow.md)
