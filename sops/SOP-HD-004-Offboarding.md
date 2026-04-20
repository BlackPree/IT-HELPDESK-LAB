# SOP-HD-004 — Employee Offboarding

**ID:** HD-004 | **Version:** 1.0 | **Tier:** L2 Sysadmin | **SLA:** 4 hours (SEV-B)

---

## Overview

Disable and archive an AD account when an employee leaves. Ship a security log snapshot to S3 for audit.

---

## Steps

**1.** Confirm last working day with HR or manager before acting. Add internal note:
```
Termination confirmed by [name] — last day: [date].
```

**2.** RDP into DC01 → PowerShell 7 as Administrator.

**3.** Run offboarding:
```powershell
cd C:\Scripts
.\Remove-HelpdeskUser.ps1 -Username "[username]" -TicketID "[ticket_number]"
```

**4.** Ship security logs to S3 as audit snapshot:
```powershell
.\Ship-LogsToS3.ps1 -BucketName "helpdesk-logs-munashe" -LogType "Security"
```

**5.** Verify account in `Disabled_Users` OU:
```powershell
Get-ADUser -Filter {SamAccountName -eq "[username]"} `
    -SearchBase "OU=Disabled_Users,DC=helpdesk,DC=local" |
    Select-Object Name, Enabled, DistinguishedName
```

**6.** Paste all output into osTicket as internal note. Set ticket to **Resolved**.

**7.** Schedule permanent deletion: 30 days from today.

---

## Permanent Deletion (After 30 Days)

```powershell
.\Remove-HelpdeskUser.ps1 -Username "[username]" -Delete -TicketID "[original_ticket]"
# Type DELETE when prompted
```

---

## Ticket Note Template
```
SOP: HD-004 | Agent: Munashe | Date: [datetime]
─────────────────────────────────────────────
Account disabled: Yes
Groups removed: [n]
Archived: Disabled_Users OU
Security log shipped to S3: Yes — [S3 path]
Confirmed by: [HR / manager name]
30-day deletion scheduled: [date]
SLA: [Met / Breached]
```
