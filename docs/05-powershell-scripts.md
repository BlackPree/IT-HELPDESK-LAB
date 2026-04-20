# Phase 5 — PowerShell Scripts

**Goal:** Deploy and test five PowerShell 7 scripts on DC01 — including bulk user creation and cross-cloud log shipping.

**Time:** ~3 hours | **Prerequisites:** AD working, AWS CLI configured on DC01

---

## Table of Contents
- [Setup](#setup)
- [Script 1 — New-BulkADUsers](#script-1--new-bulkadusers)
- [Script 2 — New-HelpdeskUser](#script-2--new-helpdeskuser)
- [Script 3 — Remove-HelpdeskUser](#script-3--remove-helpdeskuser)
- [Script 4 — Reset-HelpdeskPassword](#script-4--reset-helpdeskpassword)
- [Script 5 — Ship-LogsToS3](#script-5--ship-logsto-s3)
- [Full Test Sequence](#full-test-sequence)
- [Troubleshooting](#troubleshooting)

---

## Setup

All scripts run on **DC01** (the Azure VM). Open PowerShell 7 as Administrator.

### Install PowerShell 7 on DC01

PowerShell 5.1 ships with Windows Server 2022 but PowerShell 7 is cross-platform and more powerful:

```powershell
# Download and install PowerShell 7
Invoke-WebRequest -Uri "https://github.com/PowerShell/PowerShell/releases/download/v7.4.1/PowerShell-7.4.1-win-x64.msi" -OutFile "$env:TEMP\PS7.msi"
Start-Process msiexec.exe -Args "/i $env:TEMP\PS7.msi /quiet ADD_EXPLORER_CONTEXT_MENU_OPENPOWERSHELL=1" -Wait

# Launch PowerShell 7 from Start menu: "pwsh"
```

### Create scripts folder

```powershell
New-Item -ItemType Directory -Path "C:\Scripts" -Force

# Verify AD module is available
Get-Module -Name ActiveDirectory -ListAvailable

# Install if needed
Install-WindowsFeature RSAT-AD-PowerShell
```

---

## Script 1 — New-BulkADUsers

**File:** `C:\Scripts\New-BulkADUsers.ps1`  
**Purpose:** Read a CSV file and create hundreds of AD users instantly.

### CSV format

Create `C:\Scripts\users.csv`:
```csv
FirstName,LastName,Department,Title
Jane,Doe,Finance,Finance Analyst
Tom,Smith,IT,IT Support L1
Sarah,Johnson,HR,HR Coordinator
Mark,Taylor,HR,HR Manager
Emma,Brown,Finance,Finance Coordinator
David,Clark,IT,Systems Admin
Lisa,White,Helpdesk,Helpdesk Agent
James,Hall,Helpdesk,Helpdesk Agent
Anna,Green,Finance,Finance Manager
Paul,Harris,IT,IT Support L2
```

Add as many rows as you want — the script handles all of them.

### Usage

```powershell
cd C:\Scripts
.\New-BulkADUsers.ps1 -CsvPath "C:\Scripts\users.csv"

# Create with a different temp password
.\New-BulkADUsers.ps1 -CsvPath "C:\Scripts\users.csv" -TempPassword "BulkPass@2025!"
```

### What it does
1. Reads every row from the CSV
2. Auto-generates username: first initial + last name (e.g. `jdoe`)
3. Skips any username that already exists (no duplicates)
4. Creates the user in the correct OU based on the Department column
5. Prints a summary: X created, Y skipped

---

## Script 2 — New-HelpdeskUser

**File:** `C:\Scripts\New-HelpdeskUser.ps1`  
**Purpose:** Create a single AD user from a ticket request.

### Usage

```powershell
.\New-HelpdeskUser.ps1 -First "Jane" -Last "Doe" -Dept "Finance" -Title "Finance Analyst"

.\New-HelpdeskUser.ps1 -First "Tom" -Last "Smith" -Dept "IT" -Title "IT Support L1"
```

### What it does
1. Generates username (e.g. `jdoe`)
2. Checks for duplicates
3. Creates user in correct OU with temp password `Welcome@123!`
4. Forces password change on first login
5. Prints full account summary

---

## Script 3 — Remove-HelpdeskUser

**File:** `C:\Scripts\Remove-HelpdeskUser.ps1`  
**Purpose:** Offboard a user — disable, strip access, archive.

### Usage

```powershell
# Standard offboarding
.\Remove-HelpdeskUser.ps1 -Username "jdoe" -TicketID "2089"

# Permanent deletion (after 30-day hold)
.\Remove-HelpdeskUser.ps1 -Username "jdoe" -Delete -TicketID "2089"
```

### What it does
1. Disables the account
2. Removes all group memberships
3. Moves account to `Disabled_Users` OU
4. If `-Delete`: asks for typed `DELETE` confirmation then permanently removes

---

## Script 4 — Reset-HelpdeskPassword

**File:** `C:\Scripts\Reset-HelpdeskPassword.ps1`  
**Purpose:** Reset a user's password from an osTicket request.

### Usage

```powershell
.\Reset-HelpdeskPassword.ps1 -Username "jdoe" -TicketID "1042"
```

### What it does
1. Checks account is enabled
2. Resets password to `TempPass@1!`
3. Sets must-change-at-logon
4. Unlocks the account
5. Outputs a pre-formatted osTicket note to paste directly into the ticket

---

## Script 5 — Ship-LogsToS3

**File:** `C:\Scripts\Ship-LogsToS3.ps1`  
**Purpose:** Export Windows Event Logs from DC01 and upload them to AWS S3.

### Prerequisites
- AWS CLI installed and configured on DC01 (`aws configure` done with `dc01-log-shipper` credentials)
- S3 bucket created: `helpdesk-logs-[yourname]`

### Usage

```powershell
# Ship Security logs (login events, policy changes)
.\Ship-LogsToS3.ps1 -BucketName "helpdesk-logs-munashe" -LogType "Security"

# Ship System logs (service starts/stops, errors)
.\Ship-LogsToS3.ps1 -BucketName "helpdesk-logs-munashe" -LogType "System"

# Ship all log types
.\Ship-LogsToS3.ps1 -BucketName "helpdesk-logs-munashe" -LogType "All"
```

### What it does
1. Exports the specified Windows Event Log to a JSON file
2. Adds a timestamp to the filename: `Security-2025-01-15-1430.json`
3. Uploads the file to S3 under the correct folder: `s3://helpdesk-logs-munashe/security-logs/`
4. Deletes the local copy after successful upload
5. Prints confirmation with the S3 path

### Verify logs shipped
```powershell
# Check what's in your S3 bucket from DC01
aws s3 ls s3://helpdesk-logs-munashe/ --recursive
```

---

## Full Test Sequence

Run these in order to verify everything works:

```powershell
cd C:\Scripts

# 1. Bulk create test users from CSV
.\New-BulkADUsers.ps1 -CsvPath "C:\Scripts\users.csv"

# 2. Verify users were created
Get-ADUser -Filter * -SearchBase "OU=Finance,DC=helpdesk,DC=local" | Select-Object Name, Enabled

# 3. Create one more user manually
.\New-HelpdeskUser.ps1 -First "Test" -Last "User" -Dept "IT" -Title "Test Account"

# 4. Reset their password
.\Reset-HelpdeskPassword.ps1 -Username "tuser" -TicketID "TEST-001"

# 5. Offboard the test user
.\Remove-HelpdeskUser.ps1 -Username "tuser" -TicketID "TEST-002"

# 6. Verify in Disabled_Users OU
Get-ADUser -Filter * -SearchBase "OU=Disabled_Users,DC=helpdesk,DC=local" | Select-Object Name, Enabled

# 7. Ship today's security logs to S3
.\Ship-LogsToS3.ps1 -BucketName "helpdesk-logs-munashe" -LogType "Security"

# 8. Verify in S3
aws s3 ls s3://helpdesk-logs-munashe/security-logs/
```

---

## Troubleshooting

| Error | Cause | Fix |
|---|---|---|
| `Module not found: ActiveDirectory` | RSAT not installed | `Install-WindowsFeature RSAT-AD-PowerShell` then reopen PS |
| `Access denied` | Not running as Administrator | Right-click PowerShell → Run as Administrator |
| `Cannot find identity` | Wrong username | Check SamAccountName in AD Users and Computers |
| `aws: command not found` | AWS CLI not installed / not in PATH | Reinstall AWS CLI, then close and reopen PowerShell |
| S3 upload: `Access Denied` | Wrong IAM credentials | Run `aws configure` again with dc01-log-shipper keys |
| Bulk script: all users skipped | Users already exist from a previous run | Delete test users first or use a different CSV |

---

**Next:** [Phase 6 — Cross-Cloud Logging →](06-cross-cloud-logging.md)
