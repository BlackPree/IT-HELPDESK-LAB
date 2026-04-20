# Phase 3 — AWS Setup

**Goal:** Launch a Linux EC2 jump box, create an S3 log archive bucket, configure CloudWatch billing alarms, and set up IAM permissions.

**Time:** ~2 hours | **Cost:** $0 (AWS Free Tier)

---

## Table of Contents
- [Billing Protection](#billing-protection)
- [Launch JUMP-01 Linux EC2](#launch-jump-01-linux-ec2)
- [Configure Security Group](#configure-security-group)
- [Connect via SSH](#connect-via-ssh)
- [Create S3 Log Bucket](#create-s3-log-bucket)
- [Create IAM Policy for S3 Access](#create-iam-policy-for-s3-access)
- [Install AWS CLI on DC01](#install-aws-cli-on-dc01)
- [Stop Instances](#stop-instances)
- [Troubleshooting](#troubleshooting)

---

## Billing Protection

1. Switch region to **us-east-1** (billing alarms only work here)
2. Search → **CloudWatch** → **Alarms** → **Create alarm**
3. Select metric → Billing → Total Estimated Charge → EstimatedCharges → Select metric
4. Threshold: Greater than `1`
5. Create SNS topic → `BillingAlert` → your email → **Create topic**
6. **Open the confirmation email AWS sends and click the link** — critical step
7. Name: `$1-Billing-Alert` → Create

Repeat for a `$5-Billing-Alert`.

Also create a zero-spend budget:
Billing → **Budgets** → Create budget → **Zero spend budget** → your email → Create

---

## Launch JUMP-01 Linux EC2

The jump box is a hardened Linux server — the single entry point into your AWS environment. osTicket will also run here.

1. Search → **EC2** → **Launch instance**

2. Settings:

   | Field | Value |
   |---|---|
   | Name | `JUMP-01` |
   | AMI | **Amazon Linux 2023 AMI** *(Free tier eligible)* |
   | Instance type | **t2.micro** *(Free tier eligible)* |
   | Key pair | **Create new key pair** → name: `helpdesk-key` → RSA → `.pem` → Create |
   | Auto-assign public IP | **Enabled** |
   | Storage | **8 GiB** gp3 (well within 30 GB free limit) |

3. Click **Launch instance**

> 🔑 Save the `.pem` file securely — you cannot recover it. Store it in two places.

---

## Configure Security Group

After launching, restrict access to JUMP-01:

EC2 → Instances → click JUMP-01 → **Security** tab → click the Security Group → **Edit inbound rules**

| Type | Protocol | Port | Source | Purpose |
|---|---|---|---|---|
| SSH | TCP | 22 | **My IP** | Your home IP only — no one else can SSH in |
| HTTP | TCP | 80 | 0.0.0.0/0 | osTicket web interface (public) |
| HTTPS | TCP | 443 | 0.0.0.0/0 | osTicket over HTTPS (optional) |

> All other ports: blocked. This is the least-privilege model — only what's needed, nothing more.

---

## Connect via SSH

### Get JUMP-01's public IP
EC2 → click JUMP-01 → copy **Public IPv4 address**

### Connect

**Mac / Linux terminal:**
```bash
# Fix key permissions first (required)
chmod 400 helpdesk-key.pem

# Connect
ssh -i helpdesk-key.pem ec2-user@[PUBLIC_IP]
```

**Windows (PowerShell or Command Prompt):**
```powershell
# Move key to a safe place first, then:
ssh -i C:\Users\YourName\.ssh\helpdesk-key.pem ec2-user@[PUBLIC_IP]
```

**Windows (PuTTY):**
1. Convert the `.pem` to `.ppk` using PuTTYgen
2. PuTTY → Session → enter IP → Connection → SSH → Auth → browse to `.ppk` → Open

You are now inside your Linux jump box.

---

## Create S3 Log Bucket

S3 buckets store the log files shipped from your Azure VMs.

1. Search → **S3** → **Create bucket**

2. Settings:

   | Field | Value |
   |---|---|
   | Bucket name | `helpdesk-logs-[yourname]` (must be globally unique — add your name) |
   | Region | Same region as your EC2 (e.g. us-east-1) |
   | Block all public access | ✅ **ON** — logs must not be public |
   | Versioning | Disabled (not needed for lab) |

3. Click **Create bucket**

### Create a folder structure inside the bucket

S3 → click your bucket → **Create folder**

Create these folders:
```
helpdesk-logs-[yourname]/
├── security-logs/
├── system-logs/
└── application-logs/
```

---

## Create IAM Policy for S3 Access

DC01 (your Azure VM) needs to write logs to S3. Create an IAM user with write-only access to your specific bucket.

### Create the IAM policy

1. Search → **IAM** → **Policies** → **Create policy**
2. Switch to **JSON** tab and paste:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3LogUpload",
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::helpdesk-logs-[yourname]",
        "arn:aws:s3:::helpdesk-logs-[yourname]/*"
      ]
    }
  ]
}
```

3. Name: `HelpdeskLogUploadPolicy` → **Create policy**

> This policy allows ONLY uploading to YOUR specific bucket. No access to anything else in AWS.

### Create an IAM user for DC01

1. IAM → **Users** → **Create user**
2. Username: `dc01-log-shipper`
3. Permissions → **Attach policies directly** → search and check `HelpdeskLogUploadPolicy`
4. Create user → click the user → **Security credentials** → **Create access key**
5. Use case: **Application running outside AWS** → Create
6. **Copy both the Access Key ID and Secret Access Key** — save them securely. You will NOT see the secret again.

> ⚠️ Never commit these keys to GitHub. Store them in a password manager.

---

## Install AWS CLI on DC01

The AWS CLI lets DC01 (your Azure VM) upload files to S3 from PowerShell.

RDP into DC01 → open PowerShell as Administrator:

```powershell
# Download and install AWS CLI
Invoke-WebRequest -Uri "https://awscli.amazonaws.com/AWSCLIV2.msi" -OutFile "C:\Temp\AWSCLIV2.msi"
Start-Process msiexec.exe -Args "/i C:\Temp\AWSCLIV2.msi /quiet" -Wait

# Verify installation
aws --version

# Configure with the IAM user credentials
aws configure
# Enter: Access Key ID, Secret Access Key, Region (e.g. us-east-1), Output format: json

# Test access — should list your bucket
aws s3 ls s3://helpdesk-logs-[yourname]
```

---

## Stop Instances

Stop JUMP-01 when not using it:
EC2 → select JUMP-01 → **Instance State** → **Stop**

For Azure VMs: Azure portal → VM → **Stop** (must show Deallocated)

---

## Troubleshooting

| Problem | Solution |
|---|---|
| SSH: `Permission denied (publickey)` | Key permissions wrong — run `chmod 400 helpdesk-key.pem` |
| SSH: `Connection timed out` | Your IP changed — update the SSH rule in Security Group: Source → My IP |
| S3 access denied from DC01 | Verify `aws configure` used the dc01-log-shipper credentials, not root keys |
| Bucket name taken | S3 names are global — add more characters e.g. `helpdesk-logs-munashe-2025` |

---

**Next:** [Phase 4 — osTicket Setup →](04-osticket-setup.md)
