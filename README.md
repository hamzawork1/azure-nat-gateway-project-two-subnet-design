# 🌐 Azure NAT Gateway Egress Test (Ubuntu + Windows)

## 📘 Overview

This project demonstrates how to configure **Azure NAT Gateway** to control **outbound internet access** from virtual machines in different subnets.

- **VM1** (Ubuntu) resides in a **private subnet** with **NAT Gateway**, allowing secure outbound access.
- **VM2** (Windows 10) is placed in a **default subnet** without NAT Gateway, and cannot reach the internet.
- **Azure Bastion** is used to securely access both VMs via browser (SSH/RDP) without public IPs.

---

## 🏗️ Architecture

vnet-nat-dev (10.10.0.0/16)
│
├── snet-private-1 (10.10.1.0/24) ──▶ NAT Gateway (natgw-dev)
│ └── vm1-ubuntu-dev (Ubuntu, Outbound ✅)
│
├── snet-default-2 (10.10.2.0/24) ──▶ No NAT Gateway
│ └── vm2-win10-dev (Windows 10, Outbound ❌)
│
└── AzureBastionSubnet (10.10.100.0/27)
└── Azure Bastion (bastion-dev)


---

## 📦 Resources

| Resource Type | Name |
|---------------|------|
| VNet | `vnet-nat-dev` |
| Subnets | `snet-private-1`, `snet-default-2`, `AzureBastionSubnet` |
| NAT Gateway | `natgw-dev` |
| Public IP | `pip-natgw-dev` |
| Bastion Host | `bastion-dev` |
| VMs | `vm1-ubuntu-dev`, `vm2-win10-dev` |
| NSGs | `nsg-vm1`, `nsg-vm2` |
| NICs | `nic-vm1`, `nic-vm2` |

---

## 🔐 Security Design

Each VM is associated with its own **Network Security Group (NSG)**:

- **Inbound Rules:**
  - Allow SSH (Ubuntu) or RDP (Windows) from Azure Bastion
  - Deny all other inbound traffic

- **Outbound Rules:**
  - Allow HTTP/HTTPS, DNS for system updates
  - Additional outbound filtering (optional)

---

## 🧪 Test Instructions

### Connect to VMs:
1. Use **Azure Bastion** from the portal
2. Connect to:
   - `vm1-ubuntu-dev` via SSH
   - `vm2-win10-dev` via RDP

### Validate NAT Gateway (VM1):
```bash
curl ifconfig.me            # Returns public IP of NAT GW
sudo apt update             # Should succeed
```
Validate No Internet (VM2):
Invoke-WebRequest -Uri "http://ifconfig.me"  # Fails
ping google.com                              # Fails

✅ Expected Results

VM	Subnet	NAT Gateway	Internet Access
vm1-ubuntu-dev	snet-private-1	✅ Yes	✅ Yes
vm2-win10-dev	snet-default-2	❌ No	❌ No

📌 Notes
Public IP Prefix is not used (not required unless multiple static IPs are needed)

No load balancer or inbound web service is configured

This setup is ideal for testing controlled outbound access in real-world Azure network designs

