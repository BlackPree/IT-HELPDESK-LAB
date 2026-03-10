# IT Helpdesk Home Lab
### Built by: Munashe 
### GitHub: [github.com/yourusername](https://github.com/Munex223)  
---

## What Is This Project?

**IT Helpdesk Home Lab** is a fully functional IT support environment built from scratch in the cloud.

It simulates the real-world systems and daily tasks of a **Level 1 / Level 2 IT Helpdesk** role — including managing user accounts in Active Directory, automating tasks with PowerShell, and handling support tickets through Zendesk.

Everything in this lab was built hands-on, documented step by step, and deployed on AWS using only the **Free Tier** — total cost: **$0**.

---

## Why I Built This

I built this project to demonstrate practical IT skills without requiring a job first. Rather than just studying theory, I wanted to:

- Set up real infrastructure in the cloud (AWS EC2)
- Configure a Domain Controller and Active Directory from scratch
- Write and test actual PowerShell scripts used by real sysadmins
- Process helpdesk tickets end-to-end using professional ticketing software
- Document every step the way a real IT team would

This project covers the core skills listed in most **Helpdesk**, **Sysadmin**, and **IT Support** job descriptions.

---

## Technologies Used

| Technology | Version / Type | What I Used It For |
|---|---|---|
| Amazon Web Services | EC2, Free Tier | Cloud hosting for both Windows servers |
| Windows Server 2022 | Standard | Operating system for DC01 and SRV01 |
| Active Directory Domain Services | Built into Windows Server | User accounts, OUs, Group Policy, domain management |
| PowerShell | 5.1 (built into Windows Server) | Automating user creation, modification, deletion, and password resets |
| Zendesk | Free Trial (14 days) | Ticketing system — forms, SLAs, macros, Knowledge Base |
| Remote Desktop Protocol (RDP) | Built into Windows | Connecting to cloud servers from my local machine |
| GitHub | — | Version control and documentation hosting |

---

## Project Structure

```
it-helpdesk-lab/
│
├── README.md                  ← Project overview
├── DOCUMENTATION.md           ← This file — full setup and install guide
│
├── docs/                      ← Step-by-step setup guides
│   ├── 01-aws-setup.md        ← AWS account, EC2 instances, Security Groups
│   ├── 02-active-directory.md ← Domain Controller, OUs, users, GPO
│   ├── 03-powershell-scripts.md ← All 4 PowerShell scripts explained
│   ├── 04-zendesk-setup.md    ← Zendesk configuration
│   └── 05-scenarios.md        ← 4 end-to-end helpdesk simulations
│
├── scripts/                   ← PowerShell automation scripts
│   ├── New-HelpdeskUser.ps1   ← Create a new AD user
│   ├── Set-HelpdeskUser.ps1   ← Modify an existing AD user
│   ├── Remove-HelpdeskUser.ps1 ← Offboard / disable a user
│   └── Reset-HelpdeskPassword.ps1 ← Reset password from a Zendesk ticket
│
├── sops/                      ← Standard Operating Procedures
│   ├── SOP-AD-001-Password-Reset.md
│   ├── SOP-AD-002-Account-Unlock.md
│   ├── SOP-AD-003-New-User.md
│   └── SOP-AD-004-Offboarding.md
│
└── assets/                    ← Screenshots from the completed lab
```

---

## Lab Architecture

```
┌────────────────────────────────────────────────────────┐
│                   AWS Cloud — Free Tier                 │
│                                                        │
│   ┌─────────────────────────┐   ┌───────────────────┐  │
│   │  DC01 (t2.micro)        │   │  SRV01 (t2.micro) │  │
│   │  Windows Server 2022    │   │  Windows Server   │  │
│   │                         │   │  2022             │  │
│   │  Roles installed:       │   │                   │  │
│   │  • AD Domain Services   │   │  Joined to:       │  │
│   │  • DNS Server           │   │  helpdesk.local   │  │
│   │                         │   │                   │  │
│   │  Domain: helpdesk.local │   │                   │  │
│   │  Scripts: C:\Scripts\   │   │                   │  │
│   └─────────────────────────┘   └───────────────────┘  │
│                                                        │
│   Region: us-east-1 (or your preferred region)         │
└────────────────────────────────────────────────────────┘
                          │
                          │ Tickets flow between
                          ▼
          ┌──────────────────────────────┐
          │  Zendesk — Cloud (Free Trial) │
          │                              │
          │  • 4 Ticket Forms            │
          │  • 2 SLA Policies            │
          │  • 3 Macros                  │
          │  • 2 Agent Groups            │
          │  • Knowledge Base (SOPs)     │
          └──────────────────────────────┘
```

