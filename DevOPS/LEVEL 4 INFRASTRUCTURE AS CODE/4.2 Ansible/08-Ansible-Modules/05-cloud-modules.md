# Chapter 8.5: Cloud Modules

[← Previous: User and Group Modules](04-user-and-group-modules.md) | [Next: Delegation →](../09-Advanced-Features/01-delegation.md)

---

## Overview

Ansible provides modules for provisioning and managing cloud infrastructure across major providers — **AWS**, **Azure**, **GCP**, and others. Cloud modules run on the control node (`connection: local`) and interact with cloud APIs.

---

## Cloud Module Architecture

```
  ANSIBLE CLOUD MODULE FLOW
  ══════════════════════════════════════════════════════════

  ┌─────────────┐    API Calls    ┌──────────────────┐
  │  Control    │────────────────▶│  Cloud Provider  │
  │  Node       │                 │  API             │
  │             │◀────────────────│                  │
  │ (localhost) │    Responses    │  AWS / Azure /   │
  └─────────────┘                 │  GCP / etc.      │
                                  └──────────────────┘

  • Runs on control node (connection: local)
  • Uses cloud SDKs (boto3, azure-sdk, google-auth)
  • Authenticates via env vars, config files, or module params
```

---

## AWS (Amazon Web Services)

### Prerequisites

```bash
# Install AWS collection and SDK
pip install boto3 botocore
ansible-galaxy collection install amazon.aws
ansible-galaxy collection install community.aws
```

### Authentication

```yaml
# Option 1: Environment variables
# export AWS_ACCESS_KEY_ID='AKID...'
# export AWS_SECRET_ACCESS_KEY='secret...'
# export AWS_REGION='eu-west-1'

# Option 2: Module parameters
- name: Launch EC2
  amazon.aws.ec2_instance:
    aws_access_key: "{{ aws_access_key }}"
    aws_secret_key: "{{ aws_secret_key }}"
    region: eu-west-1
```

### EC2 Instances

```yaml
---
- name: Provision AWS infrastructure
  hosts: localhost
  connection: local
  tasks:
    - name: Create security group
      amazon.aws.ec2_security_group:
        name: web-sg
        description: Web server security group
        region: eu-west-1
        rules:
          - proto: tcp
            from_port: 80
            to_port: 80
            cidr_ip: 0.0.0.0/0
          - proto: tcp
            from_port: 22
            to_port: 22
            cidr_ip: 10.0.0.0/8

    - name: Launch EC2 instance
      amazon.aws.ec2_instance:
        name: web-server-01
        instance_type: t3.micro
        image_id: ami-0abcdef1234567890
        key_name: my-keypair
        security_groups:
          - web-sg
        region: eu-west-1
        network:
          assign_public_ip: true
        tags:
          Environment: production
          Role: webserver
        state: running
      register: ec2_result

    - name: Show instance details
      debug:
        msg: "Instance ID: {{ ec2_result.instances[0].instance_id }}"
```

### S3

```yaml
- name: Create S3 bucket
  amazon.aws.s3_bucket:
    name: my-app-assets-bucket
    region: eu-west-1
    versioning: true
    tags:
      Environment: production

- name: Upload file to S3
  amazon.aws.s3_object:
    bucket: my-app-assets-bucket
    object: config/app.conf
    src: files/app.conf
    mode: put
```

---

## Azure

### Prerequisites

```bash
# Install Azure collection and SDK
pip install azure-identity azure-mgmt-compute azure-mgmt-network
ansible-galaxy collection install azure.azcollection
pip install -r ~/.ansible/collections/ansible_collections/azure/azcollection/requirements.txt
```

### Authentication

```yaml
# Option 1: Environment variables
# export AZURE_SUBSCRIPTION_ID='...'
# export AZURE_CLIENT_ID='...'
# export AZURE_SECRET='...'
# export AZURE_TENANT='...'

# Option 2: credentials file (~/.azure/credentials)
# [default]
# subscription_id=...
# client_id=...
# secret=...
# tenant=...
```

### Azure Virtual Machine

```yaml
---
- name: Provision Azure infrastructure
  hosts: localhost
  connection: local
  tasks:
    - name: Create resource group
      azure.azcollection.azure_rm_resourcegroup:
        name: myResourceGroup
        location: uksouth

    - name: Create virtual network
      azure.azcollection.azure_rm_virtualnetwork:
        resource_group: myResourceGroup
        name: myVnet
        address_prefixes: "10.0.0.0/16"

    - name: Create subnet
      azure.azcollection.azure_rm_subnet:
        resource_group: myResourceGroup
        virtual_network: myVnet
        name: mySubnet
        address_prefix: "10.0.1.0/24"

    - name: Create public IP
      azure.azcollection.azure_rm_publicipaddress:
        resource_group: myResourceGroup
        allocation_method: Static
        name: myPublicIP

    - name: Create VM
      azure.azcollection.azure_rm_virtualmachine:
        resource_group: myResourceGroup
        name: myVM
        vm_size: Standard_B1s
        admin_username: azureuser
        ssh_password_enabled: false
        ssh_public_keys:
          - path: /home/azureuser/.ssh/authorized_keys
            key_data: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
        image:
          offer: 0001-com-ubuntu-server-jammy
          publisher: Canonical
          sku: 22_04-lts
          version: latest
```

