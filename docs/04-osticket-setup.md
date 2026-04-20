# Phase 4 — osTicket Setup

**Goal:** Install and configure osTicket (open source ticketing system) on the Linux EC2 jump box.

**Time:** ~2 hours | **Cost:** $0 (open source, self-hosted)

---

## Table of Contents
- [What is osTicket?](#what-is-osticket)
- [Install LAMP Stack](#install-lamp-stack)
- [Install osTicket](#install-osticket)
- [Configure osTicket](#configure-osticket)
- [Create Help Topics](#create-help-topics)
- [Configure SLA Plans](#configure-sla-plans)
- [Create Agent Accounts](#create-agent-accounts)
- [Test the Setup](#test-the-setup)
- [Troubleshooting](#troubleshooting)

---

## What is osTicket?

osTicket is a free, open source IT ticketing system used by real companies worldwide. Unlike Zendesk (which has a 14-day trial), osTicket:

- Runs forever for free
- Is self-hosted on your own server (the EC2 jump box)
- Demonstrates Linux server admin skills (installing a LAMP stack)
- Is used in real SME environments — it's on many job descriptions

---

## Install LAMP Stack

osTicket needs Apache (web server), MySQL (database), and PHP. SSH into JUMP-01:

```bash
# Update system packages
sudo dnf update -y

# Install Apache web server
sudo dnf install -y httpd

# Start Apache and enable on boot
sudo systemctl start httpd
sudo systemctl enable httpd

# Install MySQL (MariaDB)
sudo dnf install -y mariadb105-server

# Start MySQL
sudo systemctl start mariadb
sudo systemctl enable mariadb

# Secure MySQL installation
sudo mysql_secure_installation
# Press Enter for current password (blank)
# Set root password: choose a strong one, write it down
# Answer Y to all remaining prompts

# Install PHP and required extensions
sudo dnf install -y php php-mysqlnd php-imap php-intl php-mbstring php-xml php-gd php-fileinfo php-opcache

# Restart Apache to load PHP
sudo systemctl restart httpd
```

Verify Apache is running — open a browser and go to `http://[JUMP-01_PUBLIC_IP]`. You should see the Amazon Linux test page.

---

## Install osTicket

```bash
# Install wget and unzip
sudo dnf install -y wget unzip

# Download osTicket
cd /tmp
wget https://github.com/osTicket/osTicket/releases/download/v1.18.1/osTicket-v1.18.1.zip

# Extract to web directory
sudo unzip osTicket-v1.18.1.zip -d /var/www/html/osticket

# Set permissions
sudo chown -R apache:apache /var/www/html/osticket
sudo chmod -R 755 /var/www/html/osticket

# Copy the sample config
sudo cp /var/www/html/osticket/upload/include/ost-sampleconfig.php \
        /var/www/html/osticket/upload/include/ost-config.php

sudo chmod 0666 /var/www/html/osticket/upload/include/ost-config.php
```

### Create the osTicket database

```bash
sudo mysql -u root -p
```

Inside MySQL, run:
```sql
CREATE DATABASE osticket_db;
CREATE USER 'osticket_user'@'localhost' IDENTIFIED BY 'StrongPassword123!';
GRANT ALL PRIVILEGES ON osticket_db.* TO 'osticket_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

> ⚠️ Write down: database name `osticket_db`, user `osticket_user`, and your password. Do not commit these to GitHub.

---

## Configure osTicket

1. Open a browser and go to: `http://[JUMP-01_PUBLIC_IP]/osticket/upload/setup/`
2. The setup wizard checks requirements — all should be green with the packages installed above
3. Click **Continue**

4. Fill in the form:

   | Field | Value |
   |---|---|
   | Helpdesk Name | `Helpdesk Lab Support` |
   | Default Email | An email you can access (alerts go here) |
   | Admin First Name | Munashe |
   | Admin Last Name | Your last name |
   | Admin Email | Your email |
   | Username | `admin` |
   | Password | Set a strong admin password |
   | Database Host | `localhost` |
   | Database Name | `osticket_db` |
   | Database Username | `osticket_user` |
   | Database Password | Password you set in MySQL |

5. Click **Install Now**

### After installation — security cleanup

```bash
# Remove setup directory (important — must do this)
sudo rm -rf /var/www/html/osticket/upload/setup

# Lock config file
sudo chmod 0644 /var/www/html/osticket/upload/include/ost-config.php
```

Your osTicket admin panel is at: `http://[JUMP-01_PUBLIC_IP]/osticket/upload/scp/`  
Your user ticket portal is at: `http://[JUMP-01_PUBLIC_IP]/osticket/upload/`

---

## Configure osTicket

Log into the admin panel: `http://[IP]/osticket/upload/scp/`

### System Settings

Admin Panel → **Settings** → **System**

| Setting | Value |
|---|---|
| Helpdesk Name | `Helpdesk Lab Support` |
| Default Ticket Number Format | `#%06` |
| Status | Online |

### Email Settings

Admin Panel → **Emails** → **Emails** → click your default email → configure SMTP if you want real email sending (optional for the lab).

---

## Create Help Topics

Help Topics are the categories users choose when submitting a ticket.

Admin Panel → **Manage** → **Help Topics** → **Add New Help Topic**

| Help Topic | SLA Plan | Department |
|---|---|---|
| `Password Reset` | Urgent (2hrs) | Support |
| `Account Unlock` | Urgent (2hrs) | Support |
| `New User Request` | Normal (1 day) | Sysadmin |
| `Account Termination` | Normal (4hrs) | Sysadmin |
| `General IT Support` | Normal (1 day) | Support |

---

## Configure SLA Plans

Admin Panel → **Manage** → **SLA Plans** → **Add New SLA Plan**

| SLA Name | Grace Period | Schedule | Use for |
|---|---|---|---|
| `SEV-A Urgent` | 2 hours | 24/7 | Password resets, lockouts |
| `SEV-B Normal` | 8 hours | Business hours | New users, transfers |
| `SEV-C Low` | 24 hours | Business hours | General requests |

---

## Create Agent Accounts

Agents are the helpdesk staff who process tickets.

Admin Panel → **Agents** → **Add New Agent**

| Name | Username | Department | Role |
|---|---|---|---|
| Munashe [Surname] | `munashe` | Support | All Access |
| L1 Agent | `l1agent` | Support | Limited (tickets only) |
| L2 Admin | `l2admin` | Sysadmin | All Access |

---

## Test the Setup

1. Open the **user portal** in a new incognito window: `http://[IP]/osticket/upload/`
2. Click **Open a New Ticket**
3. Fill in:
   - Email: any test email
   - Name: Test User
   - Help Topic: **Password Reset**
   - Subject: `I cannot log into my computer`
4. Submit the ticket
5. In the **admin panel**: Admin Panel → **Tickets** → you should see the new ticket with the Urgent SLA countdown

---

## Troubleshooting

| Problem | Solution |
|---|---|
| Apache test page shows instead of osTicket | Check you are going to `/osticket/upload/` not just `/` |
| MySQL connection error during setup | Verify database name, username, and password match exactly what you created |
| Blank page after install | PHP extensions missing — check `sudo dnf install php-xml php-mbstring` |
| Can't receive emails | SMTP setup is optional for the lab — tickets work without email |
| Setup page still shows after install | Run: `sudo rm -rf /var/www/html/osticket/upload/setup` |

---

**Next:** [Phase 5 — PowerShell Scripts →](05-powershell-scripts.md)