### What each server does

**DC01 — Domain Controller**
The heart of the Windows network. Runs Active Directory Domain Services (AD DS) and DNS. All user accounts, passwords, and permissions are stored and managed here. The four PowerShell scripts run directly on this server.

**SRV01 — Member Server**
A second Windows Server joined to the `helpdesk.local` domain. Simulates a real company workstation or application server. Used to test domain logins and Group Policy application.

**Zendesk**
The cloud-based ticketing system. Users submit support requests here. The agent (me) processes them by running PowerShell on DC01, then updates and closes the ticket in Zendesk.

---

## Prerequisites

Before following the installation steps, make sure you have:

| Requirement | Details |
|---|---|
| AWS account | Free — sign up at aws.amazon.com. No charges if you follow this guide exactly. |
| Credit/debit card | Required by AWS for account verification. Not charged under Free Tier. |
| RDP client | Windows: built-in Remote Desktop Connection. Mac: Microsoft Remote Desktop (free, App Store). |
| Zendesk account | Free 14-day trial at zendesk.com — start this only at Phase 4, not before. |
| GitHub account | To host and share this documentation. |
| ~3–4 weeks | Part-time (~1–2 hours per day). Full-time would be faster. |

---

## Installation — Step by Step

Follow these five phases in order. Each phase builds on the previous one.

> ⚠️ Do not skip ahead. The phases are sequential — Active Directory must exist before you can run PowerShell scripts against it, and Zendesk is only useful once AD and scripts are working.

---

### Phase 1 — AWS Lab Setup

**Full guide:** [docs/01-aws-setup.md](docs/01-aws-setup.md)  
**Time:** ~2 hours

#### What you will do
- Create an AWS account and set up billing protection (billing alarms, zero-spend budget)
- Launch **DC01** — a Windows Server 2022 EC2 instance (t2.micro, 25 GB storage)
- Launch **SRV01** — a second Windows Server 2022 EC2 instance (t2.micro, 5 GB storage)
- Configure Security Group firewall rules for Active Directory traffic
- Connect to both servers via RDP

#### Key decisions
- **Instance type:** Always use `t2.micro` — it is the only Free Tier eligible Windows instance type
- **Storage:** DC01 = 25 GB, SRV01 = 5 GB — total must not exceed 30 GB (Free Tier limit)
- **Region:** Pick one region and use it for everything — recommended: `us-east-1`

#### Quick start commands (run inside DC01 after connecting)
```powershell
# Check Windows version
winver

# Check available disk space
Get-PSDrive C

# Check network adapter
Get-NetIPAddress -AddressFamily IPv4
```

---

### Phase 2 — Active Directory Setup

**Full guide:** [docs/02-active-directory.md](docs/02-active-directory.md)  
**Time:** ~3 hours

#### What you will do
- Set a static private IP on DC01 (required before promoting to Domain Controller)
- Install the **Active Directory Domain Services (AD DS)** role via Server Manager
- Promote DC01 to a **Domain Controller** — creating the `helpdesk.local` domain
- Create an OU (folder) structure for organising users
- Create 10 test user accounts manually
- Configure Group Policy — password complexity, lockout policy
- Join SRV01 to the `helpdesk.local` domain

#### Domain and OU structure
```
helpdesk.local
├── IT
├── HR
├── Finance
├── Helpdesk
└── Disabled_Users    ← Offboarded accounts go here
```

#### After promotion, log in as
```
Username: HELPDESK\Administrator
Password: (same as before)
```

> The `HELPDESK\` prefix is required after the server becomes a Domain Controller.

---

### Phase 3 — PowerShell Scripts

**Full guide:** [docs/03-powershell-scripts.md](docs/03-powershell-scripts.md)  
**Time:** ~3 hours

#### What you will do
- Create the `C:\Scripts\` folder on DC01
- Copy all four scripts from this repo into that folder
- Load the Active Directory PowerShell module
- Test each script with real AD users

#### Setup (run once on DC01)
```powershell
# Open PowerShell as Administrator, then:

# Create scripts folder
New-Item -ItemType Directory -Path "C:\Scripts" -Force

# Verify AD module is available
Get-Module -Name ActiveDirectory -ListAvailable

