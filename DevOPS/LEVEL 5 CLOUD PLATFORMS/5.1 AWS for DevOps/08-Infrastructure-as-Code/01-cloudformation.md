# Chapter 8.1: AWS CloudFormation — Infrastructure as Code

## Overview

AWS CloudFormation lets you **model and provision AWS resources** using declarative templates (JSON or YAML). You define the desired state, and CloudFormation creates, updates, and deletes resources in the correct order. It's the native IaC service for AWS.

---

## CloudFormation Architecture

```
┌──── CloudFormation Workflow ─────────────────────────────┐
│                                                           │
│  Developer              CloudFormation          AWS       │
│  ┌──────────┐          ┌─────────────┐     ┌──────────┐ │
│  │ Template │──upload──│             │────→│ VPC      │ │
│  │ (YAML/   │          │  Stack      │────→│ EC2      │ │
│  │  JSON)   │          │             │────→│ RDS      │ │
│  └──────────┘          │  Creates/   │────→│ S3       │ │
│                        │  Updates/   │────→│ IAM      │ │
│  Parameters ──────────→│  Deletes    │────→│ Lambda   │ │
│                        │  resources  │     └──────────┘ │
│                        │  in order   │                   │
│                        └─────────────┘                   │
│                                                           │
│  Stack = collection of AWS resources managed as a unit   │
│  ✓ Automatic dependency resolution                       │
│  ✓ Rollback on failure                                   │
│  ✓ Drift detection                                       │
└───────────────────────────────────────────────────────────┘
```

---

## Template Structure

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: "Production web application infrastructure"

# ─── Parameters: Input values at deploy time ───
Parameters:
  Environment:
    Type: String
    Default: prod
    AllowedValues: [dev, staging, prod]
  InstanceType:
    Type: String
    Default: t3.medium
    AllowedValues: [t3.micro, t3.small, t3.medium, t3.large]
  VpcCIDR:
    Type: String
    Default: "10.0.0.0/16"
    AllowedPattern: '(\d{1,3}\.){3}\d{1,3}/\d{1,2}'

# ─── Mappings: Static lookup tables ───
Mappings:
  RegionAMI:
    us-east-1:
      HVM64: ami-0abcdef1234567890
    us-west-2:
      HVM64: ami-0fedcba0987654321
    eu-west-1:
      HVM64: ami-0aabbcc112233445

# ─── Conditions: Conditional resource creation ───
Conditions:
  IsProd: !Equals [!Ref Environment, prod]
  CreateReplica: !And
    - !Condition IsProd
    - !Equals [!Ref "AWS::Region", "us-east-1"]

# ─── Resources: AWS resources to create (REQUIRED) ───
Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !Ref VpcCIDR
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: !Sub "${Environment}-vpc"

  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: "10.0.1.0/24"
      AvailabilityZone: !Select [0, !GetAZs ""]
      MapPublicIpOnLaunch: true

  WebServer:
    Type: AWS::EC2::Instance
    Condition: IsProd
    Properties:
      InstanceType: !Ref InstanceType
      ImageId: !FindInMap [RegionAMI, !Ref "AWS::Region", HVM64]
      SubnetId: !Ref PublicSubnet
      SecurityGroupIds:
        - !Ref WebSG
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
          yum update -y
          yum install -y httpd
          systemctl start httpd
          echo "<h1>${Environment} Server</h1>" > /var/www/html/index.html

  WebSG:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Web server security group
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: 0.0.0.0/0

# ─── Outputs: Values to export/display ───
Outputs:
  VPCId:
    Description: VPC ID
    Value: !Ref VPC
    Export:
      Name: !Sub "${Environment}-VPCId"
  WebServerIP:
    Condition: IsProd
    Description: Web Server Public IP
    Value: !GetAtt WebServer.PublicIp
```

---

## Intrinsic Functions Cheat Sheet

```
┌──── Key Functions ───────────────────────────────────────┐
│                                                          │
│  !Ref ResourceOrParam      → Resource ID or param value │
│  !GetAtt Resource.Attr     → Resource attribute          │
│  !Sub "Hello ${VarName}"   → String substitution        │
│  !Join ["-", [a, b, c]]   → "a-b-c"                    │
│  !Select [0, [a, b, c]]   → "a"                        │
│  !Split [",", "a,b,c"]    → [a, b, c]                  │
│  !FindInMap [Map, K1, K2]  → Mapped value               │
│  !GetAZs ""                → AZs in current region      │
│  !ImportValue ExportName   → Cross-stack reference      │
│  !If [Cond, True, False]   → Conditional value          │
│  !Equals [a, b]            → Boolean condition          │
│  Fn::Base64                → Base64 encode (UserData)   │
│  !Cidr [vpc, count, bits]  → Generate CIDR blocks       │
└──────────────────────────────────────────────────────────┘
```

---

## Stack Operations

```bash
# Create stack
aws cloudformation create-stack \
  --stack-name prod-app \
  --template-body file://template.yml \
  --parameters ParameterKey=Environment,ParameterValue=prod \
               ParameterKey=InstanceType,ParameterValue=t3.medium \
  --capabilities CAPABILITY_IAM \
  --tags Key=Project,Value=MyApp

