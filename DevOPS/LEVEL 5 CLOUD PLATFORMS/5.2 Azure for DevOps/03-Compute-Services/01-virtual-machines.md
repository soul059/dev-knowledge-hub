# Chapter 1: Virtual Machines

## Overview
Azure Virtual Machines (VMs) provide on-demand, scalable computing resources. VMs are the foundation of IaaS in Azure, giving DevOps engineers full control over the operating system, software, and configurations.

---

## 1.1 Azure VM Architecture

```
┌────────────────────── AZURE VM ARCHITECTURE ──────────────────────┐
│                                                                    │
│   ┌──────────────────────────────────────────────────────┐        │
│   │                 VIRTUAL MACHINE                       │        │
│   │                                                       │        │
│   │   ┌─────────────┐  ┌─────────────┐  ┌────────────┐  │        │
│   │   │     OS      │  │  Temporary  │  │  Data Disk │  │        │
│   │   │    Disk     │  │    Disk     │  │ (optional) │  │        │
│   │   │ (managed)   │  │ (ephemeral) │  │ (managed)  │  │        │
│   │   └─────────────┘  └─────────────┘  └────────────┘  │        │
│   │                                                       │        │
│   │   ┌─────────────┐  ┌─────────────────────────────┐  │        │
│   │   │     NIC     │  │   VM Size (CPU, RAM, IOPS)  │  │        │
│   │   │  (Network   │  │   e.g., Standard_B2s        │  │        │
│   │   │  Interface) │  │   2 vCPU, 4 GB RAM           │  │        │
│   │   └──────┬──────┘  └─────────────────────────────┘  │        │
│   │          │                                            │        │
│   └──────────┼────────────────────────────────────────────┘        │
│              │                                                     │
│   ┌──────────┴──────────┐                                         │
│   │     Subnet          │                                         │
│   │  ┌──────────────┐   │                                         │
│   │  │     NSG      │   │                                         │
│   │  │(Network Sec.)│   │                                         │
│   │  └──────────────┘   │                                         │
│   │  ┌──────────────┐   │                                         │
│   │  │  Public IP   │   │                                         │
│   │  │  (optional)  │   │                                         │
│   │  └──────────────┘   │                                         │
│   └─────────────────────┘                                         │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

---

## 1.2 VM Size Families

| Family | Prefix | Use Case | Example |
|--------|--------|----------|---------|
| **B-series** | B | Burstable, dev/test | Standard_B2s |
| **D-series** | D | General purpose | Standard_D4s_v5 |
| **E-series** | E | Memory-optimized | Standard_E8s_v5 |
| **F-series** | F | Compute-optimized | Standard_F4s_v2 |
| **L-series** | L | Storage-optimized | Standard_L8s_v3 |
| **N-series** | N | GPU (AI/ML, rendering) | Standard_NC6s_v3 |
| **M-series** | M | Memory-intensive (SAP) | Standard_M128s |

### VM Size Naming Convention

```
Standard_D4s_v5
    │      │││ │
    │      │││ └── Version (generation)
    │      ││└──── v = version indicator
    │      │└───── s = Premium SSD capable
    │      └────── 4 = vCPU count
    └───────────── D = Family (general purpose)
```

---

## 1.3 Creating VMs

### Azure CLI

```bash
# Create a Linux VM
az vm create \
  --resource-group myRG \
  --name myLinuxVM \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --admin-username azureuser \
  --generate-ssh-keys \
  --public-ip-sku Standard \
  --location eastus

# Create a Windows VM
az vm create \
  --resource-group myRG \
  --name myWinVM \
  --image Win2022Datacenter \
  --size Standard_D2s_v5 \
  --admin-username azureuser \
  --admin-password "SecureP@ss123!" \
  --public-ip-sku Standard

# Create VM with managed identity
az vm create \
  --resource-group myRG \
  --name myVM \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --assign-identity \
  --admin-username azureuser \
  --generate-ssh-keys

# Create VM in availability zone
az vm create \
  --resource-group myRG \
  --name myVM-z1 \
  --image Ubuntu2204 \
  --zone 1 \
  --size Standard_B2s \
  --admin-username azureuser \
  --generate-ssh-keys
```

### VM Management Commands

```bash
# List VMs
az vm list --output table

# Start/Stop/Restart
az vm start --resource-group myRG --name myVM
az vm stop --resource-group myRG --name myVM          # Still billing!
az vm deallocate --resource-group myRG --name myVM    # Stops billing
az vm restart --resource-group myRG --name myVM

# Resize VM
az vm resize --resource-group myRG --name myVM --size Standard_D4s_v5

# Get VM IP address
az vm show -d --resource-group myRG --name myVM --query publicIps -o tsv

# Run command on VM
az vm run-command invoke \
  --resource-group myRG \
  --name myVM \
  --command-id RunShellScript \
  --scripts "apt update && apt install -y nginx"

