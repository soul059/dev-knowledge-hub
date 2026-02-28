# Chapter 1: Provisioners

[← Previous: Custom Providers](../07-Providers/05-custom-providers.md) | [Back to README](../README.md) | [Next: Terraform Functions Deep Dive →](02-terraform-functions.md)

---

## Overview

**Provisioners** execute scripts or commands on a local or remote machine as part of resource creation or destruction. They are a **last resort** mechanism — Terraform recommends using native provider features, cloud-init, or configuration management tools instead.

---

## 1.1 When to Use Provisioners

```
┌──────────────────────────────────────────────────────────┐
│           PROVISIONERS: LAST RESORT                       │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  PREFER these alternatives:                               │
│  ├── Cloud-init / user_data (AWS, Azure, GCP)            │
│  ├── Packer (pre-baked AMIs with software)               │
│  ├── Ansible / Chef / Puppet (config management)         │
│  ├── Kubernetes manifests (container workloads)          │
│  └── Provider-native resources (SSM, Run Command)        │
│                                                           │
│  USE provisioners when:                                   │
│  ├── No provider support for the action                  │
│  ├── Bootstrapping that can't use cloud-init             │
│  ├── Running one-off scripts on create/destroy           │
│  └── Integrating with legacy systems                     │
│                                                           │
│  PROBLEMS with provisioners:                              │
│  ├── Not in Terraform state (plan can't show them)       │
│  ├── If they fail, resource is tainted                   │
│  ├── Not idempotent by default                           │
│  └── Require network access to remote machines           │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 1.2 Provisioner Types

### local-exec

Runs commands on the **machine running Terraform**.

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"

  # Runs on YOUR machine (where terraform runs)
  provisioner "local-exec" {
    command = "echo ${self.private_ip} >> inventory.txt"
  }

  # With interpreter (Windows)
  provisioner "local-exec" {
    command     = "echo ${self.private_ip} > inventory.txt"
    interpreter = ["PowerShell", "-Command"]
  }

  # With working directory and environment
  provisioner "local-exec" {
    command     = "./scripts/register.sh"
    working_dir = path.module
    environment = {
      SERVER_IP   = self.private_ip
      SERVER_NAME = self.tags["Name"]
    }
  }
}
```

### remote-exec

Runs commands **on the remote resource** via SSH or WinRM.

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  key_name      = aws_key_pair.deployer.key_name

  # Connection details for remote-exec
  connection {
    type        = "ssh"
    user        = "ubuntu"
    private_key = file("~/.ssh/deployer.pem")
    host        = self.public_ip
  }

  # Inline commands
  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update",
      "sudo apt-get install -y nginx",
      "sudo systemctl start nginx",
    ]
  }

  # Or run a script
  provisioner "remote-exec" {
    script = "scripts/setup.sh"
  }

  # Or multiple scripts
  provisioner "remote-exec" {
    scripts = [
      "scripts/install.sh",
      "scripts/configure.sh",
    ]
  }
}
```

### file

Copies files or directories to a remote resource.

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  key_name      = aws_key_pair.deployer.key_name

  connection {
    type        = "ssh"
    user        = "ubuntu"
    private_key = file("~/.ssh/deployer.pem")
    host        = self.public_ip
  }

  # Copy a single file
  provisioner "file" {
    source      = "configs/app.conf"
    destination = "/etc/myapp/app.conf"
  }

  # Copy a directory
  provisioner "file" {
    source      = "configs/"
    destination = "/etc/myapp"
  }

  # Create file from content
  provisioner "file" {
    content     = templatefile("${path.module}/templates/config.tpl", {
      db_host = aws_db_instance.main.address
      db_port = aws_db_instance.main.port
    })
    destination = "/etc/myapp/config.yml"
  }
}
```

---

## 1.3 Provisioner Lifecycle

