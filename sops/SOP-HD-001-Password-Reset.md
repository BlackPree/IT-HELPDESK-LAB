# SOP-HD-001 — Password Reset Procedure

**ID:** HD-001 | **Version:** 1.0 | **Tier:** L1 Support | **SLA:** 2 hours (SEV-A)

---

## Overview

Reset an Active Directory user's password following a verified osTicket request.

---

## Identity Verification — Required

Verify the user before acting. Use one of:

| Method | How |
|---|---|
| Employee ID | Ask user to provide it — confirm it matches their osTicket profile |
| Manager confirmation | Contact manager separately to confirm the request |

Add an **internal note** to the ticket: `Identity verified via [method] at [time].`

---

## Steps

**1.** Open osTicket. Assign ticket to yourself. Check SLA countdown.

**2.** Verify identity. Add internal note.

**3.** RDP into DC01 as `HELPDESK\azureuser`.

**4.** Open PowerShell 7 as Administrator → `cd C:\Scripts`

**5.** Run:
```powershell
.\Reset-HelpdeskPassword.ps1 -Username "[username]" -TicketID "[ticket_id]"
```

**6.** Copy the output block and paste as internal note in osTicket.

**7.** Reply to user with temp password. Set ticket to **Resolved**.

---

## Ticket Note Template
```
SOP: HD-001 | Agent: Munashe | Date: [datetime]
─────────────────────────────────────────────
Actions: Password reset, must-change enabled, account unlocked.
AD username: [username] | Ticket: #[id]
Identity verified: [method]
Temp PW sent: osTicket reply
SLA: [Met / Breached]
```

---

## Escalate to L2 if
- Account is disabled
- User is in `Disabled_Users` OU
- User reports repeated lockouts after reset
