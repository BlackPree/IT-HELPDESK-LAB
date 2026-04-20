# SOP-HD-002 — Account Unlock Procedure

**ID:** HD-002 | **Version:** 1.0 | **Tier:** L1 Support | **SLA:** 2 hours (SEV-A)

---

## Overview

Unlock an Active Directory account locked after too many failed login attempts (threshold: 5 per GPO).

---

## Steps

**1.** Verify identity. Add internal note to osTicket.

**2.** RDP into DC01 → PowerShell 7 as Administrator.

**3.** Check lock status:
```powershell
Get-ADUser -Identity "[username]" -Properties LockedOut, BadLogonCount, LastBadPasswordAttempt |
    Select-Object Name, LockedOut, BadLogonCount, LastBadPasswordAttempt
```

**4.** Unlock:
```powershell
Unlock-ADAccount -Identity "[username]"
```

**5.** Verify:
```powershell
Get-ADUser -Identity "[username]" -Properties LockedOut | Select-Object Name, LockedOut
# Should show: False
```

**6.** Reply to user. Set ticket to **Resolved**.

---

## Common Cause of Repeat Lockouts

A saved password on a phone or another PC keeps retrying the old password. Ask the user to update or remove saved credentials on **all devices**.

---

## Ticket Note Template
```
SOP: HD-002 | Agent: Munashe | Date: [datetime]
─────────────────────────────────────────────
Action: Account unlocked via Unlock-ADAccount
Bad logon count at action: [number]
User advised: update saved credentials on all devices
SLA: [Met / Breached]
```
