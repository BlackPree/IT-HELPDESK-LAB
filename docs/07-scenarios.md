# Phase 7 — Helpdesk Scenarios

**Goal:** Complete four end-to-end helpdesk simulations connecting osTicket tickets to Active Directory actions.

**Time:** ~2 hours | **Prerequisites:** All previous phases complete

---

## How Each Scenario Works

```
User submits ticket in osTicket
        ↓
Agent verifies identity / request
        ↓
Agent RDPs into DC01 and runs PowerShell
        ↓
Agent pastes output into osTicket as internal note
        ↓
Agent replies to user and closes ticket
```

---

## Scenario 1 — Password Reset

**SOP:** [SOP-HD-001](../sops/SOP-HD-001-Password-Reset.md)  
**Script:** `Reset-HelpdeskPassword.ps1`  
**SLA:** 2 hours

### Ticket (submit as end user in osTicket portal)

```
Help Topic : Password Reset
Name       : John Smith
Email      : jsmith@test.com
Subject    : Cannot log in — please reset my password
Message    : I entered my password this morning and my account is now locked.
             My username is jsmith.
```

### Agent steps

```powershell
# On DC01 — PowerShell as Administrator
cd C:\Scripts
.\Reset-HelpdeskPassword.ps1 -Username "jsmith" -TicketID "[osTicket number]"
```

Paste the script output into the ticket as an **internal note**.

Reply to user:
```
Hi John,

Your password has been reset. Your temporary password is: TempPass@1!

Please log in — you will be prompted to set a new password immediately.

IT Helpdesk
```

Set ticket status → **Resolved**.

---

## Scenario 2 — Bulk New Employees (IT Strength Demo)

**SOP:** [SOP-HD-003](../sops/SOP-HD-003-New-User.md)  
**Script:** `New-BulkADUsers.ps1`  
**SLA:** 1 business day

This scenario is unique to this project — demonstrates PowerShell at scale.

### Ticket

```
Help Topic : New User Request
Subject    : 5 new starters joining Finance on Monday
Message    : Please create accounts for the following:
             1. Rachel Green — Finance Analyst
             2. Joey Tribbiani — Finance Coordinator  
             3. Monica Geller — Finance Manager
             4. Chandler Bing — Finance Analyst
             5. Ross Geller — Finance Director
```

### Agent steps

Create `C:\Scripts\new-starters.csv`:
```csv
FirstName,LastName,Department,Title
Rachel,Green,Finance,Finance Analyst
Joey,Tribbiani,Finance,Finance Coordinator
Monica,Geller,Finance,Finance Manager
Chandler,Bing,Finance,Finance Analyst
Ross,Geller,Finance,Finance Director
```

```powershell
cd C:\Scripts
.\New-BulkADUsers.ps1 -CsvPath "C:\Scripts\new-starters.csv"
```

Paste output (5 accounts created) into ticket as internal note.

Reply to manager with all 5 usernames and temp passwords. Resolve ticket.

---

## Scenario 3 — Employee Offboarding

**SOP:** [SOP-HD-004](../sops/SOP-HD-004-Offboarding.md)  
**Script:** `Remove-HelpdeskUser.ps1` + `Ship-LogsToS3.ps1`  
**SLA:** 4 hours

### Ticket

```
Help Topic : Account Termination
Subject    : Account closure — David Clark (dclark)
Message    : David Clark's last day is today. Please disable his account
             immediately and ensure all access is revoked.
```

### Agent steps

```powershell
cd C:\Scripts

# Offboard the user
.\Remove-HelpdeskUser.ps1 -Username "dclark" -TicketID "[ticket number]"

# Ship a security log snapshot to S3 as evidence
.\Ship-LogsToS3.ps1 -BucketName "helpdesk-logs-munashe" -LogType "Security"

# Note the S3 path in the ticket for audit trail
aws s3 ls s3://helpdesk-logs-munashe/security-logs/ | Select-String "$(Get-Date -Format yyyy-MM-dd)"
```

Paste all output into the ticket. Resolve.

---

## Scenario 4 — New Domain Machine (CLIENT01)

**Goal:** Walk through joining CLIENT01 to the domain and logging in as a domain user.

1. On CLIENT01: join `helpdesk.local` (if not done in Phase 2)
2. Log in as `HELPDESK\jsmith` with the temp password
3. Windows prompts to change password — set a new one
4. Open Command Prompt → run `whoami` → should show `helpdesk\jsmith`
5. Run `gpresult /r` → shows Group Policy applied from DC01

Screenshot each step — this proves your domain is fully working.

---

## Project Complete Checklist

### Azure
- [ ] Resource Group `helpdesk-lab-rg` created
- [ ] VNet `helpdesk-vnet` created
- [ ] DC01 (Standard_B1s, Windows Server 2022) running
- [ ] CLIENT01 (Standard_B1s, Windows 10) running
- [ ] NSG rules: least-privilege on both VMs

### Active Directory
- [ ] DC01 promoted to Domain Controller (`helpdesk.local`)
- [ ] 5 OUs created (IT, HR, Finance, Helpdesk, Disabled_Users)
- [ ] 10+ manual test users created
- [ ] Group Policy: password and lockout policies applied
- [ ] CLIENT01 joined to domain

### AWS
- [ ] JUMP-01 (t2.micro, Amazon Linux) running
- [ ] Security Group: SSH only from your IP
- [ ] S3 bucket created with folder structure
- [ ] IAM policy `HelpdeskLogUploadPolicy` created (least-privilege)
- [ ] IAM user `dc01-log-shipper` created
- [ ] CloudWatch billing alarms: $1 and $5
- [ ] Zero-spend budget configured

### osTicket
- [ ] LAMP stack installed on JUMP-01
- [ ] osTicket installed and accessible
- [ ] Setup directory deleted
- [ ] 4 help topics created
- [ ] SLA plans: SEV-A, SEV-B, SEV-C
- [ ] Agent accounts created

### PowerShell
- [ ] PowerShell 7 installed on DC01
- [ ] `C:\Scripts\` folder created
- [ ] All 5 scripts deployed and tested
- [ ] Bulk users created from CSV
- [ ] AWS CLI configured on DC01
- [ ] `Ship-LogsToS3.ps1` tested — files visible in S3

### Scenarios
- [ ] Scenario 1: Password reset — ticket opened, PS run, ticket resolved
- [ ] Scenario 2: Bulk onboarding — 5 users created from CSV, ticket resolved
- [ ] Scenario 3: Offboarding — account disabled, logs shipped to S3, ticket resolved
- [ ] Scenario 4: CLIENT01 domain login confirmed

### Documentation & GitHub
- [ ] All screenshots in `assets/` folder
- [ ] Repo public on GitHub
- [ ] README pinned to GitHub profile
- [ ] Link added to LinkedIn

---

**Project complete.** 🎉
