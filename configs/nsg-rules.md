# Azure NSG Rules — Reference

These are the exact Network Security Group rules applied to each VM in this lab.

---

## DC01 — Inbound Rules

| Priority | Name | Port | Protocol | Source | Action |
|---|---|---|---|---|---|
| 100 | Allow-RDP | 3389 | TCP | Your home IP | Allow |
| 110 | Allow-DNS-TCP | 53 | TCP | 10.0.0.0/16 | Allow |
| 120 | Allow-DNS-UDP | 53 | UDP | 10.0.0.0/16 | Allow |
| 130 | Allow-Kerberos-TCP | 88 | TCP | 10.0.0.0/16 | Allow |
| 140 | Allow-Kerberos-UDP | 88 | UDP | 10.0.0.0/16 | Allow |
| 150 | Allow-LDAP | 389 | TCP | 10.0.0.0/16 | Allow |
| 160 | Allow-SMB | 445 | TCP | 10.0.0.0/16 | Allow |
| 170 | Allow-LDAPS | 636 | TCP | 10.0.0.0/16 | Allow |
| 180 | Allow-RPC | 135 | TCP | 10.0.0.0/16 | Allow |
| 190 | Allow-ICMP | * | ICMP | 10.0.0.0/16 | Allow |
| 4096 | Deny-All-Inbound | * | * | * | **Deny** |

## CLIENT01 — Inbound Rules

| Priority | Name | Port | Protocol | Source | Action |
|---|---|---|---|---|---|
| 100 | Allow-RDP | 3389 | TCP | Your home IP | Allow |
| 4096 | Deny-All-Inbound | * | * | * | **Deny** |

## JUMP-01 (AWS) — Inbound Rules

| Type | Port | Source | Purpose |
|---|---|---|---|
| SSH | 22 | Your home IP | Secure shell access |
| HTTP | 80 | 0.0.0.0/0 | osTicket web portal |
| HTTPS | 443 | 0.0.0.0/0 | osTicket over SSL |