# Delete VM
az vm delete --resource-group myRG --name myVM --yes
```

---

## 1.4 VM Extensions

Extensions add post-deployment configuration and automation.

```bash
# Install Custom Script Extension (Linux)
az vm extension set \
  --resource-group myRG \
  --vm-name myVM \
  --name customScript \
  --publisher Microsoft.Azure.Extensions \
  --settings '{"commandToExecute": "apt update && apt install -y nginx"}'

# Install Azure Monitor agent
az vm extension set \
  --resource-group myRG \
  --vm-name myVM \
  --name AzureMonitorLinuxAgent \
  --publisher Microsoft.Azure.Monitor

# List extensions
az vm extension list --resource-group myRG --vm-name myVM --output table
```

---

## 1.5 Cloud-Init (Linux VM Bootstrapping)

```yaml
#cloud-config
package_upgrade: true
packages:
  - nginx
  - docker.io
  - git
write_files:
  - path: /var/www/html/index.html
    content: |
      <h1>Hello from Azure VM!</h1>
      <p>Deployed via cloud-init</p>
runcmd:
  - systemctl enable nginx
  - systemctl start nginx
  - usermod -aG docker azureuser
```

```bash
# Create VM with cloud-init
az vm create \
  --resource-group myRG \
  --name myVM \
  --image Ubuntu2204 \
  --custom-data cloud-init.yaml \
  --admin-username azureuser \
  --generate-ssh-keys
```

---

## 1.6 VM Availability Options

```
┌──────────── VM AVAILABILITY OPTIONS ──────────────┐
│                                                     │
│   AVAILABILITY SETS          AVAILABILITY ZONES     │
│   ──────────────────         ──────────────────     │
│                                                     │
│   Same Datacenter            Across Datacenters     │
│   99.95% SLA                 99.99% SLA             │
│                                                     │
│   ┌───────────────┐         Zone1   Zone2   Zone3  │
│   │  Fault Domain │         ┌───┐   ┌───┐   ┌───┐ │
│   │  ┌───┐ ┌───┐  │         │VM1│   │VM2│   │VM3│ │
│   │  │VM1│ │VM2│  │         └───┘   └───┘   └───┘ │
│   │  └───┘ └───┘  │                                │
│   ├───────────────┤                                │
│   │ Update Domain │                                │
│   │  ┌───┐ ┌───┐  │                                │
│   │  │VM3│ │VM4│  │                                │
│   │  └───┘ └───┘  │                                │
│   └───────────────┘                                │
│                                                     │
│   Fault Domain: Separate rack (power/network)       │
│   Update Domain: Separate maintenance window        │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## 1.7 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| VM won't start | Quota exceeded | Check: `az vm list-usage --location eastus` |
| Can't SSH/RDP | NSG blocking traffic | Check NSG rules: `az network nsg rule list` |
| VM is slow | Wrong size or disk type | Resize or upgrade to Premium SSD |
| High cost | VM running when not needed | Auto-shutdown or use `az vm deallocate` |
| Custom Script fails | Download error or permissions | Check extension logs: `/var/log/azure/` |
| `Stop` still billing | `stop` ≠ `deallocate` | Use `az vm deallocate` to stop charges |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **VM Families** | B (burst), D (general), E (memory), F (compute), N (GPU) |
| **Disks** | OS disk (managed), temp disk (ephemeral), data disks (optional) |
| **Availability** | AZ (99.99%) > Availability Set (99.95%) > Single VM (99.9%) |
| **Extensions** | Post-deployment automation (Custom Script, Monitor Agent) |
| **Cloud-Init** | YAML config for Linux VM bootstrapping |
| **Cost Control** | Deallocate (not just stop) to eliminate charges |

---

## Quick Revision Questions

1. **What is the difference between `az vm stop` and `az vm deallocate`?**
   > `stop` keeps the VM allocated (still charges for compute), while `deallocate` releases compute resources and stops billing.

2. **What does the 's' in Standard_D4s_v5 mean?**
   > It indicates the VM supports Premium SSD storage.

3. **Name three VM availability strategies in order of increasing SLA.**
   > Single VM (99.9%), Availability Set (99.95%), Availability Zones (99.99%).

4. **What is cloud-init?**
   > A YAML-based configuration format for bootstrapping Linux VMs during first boot — installing packages, writing files, running commands.

5. **What are Fault Domains and Update Domains in an Availability Set?**
   > Fault Domains separate VMs across different racks (power/network). Update Domains ensure VMs aren't all updated/rebooted simultaneously during maintenance.

6. **How do you run a command on a VM without SSH access?**
   > `az vm run-command invoke --command-id RunShellScript --scripts "your command"`.

---

[⬅ Previous: Azure Policy](../02-Identity-and-Access/05-azure-policy.md) | [⬆ Back to Table of Contents](../README.md) | [Next: VM Scale Sets ➡](02-vm-scale-sets.md)
