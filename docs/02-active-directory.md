# Phase 2 — Active Directory Setup

**Goal:** Promote DC01 to a Domain Controller running `helpdesk.local`, build the OU structure, create test users, configure Group Policy, and join CLIENT01 to the domain.

**Time:** ~3 hours | **Prerequisites:** [Phase 1 complete](01-azure-setup.md)

---

## Table of Contents
- [Set Static Private IP](#set-static-private-ip)
- [Install AD DS Role](#install-ad-ds-role)
- [Promote to Domain Controller](#promote-to-domain-controller)
- [Create OU Structure](#create-ou-structure)
- [Create Test Users Manually](#create-test-users-manually)
- [Configure Group Policy](#configure-group-policy)
- [Join CLIENT01 to Domain](#join-client01-to-domain)
- [Verification Checklist](#verification-checklist)
- [Troubleshooting](#troubleshooting)

---

## Set Static Private IP

AD DS requires a fixed internal IP. Do this before installing any roles.

1. RDP into DC01
2. Right-click network icon (system tray) → **Open Network & Internet settings**
3. **Change adapter options** → right-click **Ethernet** → **Properties**
4. **Internet Protocol Version 4 (TCP/IPv4)** → **Properties**
5. Select **Use the following IP address**:

   | Field | Value |
   |---|---|
   | IP address | DC01's current private IP (Azure portal → DC01 → Private IP address, e.g. `10.0.1.4`) |
   | Subnet mask | `255.255.255.0` |
   | Default gateway | `10.0.1.1` |
   | Preferred DNS | `127.0.0.1` (DC points to itself) |
   | Alternate DNS | `8.8.8.8` (Google fallback for internet) |

6. Click **OK** → **Close**

> Also set the static IP in Azure itself: Azure portal → DC01 → **Networking** → click the NIC → **IP configurations** → click `ipconfig1` → set Assignment to **Static** → Save.

---

## Install AD DS Role

1. RDP into DC01 → open **Server Manager**
2. **Add roles and features** → Next → Next → Next
3. Check **Active Directory Domain Services** → **Add Features** → Next → Next → Next → **Install**
4. Wait ~5 minutes → click **Close** (do not restart yet)

---

## Promote to Domain Controller

1. Server Manager → **yellow flag** → **Promote this server to a domain controller**

2. Wizard settings:

   | Screen | Setting |
   |---|---|
   | Deployment | **Add a new forest** |
   | Root domain name | `helpdesk.local` |
   | Forest/Domain functional level | Windows Server 2016 |
   | DNS Server | ✅ checked |
   | DSRM Password | Set one and write it down |
   | DNS Delegation | Ignore warning — click Next |
   | NetBIOS name | `HELPDESK` (auto-filled) |
   | Paths | Leave all defaults |

3. Click **Install** — server restarts automatically (~5 minutes)

### After restart

Login as:
```
Username: HELPDESK\azureuser
Password: same as before
```

---

## Create OU Structure

1. Start → **Active Directory Users and Computers**
2. Expand `helpdesk.local` → right-click → **New** → **Organizational Unit**
3. Create all 5 OUs:

| OU | Purpose |
|---|---|
| `IT` | IT department staff |
| `HR` | Human Resources |
| `Finance` | Finance team |
| `Helpdesk` | Helpdesk agents |
| `Disabled_Users` | Offboarded accounts — archived, not deleted |

---

## Create Test Users Manually

Create these 10 users by hand before automating — builds familiarity with AD.

Right-click the OU → **New** → **User** → fill in details → Next → set password → check **User must change password at next logon** → Finish.

| OU | First | Last | Username | Title |
|---|---|---|---|---|
| IT | John | Smith | `jsmith` | IT Support L1 |
| IT | Alice | Walker | `awalker` | IT Support L2 |
| IT | Bob | Lee | `blee` | Systems Admin |
| HR | Sarah | Johnson | `sjohnson` | HR Coordinator |
| HR | Mark | Taylor | `mtaylor` | HR Manager |
| Finance | Emma | Brown | `ebrown` | Finance Analyst |
| Finance | David | Clark | `dclark` | Finance Coordinator |
| Finance | Tom | Davis | `tdavis` | Finance Manager |
| Helpdesk | Lisa | White | `lwhite` | Helpdesk Agent |
| Helpdesk | James | Hall | `jhall` | Helpdesk Agent |

---

## Configure Group Policy

1. Start → **Group Policy Management**
2. Expand: Forest → Domains → `helpdesk.local`
3. Right-click **Default Domain Policy** → **Edit**

### Password Policy
Path: `Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies → Password Policy`

| Setting | Value |
|---|---|
| Minimum password length | 8 |
| Password complexity | Enabled |
| Maximum password age | 90 days |
| Minimum password age | 1 day |
| Enforce password history | 5 |

### Account Lockout Policy
Path: `Account Policies → Account Lockout Policy`

| Setting | Value |
|---|---|
| Lockout threshold | 5 attempts |
| Lockout duration | 30 minutes |
| Reset counter after | 30 minutes |

Apply immediately:
```powershell
gpupdate /force
```

---

## Join CLIENT01 to Domain

**On CLIENT01** (RDP in separately):

**Step 1 — Point DNS to DC01:**
1. Network adapter settings → IPv4 → Properties
2. Preferred DNS: DC01's private IP (e.g. `10.0.1.4`)

**Step 2 — Join the domain:**
1. Right-click Start → **System** → **Rename this PC (advanced)** → **Change**
2. Select **Domain** → type `helpdesk.local`
3. Credentials: `HELPDESK\azureuser` + password
4. Click OK → restart

**Step 3 — Verify:**
Back on DC01 → AD Users and Computers → **Computers** container → CLIENT01 should appear.

---

## Verification Checklist

- [ ] DC01 shows Domain Controller role in Server Manager
- [ ] `helpdesk.local` domain visible in AD Users and Computers
- [ ] All 5 OUs created
- [ ] 10 test users created in correct OUs
- [ ] Group Policy applied — `gpupdate /force` ran successfully
- [ ] CLIENT01 visible in AD Computers container
- [ ] Can log into CLIENT01 using a domain account (e.g. `HELPDESK\jsmith`)

---

## Troubleshooting

| Problem | Solution |
|---|---|
| Post-restart login fails | Use `HELPDESK\azureuser` — not just `azureuser` |
| CLIENT01 can't find `helpdesk.local` | DC01's private IP must be CLIENT01's Preferred DNS |
| `gpupdate` shows errors | Check Event Viewer → Windows Logs → System for Group Policy errors |
| Domain join fails with "domain not found" | Ping DC01 from CLIENT01 — if it fails, check NSG ICMP rule |

---

**Next:** [Phase 3 — AWS Setup →](03-aws-setup.md)