```
┌──────────────────────────────────────────────────────────┐
│          PROVISIONER EXECUTION FLOW                       │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  terraform apply                                          │
│       │                                                   │
│       ▼                                                   │
│  Create Resource ──► Success ──► Run Provisioners         │
│       │                               │                   │
│       │                         ┌─────┴─────┐            │
│       │                         │           │             │
│       │                     Success      Fail             │
│       │                         │           │             │
│       │                         ▼           ▼             │
│       │                      Done      Resource           │
│       │                                TAINTED            │
│       │                                   │               │
│       │                                   ▼               │
│       │                            Next apply:            │
│       │                            Destroy +              │
│       │                            Re-create              │
│       │                                                   │
│  on_failure = "continue" ──► Ignore error, mark success  │
│  on_failure = "fail"    ──► Taint resource (default)     │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 1.4 Creation-Time vs Destroy-Time

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"

  # ─── Creation-time provisioner (default) ────────
  provisioner "local-exec" {
    command = "echo 'Instance ${self.id} created' >> log.txt"
  }

  # ─── Destroy-time provisioner ──────────────────
  provisioner "local-exec" {
    when    = destroy
    command = "echo 'Instance ${self.id} being destroyed' >> log.txt"
  }

  # ─── Handle failure gracefully ──────────────────
  provisioner "local-exec" {
    command    = "scripts/optional-setup.sh"
    on_failure = continue    # Don't taint resource if this fails
  }
}
```

> **Important:** Destroy-time provisioners can only reference `self`, `count.index`, and `each.key`. They cannot reference other resources.

---

## 1.5 terraform_data Resource (Replacement)

```hcl
# Modern alternative to null_resource for running provisioners
resource "terraform_data" "bootstrap" {
  # Re-run when these values change
  triggers_replace = [
    aws_instance.web.id,
    var.config_version,
  ]

  # Store data in state
  input = {
    instance_ip = aws_instance.web.private_ip
    version     = var.app_version
  }

  connection {
    type        = "ssh"
    host        = aws_instance.web.public_ip
    user        = "ubuntu"
    private_key = file("~/.ssh/key.pem")
  }

  provisioner "remote-exec" {
    inline = [
      "sudo systemctl restart myapp",
    ]
  }
}

# Access stored input
output "bootstrap_data" {
  value = terraform_data.bootstrap.output
}
```

---

## 1.6 Connection Block

```hcl
# ─── SSH Connection ──────────────────────────────
connection {
  type        = "ssh"
  user        = "ubuntu"
  private_key = file("~/.ssh/key.pem")
  host        = self.public_ip
  port        = 22
  timeout     = "5m"

  # Optional: bastion/jump host
  bastion_host        = "bastion.example.com"
  bastion_user        = "admin"
  bastion_private_key = file("~/.ssh/bastion.pem")
}

# ─── WinRM Connection (Windows) ──────────────────
connection {
  type     = "winrm"
  user     = "Administrator"
  password = var.admin_password
  host     = self.public_ip
  https    = true
  insecure = true
  port     = 5986
  timeout  = "10m"
}
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Resource tainted after provisioner | Provisioner script failed | Fix script; use `on_failure = continue` if optional |
| SSH connection timeout | Security group blocking port 22 | Ensure SG allows SSH from Terraform runner |
| Remote-exec hangs | Interactive prompt in script | Use `-y` flags, set `DEBIAN_FRONTEND=noninteractive` |
| Destroy provisioner can't access vars | Limitation of destroy provisioners | Use only `self` references in destroy provisioners |
| File provisioner permission denied | Wrong user or directory | Ensure user has write access to destination |

---

## Summary Table

| Provisioner | Runs On | Use Case |
|-------------|---------|----------|
| **local-exec** | Terraform machine | Scripts, API calls, inventory updates |
| **remote-exec** | Target resource | Software installation, configuration |
| **file** | Target resource | Copy files/configs to remote machine |
| **connection** | N/A (config) | SSH/WinRM connection details |
| **terraform_data** | N/A (resource) | Modern null_resource for provisioners |

---

## Quick Revision Questions

1. **Why are provisioners considered a last resort?**
   > They aren't reflected in the plan, aren't idempotent, can taint resources on failure, and require network access. Provider-native resources, cloud-init, and Packer are better alternatives.

2. **What is the difference between local-exec and remote-exec?**
   > `local-exec` runs commands on the machine running Terraform. `remote-exec` runs commands on the target resource via SSH or WinRM.

3. **What happens when a provisioner fails?**
   > By default (`on_failure = fail`), the resource is marked as tainted and will be destroyed and recreated on the next apply.

4. **What is `terraform_data` and why use it over `null_resource`?**
   > `terraform_data` is a built-in resource (no provider needed) that replaces `null_resource`. It supports `triggers_replace`, `input`/`output` for storing data, and provisioners.

5. **Can destroy-time provisioners reference other resources?**
   > No. They can only reference `self`, `count.index`, and `each.key` because other resources may already be destroyed.

---

[← Previous: Custom Providers](../07-Providers/05-custom-providers.md) | [Back to README](../README.md) | [Next: Terraform Functions Deep Dive →](02-terraform-functions.md)