# If not found, install it
Install-WindowsFeature RSAT-AD-PowerShell
```

#### Copy scripts to DC01
The easiest way is to copy-paste the script contents from this repo directly into a new `.ps1` file on DC01 using Notepad or the PowerShell ISE editor.

Or use RDP clipboard:
1. Copy the script text from GitHub on your local machine
2. In your RDP session, open Notepad → paste → save as `C:\Scripts\New-HelpdeskUser.ps1`
3. Repeat for all four scripts

#### Scripts summary

| Script | Command | What it does |
|---|---|---|
| `New-HelpdeskUser.ps1` | `.\New-HelpdeskUser.ps1 -First "Jane" -Last "Doe" -Dept "Finance" -Title "Finance Analyst"` | Creates a new AD user in the specified OU |
| `Set-HelpdeskUser.ps1` | `.\Set-HelpdeskUser.ps1 -Username "jdoe" -NewTitle "Senior Analyst"` | Updates user attributes |
| `Remove-HelpdeskUser.ps1` | `.\Remove-HelpdeskUser.ps1 -Username "jdoe" -TicketID "2089"` | Disables, strips access, archives user |
| `Reset-HelpdeskPassword.ps1` | `.\Reset-HelpdeskPassword.ps1 -Username "jdoe" -TicketID "1042"` | Resets password and unlocks account |

---

### Phase 4 — Zendesk Setup

**Full guide:** [docs/04-zendesk-setup.md](docs/04-zendesk-setup.md)  
**Time:** ~2 hours

> ⏰ Start your Zendesk free trial NOW — not before this phase. The trial lasts 14 days.

#### What you will do
- Sign up for a Zendesk free trial at [zendesk.com](https://zendesk.com)
- Configure support email, business hours, branding
- Create **2 agent groups** — L1 Helpdesk and L2 Sysadmin
- Create **4 ticket forms** — Password Reset, Account Unlock, New User Setup, Account Termination
- Set **2 SLA policies** — Urgent (2 hrs) and Standard (1 business day)
- Create **3 macros** — pre-written response templates for common ticket types

#### Ticket forms created

| Form | Assigned to | SLA |
|---|---|---|
| Password Reset | L1 Helpdesk | 2 hours |
| Account Unlock | L1 Helpdesk | 2 hours |
| New User Setup | L2 Sysadmin | 1 business day |
| Account Termination | L2 Sysadmin | 4 hours |

---

### Phase 5 — Helpdesk Scenarios

**Full guide:** [docs/05-scenarios.md](docs/05-scenarios.md)  
**Time:** ~2 hours

#### What you will do
Run four complete helpdesk simulations — submitting a real Zendesk ticket, processing it with PowerShell on DC01, and closing the ticket with full documentation.

| # | Scenario | Script used |
|---|---|---|
| 1 | Password Reset — user locked out, ticket submitted, password reset, ticket closed | `Reset-HelpdeskPassword.ps1` |
| 2 | New Employee — manager submits request, account created, credentials sent | `New-HelpdeskUser.ps1` |
| 3 | Offboarding — HR submits request, account disabled and archived, ticket closed | `Remove-HelpdeskUser.ps1` |
| 4 | Knowledge Base — SOP written and published in Zendesk Guide | N/A |

---

## How to Run the Scripts

All scripts run on **DC01** from an **elevated PowerShell session**.

### Open PowerShell as Administrator
```
Start → search "PowerShell" → right-click Windows PowerShell → Run as Administrator
```

### Navigate to scripts folder
```powershell
cd C:\Scripts
```

### Run a script
```powershell
# Create a new user
.\New-HelpdeskUser.ps1 -First "Jane" -Last "Doe" -Dept "Finance" -Title "Finance Analyst"

# Modify a user
.\Set-HelpdeskUser.ps1 -Username "jdoe" -NewTitle "Senior Finance Analyst"

# Offboard a user
.\Remove-HelpdeskUser.ps1 -Username "jdoe" -TicketID "2089"

