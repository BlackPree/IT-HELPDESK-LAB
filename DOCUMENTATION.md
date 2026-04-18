# Cloud-Native Hybrid Helpdesk Ecosystem
## Full Project Documentation

**Author:** Munashe B Nyarwendo 
**LinkedIn:**(https://www.linkedin.com/in/munashenyarwendo/))

---

## What Is This Project?

**Cloud-Native Hybrid Helpdesk Ecosystem** is an enterprise-grade IT support environment built entirely on free cloud resources across Azure and AWS.

It simulates the infrastructure of a real company IT department — with a Windows domain, automated user management, secure remote access, centralised logging, and a full ticketing system.

This project was built to demonstrate skills directly relevant to **Cloud Engineer**, **Systems Administrator**, **IT Support L2/L3**, and **Junior DevOps** roles.

---

## Why Hybrid Cloud?

Most home labs use a single cloud provider. This project deliberately bridges **Azure and AWS** because:

- Real enterprises almost never use just one cloud — hybrid and multi-cloud are the norm
- Azure is dominant for identity (Active Directory, Entra ID) — it's what most companies use for users and access
- AWS is dominant for infrastructure and storage — S3, EC2, and CloudWatch are industry standard
- Building on both demonstrates you can work in real enterprise environments, not just follow a single tutorial

---

## Architecture Decisions

| Decision | Why |
|---|---|
| Azure for Domain Controller | Azure integrates natively with Microsoft products — AD DS runs best on Azure VMs |
| AWS for Jump Box and Storage | Demonstrates cross-cloud access patterns; S3 is the industry standard for log archival |
| osTicket over Zendesk | Open source, self-hosted, no trial expiry — more impressive on a portfolio |
| Linux EC2 jump box | Shows Linux comfort; SSH-based access is standard in enterprise environments |
| PowerShell 7 | Cross-platform PS7 works on Windows and Linux — more versatile than PS5 |
| Least-privilege NSG/SG rules | Security best practice — only the exact ports needed, nothing more |

---

## Prerequisites

| Requirement | Details |
|---|---|
| Azure free account | Sign up at portal.azure.com — includes $200 credit + 12-month free services |
| AWS free account | Sign up at aws.amazon.com — 12-month free tier |
| SSH client | Windows: built-in OpenSSH or PuTTY. Mac/Linux: built-in terminal |
| RDP client | Windows: built-in. Mac: Microsoft Remote Desktop (free, App Store) |
| Basic PowerShell knowledge | Helpful but not required — all commands are provided |
| ~4 weeks part-time | ~1–2 hours per day across 7 phases |

---

## Installation — All 7 Phases

---

### Phase 1 — Azure Setup

**Full guide:** [docs/01-azure-setup.md](docs/01-azure-setup.md)  
**Time:** ~2 hours

#### What you will build
- Azure free account with billing alerts configured
- Resource Group: `helpdesk-lab-rg`
- Virtual Network: `helpdesk-vnet` (10.0.0.0/16)
- DC01: Windows Server 2022 VM (Standard_B1s — free tier)
- CLIENT01: Windows 10 VM (Standard_B1s — free tier)
- Network Security Group with least-privilege rules

#### Free tier VM specs

| VM | Size | vCPU | RAM | Cost |
|---|---|---|---|---|
| DC01 | Standard_B1s | 1 | 1 GB | Free (750 hrs/month) |
| CLIENT01 | Standard_B1s | 1 | 1 GB | Free (750 hrs/month) |

#### Key Azure NSG inbound rules for DC01

| Port | Protocol | Source | Purpose |
|---|---|---|---|
| 3389 | TCP | Your IP only | RDP access |
| 53 | TCP/UDP | VNet | AD DNS |
| 88 | TCP/UDP | VNet | Kerberos authentication |
| 389 | TCP | VNet | LDAP |
| 445 | TCP | VNet | SMB / Group Policy |
| 636 | TCP | VNet | LDAPS (secure) |
| 135 | TCP | VNet | RPC |

> Stop both VMs at the end of every session: VM → Overview → **Stop**

---

### Phase 2 — Active Directory Setup

**Full guide:** [docs/02-active-directory.md](docs/02-active-directory.md)  
**Time:** ~3 hours

#### What you will build
- Static private IP on DC01
- AD DS role installed and DC promoted to `helpdesk.local`
- OU structure for the organisation
- 10 manual test users + bulk import via PowerShell
- Group Policy: password policy, account lockout
- CLIENT01 joined to the domain

#### OU structure

```
helpdesk.local
├── IT
├── HR
├── Finance
├── Helpdesk
└── Disabled_Users
```

#### After promotion, login changes to
```
Username: HELPDESK\AzureUser  (or whatever your VM admin username is)
Password: same as before
```

---

### Phase 3 — AWS Setup

**Full guide:** [docs/03-aws-setup.md](docs/03-aws-setup.md)  
**Time:** ~2 hours

#### What you will build
- AWS free account with billing alarms ($1 and $5 CloudWatch alarms)
- EC2 Linux instance (t2.micro) — JUMP-01
- S3 bucket: `helpdesk-logs-[yourname]`
- IAM role for EC2 to write to S3
- Security Group: SSH only from your IP
- osTicket installed on JUMP-01

#### Why a Linux jump box?
A jump box (also called a bastion host) is the single, hardened entry point into a network. Only one server is exposed to the internet (the jump box) — everything else is only reachable from inside the network. This is standard security architecture in enterprise environments.

#### Connect to JUMP-01
```bash
# From your terminal (Mac/Linux)
ssh -i helpdesk-key.pem ec2-user@[PUBLIC_IP]

# From Windows (PowerShell or Command Prompt)
ssh -i helpdesk-key.pem ec2-user@[PUBLIC_IP]
```

---

### Phase 4 — osTicket Setup

**Full guide:** [docs/04-osticket-setup.md](docs/04-osticket-setup.md)  
**Time:** ~2 hours

#### What you will build
- osTicket installed on the Linux EC2 jump box (LAMP stack)
- 4 help topics (ticket categories)
- SLA plans
- Agent accounts
- Email notifications configured

#### Why osTicket over Zendesk?
- Open source — no trial expiry, runs forever
- Self-hosted on your own EC2 — shows infrastructure knowledge
- Used in real SME and enterprise environments
- Configuring it from scratch is a much stronger portfolio piece

#### Help topics to create

| Help Topic | SLA | Assigned Team |
|---|---|---|
| Password Reset | 2 hours | L1 Support |
| Account Unlock | 2 hours | L1 Support |
| New User Request | 1 business day | L2 Sysadmin |
| Account Termination | 4 hours | L2 Sysadmin |

---

### Phase 5 — PowerShell Scripts

**Full guide:** [docs/05-powershell-scripts.md](docs/05-powershell-scripts.md)  
**Time:** ~3 hours

#### What you will build
- 5 PowerShell scripts saved to `C:\Scripts\` on DC01
- Bulk user import from CSV
- Single user management scripts
- Cross-cloud log shipping script

#### Run all scripts from DC01
```powershell
# Open PowerShell 7 as Administrator on DC01
cd C:\Scripts

# Bulk create 50 test users from CSV
.\New-BulkADUsers.ps1 -CsvPath "C:\Scripts\users.csv"

# Create a single user
.\New-HelpdeskUser.ps1 -First "Jane" -Last "Doe" -Dept "Finance" -Title "Finance Analyst"

# Password reset (linked to osTicket)
.\Reset-HelpdeskPassword.ps1 -Username "jdoe" -TicketID "1042"

# Offboard a user
.\Remove-HelpdeskUser.ps1 -Username "jdoe" -TicketID "2089"

# Ship today's logs to S3
.\Ship-LogsToS3.ps1 -BucketName "helpdesk-logs-munashe" -LogType "Security"
```

#### Sample CSV format for bulk user creation
```csv
FirstName,LastName,Department,Title
Jane,Doe,Finance,Finance Analyst
Tom,Smith,IT,IT Support L1
Sarah,Johnson,HR,HR Coordinator
Mark,Taylor,HR,HR Manager
Emma,Brown,Finance,Finance Coordinator
```

---

### Phase 6 — Cross-Cloud Logging

**Full guide:** [docs/06-cross-cloud-logging.md](docs/06-cross-cloud-logging.md)  
**Time:** ~2 hours

#### What you will build
- AWS IAM user with write-only S3 permissions
- AWS CLI configured on DC01 (Azure VM)
- PowerShell script that exports Windows Event Logs and ships them to S3
- S3 bucket policy locking down access

#### Why this matters
Cross-cloud log aggregation is a real enterprise requirement. Security teams need logs from every system in one place — regardless of which cloud they're on. This demonstrates:
- Understanding of cloud IAM and access policies
- Log management fundamentals
- Cross-cloud integration skills

#### Log shipping flow
```
DC01 (Azure VM)
  → Windows Event Log (Security, System, Application)
  → PowerShell exports to .evtx / .json
  → AWS CLI uploads to S3 bucket
  → S3: helpdesk-logs-munashe/YYYY-MM-DD/
```

---

### Phase 7 — Helpdesk Scenarios

**Full guide:** [docs/07-scenarios.md](docs/07-scenarios.md)  
**Time:** ~2 hours

#### 4 end-to-end simulations

| # | Scenario | Scripts used | Ticket flow |
|---|---|---|---|
| 1 | Password Reset | `Reset-HelpdeskPassword.ps1` | osTicket → PowerShell → Close ticket |
| 2 | New Employee Onboarding | `New-HelpdeskUser.ps1` | osTicket → PowerShell → Notify manager |
| 3 | Employee Offboarding | `Remove-HelpdeskUser.ps1` | osTicket → PowerShell → Log to S3 |
| 4 | Bulk Department Onboarding | `New-BulkADUsers.ps1` | CSV → 50 users created → osTicket resolved |

---

## What to Include in Your GitHub Repository

### ✅ Always include

| File / Folder | Why |
|---|---|
| `README.md` | First thing recruiters see — your project's front page |
| `DOCUMENTATION.md` | Explains the project name, purpose, and full setup |
| `docs/*.md` | Step-by-step guides prove you understand every phase |
| `scripts/*.ps1` | Actual code — the most impressive part of the portfolio |
| `configs/nsg-rules.md` | Shows security awareness and network knowledge |
| `configs/aws-iam-policy.json` | Demonstrates IAM and least-privilege understanding |
| `sops/*.md` | Shows you think like a real IT professional |
| `assets/screenshots` | Evidence the lab is real and working |

### ❌ Never include

| File / Type | Why not |
|---|---|
| `.pem` key files | Your private SSH key — anyone who gets this owns your server |
| `id_rsa` / `id_ed25519` | Same — private SSH keys, never commit these |
| Any file with passwords | Hard-coded passwords in scripts or config files |
| `.env` files | Often contain secrets, API keys, connection strings |
| AWS Access Key ID / Secret | Anyone who gets these can rack up charges on your account |
| Azure credentials or service principal secrets | Same risk as AWS keys |
| `terraform.tfstate` | Can contain sensitive infrastructure details |
| Personal info beyond your name | Phone numbers, home address, personal email |
| Large `.evtx` log files | Binary files, huge, serve no purpose in a repo |
| VM disk images or snapshots | Far too large, never put these anywhere near GitHub |

### ⚠️ Be careful with

| File | What to do |
|---|---|
| `configs/osticket-config.md` | Remove any database passwords before committing |
| Scripts with hardcoded IPs | Replace real IPs with `[YOUR_IP]` placeholders |
| Scripts with temp passwords | Document them as examples — make clear they should be changed |
| `users.csv` | If using real names, replace with fake test data before uploading |

---

## How to Keep Secrets Out of GitHub

### Use a `.gitignore` file

Create a file called `.gitignore` in the root of your repo with these contents:

```
# SSH and cloud credentials
*.pem
*.ppk
id_rsa
id_ed25519
.env
credentials

# AWS
.aws/credentials
aws-credentials.json

# Azure
azure-credentials.json
*.publishsettings

# Logs and large files
*.evtx
*.vhd
*.vmdk

# OS files
.DS_Store
Thumbs.db
desktop.ini

# Terraform
*.tfstate
*.tfstate.backup
.terraform/
```

Any file listed here will never be accidentally committed, even if you drag and drop your whole project folder.

---

## Skills This Project Demonstrates

### Cloud Infrastructure
- Provisioning VMs on Azure (Resource Groups, VNets, NSGs)
- Provisioning EC2 instances on AWS (Security Groups, IAM roles)
- Cross-cloud architecture design
- Free tier cost management on both platforms

### Security
- Least-privilege network rules (NSG and Security Groups)
- Jump box / bastion host architecture
- IAM policies with minimum required permissions
- Secrets management (what not to commit)

### Identity & Access Management
- Active Directory domain setup from scratch
- OU hierarchy and user lifecycle management
- Group Policy configuration
- Bulk user provisioning at scale

### Scripting & Automation
- PowerShell 7 with the ActiveDirectory module
- CSV-driven bulk operations
- Cross-cloud scripting (Azure VM writing to AWS S3)
- Linux bash basics (jump box management)

### ITSM
- osTicket installation and configuration on Linux
- Help topics, SLA plans, agent teams
- End-to-end ticket lifecycle

### Documentation
- Technical SOPs and runbooks
- GitHub-hosted project documentation
- Architecture diagrams

---

*Built by Munashe — hybrid cloud helpdesk lab demonstrating enterprise-grade infrastructure, security, and automation skills.*