# Update stack
aws cloudformation update-stack \
  --stack-name prod-app \
  --template-body file://template.yml \
  --parameters ParameterKey=InstanceType,ParameterValue=t3.large

# Create change set (preview changes before applying)
aws cloudformation create-change-set \
  --stack-name prod-app \
  --change-set-name update-instance-type \
  --template-body file://template.yml \
  --parameters ParameterKey=InstanceType,ParameterValue=t3.large

# Execute change set
aws cloudformation execute-change-set \
  --stack-name prod-app \
  --change-set-name update-instance-type

# Delete stack
aws cloudformation delete-stack --stack-name prod-app

# Validate template
aws cloudformation validate-template --template-body file://template.yml

# Detect drift
aws cloudformation detect-stack-drift --stack-name prod-app
```

---

## Nested Stacks & Cross-Stack References

```
┌──── Nested Stacks ───────────────────────────────────────┐
│                                                          │
│  Root Stack                                              │
│  ├── Network Stack (VPC, Subnets, IGW)                  │
│  │   └── Outputs: VPCId, SubnetIds                      │
│  ├── Security Stack (SGs, NACLs, IAM)                   │
│  │   └── Outputs: SGIds, RoleArns                       │
│  └── App Stack (EC2, ALB, ASG)                          │
│      └── Uses outputs from Network & Security stacks    │
│                                                          │
│  Cross-Stack References:                                 │
│  ┌──────────────┐                ┌──────────────┐       │
│  │ Stack A      │                │ Stack B      │       │
│  │ Outputs:     │                │ Resources:   │       │
│  │  Export:     │ !ImportValue   │  VpcId:      │       │
│  │   VPCId  ───────────────────→│   !Import..  │       │
│  └──────────────┘                └──────────────┘       │
└──────────────────────────────────────────────────────────┘
```

```yaml
# Nested stack resource
Resources:
  NetworkStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/mybucket/network.yml
      Parameters:
        Environment: !Ref Environment
```

---

## Stack Policies & Protection

```json
{
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "Update:*",
      "Principal": "*",
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Action": "Update:Replace",
      "Principal": "*",
      "Resource": "LogicalResourceId/Database"
    }
  ]
}
```

```bash
# Enable termination protection
aws cloudformation update-termination-protection \
  --stack-name prod-app \
  --enable-termination-protection

# Set stack policy (prevent accidental resource replacement)
aws cloudformation set-stack-policy \
  --stack-name prod-app \
  --stack-policy-body file://stack-policy.json
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| `CREATE_FAILED` rollback | Resource creation error | Check events tab; fix template and retry |
| `UPDATE_ROLLBACK_FAILED` | Can't restore previous state | Use `continue-update-rollback` with resources to skip |
| `CAPABILITY_IAM` required | Template creates IAM resources | Add `--capabilities CAPABILITY_IAM` or `CAPABILITY_NAMED_IAM` |
| Circular dependency | Resources reference each other | Use `DependsOn` or break into separate stacks |
| Drift detected | Manual changes outside CloudFormation | Import changes or revert; always use CF for changes |
| Delete fails | Non-empty S3 bucket or resource in use | Empty bucket first; remove dependencies |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **CloudFormation** | Declarative IaC. Define desired state in YAML/JSON templates. |
| **Stack** | Collection of resources managed as one unit |
| **Change Sets** | Preview changes before applying to a stack |
| **Intrinsic Functions** | !Ref, !Sub, !GetAtt, !FindInMap, !If, etc. |
| **Nested Stacks** | Reusable template components; parent-child relationship |
| **Cross-Stack** | Export/ImportValue for sharing between independent stacks |
| **Drift Detection** | Detect manual changes to stack resources |

---

## Quick Revision Questions

1. **What is a CloudFormation stack?**
   > A collection of AWS resources created, updated, and deleted as a single unit from a template.

2. **What is the only required section in a CloudFormation template?**
   > `Resources` — defines the AWS resources to create.

3. **What is a change set?**
   > A preview of proposed changes before applying an update, letting you verify what will be added, modified, or replaced.

4. **How do you share values between stacks?**
   > Use `Export` in Outputs of Stack A and `!ImportValue` in Stack B. Or use nested stacks with parameter passing.

5. **What happens when a stack creation fails?**
   > CloudFormation automatically rolls back, deleting all resources that were created during the failed attempt.

6. **What is drift detection?**
   > Identifies differences between the expected (template) state and actual state of resources, caused by manual changes outside CloudFormation.

---

[← Previous: 7.6 GitHub Actions](../07-CICD-on-AWS/06-github-actions-with-aws.md) | [Next: 8.2 CDK →](02-cdk.md)

[← Back to README](../README.md)
