# Phase 1 — Azure Setup

**Goal:** Create two free-tier Azure VMs (DC01 and CLIENT01) with a secure Virtual Network and least-privilege NSG rules.

**Time:** ~2 hours | **Cost:** $0 (Azure Free Tier)

---

## Table of Contents
- [Billing Protection First](#billing-protection-first)
- [Create Resource Group](#create-resource-group)
- [Create Virtual Network](#create-virtual-network)
- [Launch DC01](#launch-dc01)
- [Launch CLIENT01](#launch-client01)
- [Configure NSG Rules](#configure-nsg-rules)
- [Connect via RDP](#connect-via-rdp)
- [Stop VMs](#stop-vms)
- [Troubleshooting](#troubleshooting)

---

## Billing Protection First

> ⚠️ Do this before creating anything. Azure gives you $200 credit — but it can run out if you leave VMs running.

### Enable Cost Alerts

1. Go to [portal.azure.com](https://portal.azure.com)
2. Search bar → **Cost Management + Billing** → open it
3. Left sidebar → **Cost alerts** → **Add**
4. Alert type: **Budget**
5. Budget name: `helpdesk-lab-budget`
6. Amount: `$10`
7. Alert condition: 80% of budget
8. Email: your email address
9. Click **Create**

You will now get an email if your spend approaches $10.

---

## Create Resource Group

A Resource Group is a folder that holds all your Azure resources for this project.

1. Search bar → **Resource groups** → **Create**
2. Fill in:

   | Field | Value |
   |---|---|
   | Subscription | Your free subscription |
   | Resource group name | `helpdesk-lab-rg` |
   | Region | Choose nearest to you (e.g. East US, UK South) |

3. Click **Review + create** → **Create**

> Keep everything in the same Resource Group. When you are done with the lab, deleting the Resource Group deletes everything inside it — clean and easy.

---

## Create Virtual Network

The VNet is the private network your VMs communicate over.

1. Search → **Virtual networks** → **Create**
2. Fill in:

   | Field | Value |
   |---|---|
   | Resource group | `helpdesk-lab-rg` |
   | Name | `helpdesk-vnet` |
   | Region | Same as your Resource Group |

3. **IP Addresses tab:**
   - Address space: `10.0.0.0/16`
   - Subnet name: `default`
   - Subnet range: `10.0.1.0/24`

4. Click **Review + create** → **Create**

---

## Launch DC01

DC01 will become your Domain Controller.

1. Search → **Virtual machines** → **Create** → **Azure virtual machine**

2. **Basics tab:**

   | Field | Value |
   |---|---|
   | Resource group | `helpdesk-lab-rg` |
   | VM name | `DC01` |
   | Region | Same as VNet |
   | Image | **Windows Server 2022 Datacenter** |
   | Size | Click **See all sizes** → search `B1s` → select **Standard_B1s** |
   | Username | `azureuser` |
   | Password | Set a strong password — write it down |
   | Inbound ports | **RDP (3389)** — we'll restrict this further with NSG |

3. **Disks tab:**
   - OS disk type: **Standard HDD** (cheapest, fine for lab)

4. **Networking tab:**
   - Virtual network: `helpdesk-vnet`
   - Subnet: `default`
   - Public IP: create new, name it `DC01-pip`
   - NIC NSG: **Basic** for now (we'll set Advanced rules next)

5. Click **Review + create** → **Create**

> ⚠️ Standard_B1s (1 vCPU, 1 GB RAM) is free for 750 hours/month for 12 months. Do NOT select any other size.

---

## Launch CLIENT01

1. Repeat the same steps with these differences:

   | Field | Value |
   |---|---|
   | VM name | `CLIENT01` |
   | Image | **Windows 10 Pro** (search "Windows 10" in the image selector) |
   | Size | Standard_B1s |
   | Public IP | `CLIENT01-pip` |

---

## Configure NSG Rules

Now restrict exactly what traffic can reach DC01. This is the **least-privilege** security model.

1. Search → **Network security groups** → find the NSG attached to DC01 (named something like `DC01-nsg`)
2. Left sidebar → **Inbound security rules** → **Add** for each rule below:

### DC01 NSG — Inbound Rules

| Priority | Name | Port | Protocol | Source | Action | Purpose |
|---|---|---|---|---|---|---|
| 100 | Allow-RDP | 3389 | TCP | **Your home IP** | Allow | Remote desktop — your IP only |
| 110 | Allow-DNS-TCP | 53 | TCP | 10.0.0.0/16 | Allow | AD DNS over TCP |
| 120 | Allow-DNS-UDP | 53 | UDP | 10.0.0.0/16 | Allow | AD DNS over UDP |
| 130 | Allow-Kerberos-TCP | 88 | TCP | 10.0.0.0/16 | Allow | Authentication |
| 140 | Allow-Kerberos-UDP | 88 | UDP | 10.0.0.0/16 | Allow | Authentication |
| 150 | Allow-LDAP | 389 | TCP | 10.0.0.0/16 | Allow | AD queries |
| 160 | Allow-SMB | 445 | TCP | 10.0.0.0/16 | Allow | Group Policy |
| 170 | Allow-LDAPS | 636 | TCP | 10.0.0.0/16 | Allow | Secure LDAP |
| 180 | Allow-RPC | 135 | TCP | 10.0.0.0/16 | Allow | Remote Procedure Call |
| 190 | Allow-ICMP | * | ICMP | 10.0.0.0/16 | Allow | Ping / connectivity test |
| 4096 | Deny-All-Inbound | * | * | * | **Deny** | Block everything else |

> **What is 10.0.0.0/16?** This is your VNet address range — meaning only other VMs inside your Azure network can use those ports. Not the open internet.

### CLIENT01 NSG — Inbound Rules

CLIENT01 only needs RDP from your IP:

| Priority | Name | Port | Source | Action |
|---|---|---|---|---|
| 100 | Allow-RDP | 3389 | Your IP | Allow |
| 4096 | Deny-All | * | * | Deny |

---

## Connect via RDP

### Find the public IP
Azure portal → **Virtual machines** → click **DC01** → copy the **Public IP address**

### Connect
- **Windows:** Start → Remote Desktop Connection → enter the public IP → username: `azureuser`
- **Mac:** Microsoft Remote Desktop app → Add PC → enter IP → username: `azureuser`

> ℹ️ The public IP **changes every time you stop and start** the VM unless you assign a static public IP. Always check the portal for the current IP.

---

## Stop VMs

> ✅ Stop both VMs at the end of every session to preserve your free hours.

Azure portal → **Virtual machines** → click the VM → **Stop**

Wait for the status to show **Stopped (deallocated)** — "Stopped" alone still charges you. It must say **deallocated**.

**Starting again:** VM → **Start** → wait ~2 minutes → check new public IP → RDP in.

---

## Troubleshooting

| Problem | Solution |
|---|---|
| RDP connection refused | Your IP changed — update the NSG RDP rule: Source → My IP |
| VM status shows "Stopped" but still charging | Must be "Stopped (deallocated)" — click Stop again from the portal |
| Can't find Standard_B1s size | Filter by: Free services eligible, or search `B1s` in the size selector |
| VM creation fails — quota error | Azure free accounts have vCPU quotas per region — try a different region |

---

**Next:** [Phase 2 — Active Directory Setup →](02-active-directory.md)
