# osTicket Configuration Reference

Settings used in this lab. **Remove any actual passwords before committing.**

---

## System Settings

| Setting | Value |
|---|---|
| Helpdesk Name | Helpdesk Lab Support |
| Default Email | [your email] |
| Ticket Number Format | `#%06` |
| Status | Online |

## Help Topics

| Topic | SLA | Department |
|---|---|---|
| Password Reset | SEV-A (2 hrs) | Support |
| Account Unlock | SEV-A (2 hrs) | Support |
| New User Request | SEV-B (8 hrs) | Sysadmin |
| Account Termination | SEV-B (8 hrs) | Sysadmin |
| General IT Support | SEV-C (24 hrs) | Support |

## SLA Plans

| Name | Grace Period | Schedule |
|---|---|---|
| SEV-A Urgent | 2 hours | 24/7 |
| SEV-B Normal | 8 hours | Business hours |
| SEV-C Low | 24 hours | Business hours |

## Database

- Host: `localhost`
- Database name: `osticket_db`
- User: `osticket_user`
- **Password: NOT stored here — stored in a password manager only**

## Install Path

- Web root: `/var/www/html/osticket/upload/`
- Admin panel: `http://[IP]/osticket/upload/scp/`
- User portal: `http://[IP]/osticket/upload/`
