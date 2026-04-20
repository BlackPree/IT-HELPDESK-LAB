# SOP-HD-003 — New User Setup

**ID:** HD-003 | **Version:** 1.0 | **Tier:** L2 Sysadmin | **SLA:** 1 business day (SEV-B)

---

## Overview

Create Active Directory accounts for new employees following a verified manager request.

---

## Steps

**1.** Confirm request came from an authorised manager. Add internal note:
```
Request validated — submitted by manager [name] at [time].
```

**2.** Confirm all details present: name, department, title, manager, start date.

**3.** RDP into DC01 → PowerShell 7 as Administrator.

**4a.** Single user:
```powershell
cd C:\Scripts
.\New-HelpdeskUser.ps1 -First "[First]" -Last "[Last]" -Dept "[Dept]" -Title "[Title]"
.\New-HelpdeskUser.ps1 ... (set manager if provided)
```

**4b.** Multiple users — use bulk script:
```powershell
.\New-BulkADUsers.ps1 -CsvPath "C:\Scripts\[filename].csv"
```

**5.** Verify accounts created:
```powershell
Get-ADUser -Filter * -SearchBase "OU=[Dept],DC=helpdesk,DC=local" | Select-Object Name, Enabled
```

**6.** Reply to manager with username(s) and temp password(s). Set ticket to **Resolved**.

---

## Reply Template
```
Hi [Manager Name],

Account(s) created and ready for [start date].

  Username   : [username]
  UPN        : [username]@helpdesk.local
  Temp PW    : Welcome@123!
  Must change password on first login: Yes

IT Helpdesk
```

---

---