---

## GCP (Google Cloud Platform)

### Prerequisites

```bash
pip install google-auth google-api-python-client
ansible-galaxy collection install google.cloud
```

### GCE Instance

```yaml
---
- name: Provision GCP infrastructure
  hosts: localhost
  connection: local
  vars:
    gcp_project: my-project-id
    gcp_zone: europe-west2-a

  tasks:
    - name: Create GCE instance
      google.cloud.gcp_compute_instance:
        name: web-server-01
        machine_type: e2-micro
        zone: "{{ gcp_zone }}"
        project: "{{ gcp_project }}"
        auth_kind: serviceaccount
        service_account_file: /path/to/sa-key.json
        disks:
          - auto_delete: true
            boot: true
            initialize_params:
              source_image: projects/ubuntu-os-cloud/global/images/family/ubuntu-2204-lts
              disk_size_gb: 20
        network_interfaces:
          - network:
              selfLink: global/networks/default
            access_configs:
              - name: External NAT
                type: ONE_TO_ONE_NAT
        tags:
          items:
            - http-server
        state: present
```

---

## Dynamic Inventory with Cloud

```yaml
# aws_ec2.yml — dynamic inventory plugin
plugin: amazon.aws.aws_ec2
regions:
  - eu-west-1
filters:
  tag:Environment: production
  instance-state-name: running
keyed_groups:
  - key: tags.Role
    prefix: role
  - key: placement.availability_zone
    prefix: az
hostnames:
  - private-ip-address
compose:
  ansible_host: private_ip_address
```

```bash
# Use dynamic inventory
ansible-inventory -i aws_ec2.yml --graph
ansible-playbook -i aws_ec2.yml site.yml
```

---

## Cloud Module Pattern: Provision then Configure

```yaml
# 1. Provision infrastructure
- name: Provision servers
  hosts: localhost
  connection: local
  tasks:
    - name: Create EC2 instances
      amazon.aws.ec2_instance:
        name: "web-{{ item }}"
        instance_type: t3.micro
        image_id: ami-0abcdef1234567890
        state: running
      loop: [1, 2, 3]
      register: ec2_instances

    - name: Add to in-memory inventory
      add_host:
        name: "{{ item.instances[0].private_ip_address }}"
        groups: webservers
      loop: "{{ ec2_instances.results }}"

    - name: Wait for SSH
      wait_for:
        host: "{{ item.instances[0].private_ip_address }}"
        port: 22
        timeout: 300
      loop: "{{ ec2_instances.results }}"

# 2. Configure the provisioned servers
- name: Configure web servers
  hosts: webservers
  become: yes
  roles:
    - common
    - webserver
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| `boto3 required for this module` | `pip install boto3 botocore` |
| Authentication failure | Check env vars or credentials file |
| Collection not found | `ansible-galaxy collection install <name>` |
| `connection: local` missing | Cloud modules run on control node, not remote |
| Timeout creating resources | Increase module `wait_timeout` parameter |
| Idempotency issues | Use `name` or `tags` to identify existing resources |

---

## Summary Table

| Provider | Collection | SDK | Auth Method |
|----------|-----------|-----|-------------|
| AWS | `amazon.aws` | boto3 | Env vars, IAM role, config |
| Azure | `azure.azcollection` | azure-sdk | Env vars, credentials file, MSI |
| GCP | `google.cloud` | google-auth | Service account JSON |
| DigitalOcean | `community.digitalocean` | — | API token |
| VMware | `community.vmware` | PyVmomi | vCenter credentials |

---

## Quick Revision Questions

1. **Why do cloud module playbooks use `connection: local`?**
   > Cloud modules make API calls from the control node — they don't SSH into the cloud instances.

2. **What Python library is required for AWS modules?**
   > `boto3` and `botocore` — the AWS SDK for Python.

3. **How do you make cloud provisioning idempotent?**
   > Use unique resource names or tags. The modules check if a resource with that identifier exists before creating.

4. **What is the typical two-stage pattern for cloud automation?**
   > Stage 1: Provision infrastructure (`connection: local`). Stage 2: Configure the provisioned hosts (normal SSH connection).

5. **How do cloud dynamic inventory plugins work?**
   > They query the cloud API for running instances and automatically build the Ansible inventory, grouping hosts by tags, regions, or other attributes.

---

[← Previous: User and Group Modules](04-user-and-group-modules.md) | [Next: Delegation →](../09-Advanced-Features/01-delegation.md)