# Reset a password (from a Zendesk ticket)
.\Reset-HelpdeskPassword.ps1 -Username "jdoe" -TicketID "1042"
```

### Get built-in help for any script
```powershell
Get-Help .\New-HelpdeskUser.ps1 -Full
Get-Help .\Reset-HelpdeskPassword.ps1 -Examples
```

---

## Common Issues

| Problem | Likely cause | Fix |
|---|---|---|
| Can't RDP into an instance | Your home IP address changed | EC2 → Security Group → Edit inbound rules → RDP → Source → My IP |
| Login fails with just `Administrator` | Server is now a Domain Controller | Use `HELPDESK\Administrator` instead |
| PowerShell says "module not found" | AD module not installed | Run: `Install-WindowsFeature RSAT-AD-PowerShell` |
| Script says "Access Denied" | PowerShell not running as Administrator | Right-click PowerShell → Run as Administrator |
| SRV01 can't find `helpdesk.local` | DNS not pointing to DC01 | Set SRV01's Preferred DNS to DC01's private IP address |
| AWS charges appearing | Instance left running or EBS over 30 GB | Stop both instances; check Free Tier usage dashboard in Billing |
| Zendesk trial expired | Trial started too early | Zendesk offers a 30-day extension if you contact support |

---

## Cost

This entire lab was built for **$0** using the AWS Free Tier.

| Resource | Free allowance | Used in this project |
|---|---|---|
| EC2 t2.micro | 750 hours / month for 12 months | ~80–100 hours total |
| EBS Storage | 30 GB total | 30 GB (DC01: 25 GB, SRV01: 5 GB) |
| Data Transfer OUT | 1 GB / month | < 0.1 GB |
| Zendesk | 14-day free trial | Full trial used during Phase 4–5 |

**The two rules that keep the cost at $0:**
1. Always use instance type `t2.micro` — nothing larger
2. Stop both instances at the end of every session — don't leave them running overnight

---

## Skills This Project Covers

The following are directly demonstrated through building and completing this lab:

**Cloud & Infrastructure**
- Provisioning and configuring EC2 instances on AWS
- Managing Security Groups, VPCs, and networking rules
- Free Tier cost management and billing protection

**Windows Server Administration**
- Installing server roles via Server Manager
- Configuring static IP addresses and DNS
- Managing servers remotely via RDP

**Active Directory**
- Promoting a server to Domain Controller
- Creating and managing Organisational Units (OUs)
- Managing the full user lifecycle — create, modify, disable, delete
- Configuring Group Policy (password policy, account lockout)
- Domain join for member servers

**PowerShell**
- Writing scripts with parameters, validation, and error handling
- Using the `ActiveDirectory` module
- Automating repetitive AD administration tasks
- Writing script documentation with `Get-Help` support

**IT Ticketing & ITSM**
- Configuring a ticketing system from scratch (Zendesk)
- Creating ticket forms, SLA policies, and macros
- Following a ticket through its full lifecycle
- Writing and publishing SOPs in a Knowledge Base

**Documentation**
- Writing technical SOPs and runbooks
- Structuring project documentation on GitHub
- Using Markdown for professional documentation

---

## Folder: `sops/`

The `sops/` folder contains Standard Operating Procedures — the written guides that tell any helpdesk agent exactly how to complete a task consistently.

| SOP | Task covered |
|---|---|
| [SOP-AD-001](sops/SOP-AD-001-Password-Reset.md) | Password reset — identity verification, PowerShell steps, ticket template |
| [SOP-AD-002](sops/SOP-AD-002-Account-Unlock.md) | Account unlock — check lockout status, unlock, advise user on saved credentials |
| [SOP-AD-003](sops/SOP-AD-003-New-User.md) | New user setup — validate request, create account, assign manager, notify manager |
| [SOP-AD-004](sops/SOP-AD-004-Offboarding.md) | Offboarding — confirm last day, disable account, strip access, schedule deletion |

---

## Folder: `scripts/`

All scripts follow the same conventions:

- **Documented** — every script has a `.SYNOPSIS`, `.DESCRIPTION`, `.PARAMETER`, and `.EXAMPLE` block readable via `Get-Help`
- **Safe** — scripts validate inputs before acting. They check the user exists, the OU exists, and the account is in a valid state
- **Auditable** — every script accepts a `-TicketID` parameter so actions can be traced back to a Zendesk ticket
- **Coloured output** — green for success, yellow for warnings, red for errors, making it easy to confirm what happened at a glance

---

## How to Contribute or Adapt

Feel free to fork this repo and adapt it for your own lab. Some ideas:

- Add a bulk import script that reads a CSV and creates multiple users at once
- Add a `Get-HelpdeskUser.ps1` script that searches AD and returns a formatted user profile
- Extend the Zendesk setup with triggers and automations (auto-assign by keyword, auto-close after 72 hours)
- Add a second domain controller for AD replication practice
- Replace Zendesk with ServiceNow Developer Instance (free) for a more enterprise-grade ITSM tool

---

## Author

- GitHub: [github.com/yourusername](https://github.com/Munex223)
---

*Built as a self-directed IT Helpdesk portfolio project. All infrastructure, scripts, and documentation written from scratch.*
