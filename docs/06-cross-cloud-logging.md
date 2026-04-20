# Phase 6 — Cross-Cloud Logging

**Goal:** Ship Windows Event Logs from DC01 (Azure) to an S3 bucket (AWS) — centralised audit trail across both clouds.

**Time:** ~2 hours | **Prerequisites:** AWS CLI configured on DC01, S3 bucket created

---

## Why Cross-Cloud Logging?

In real enterprise environments, security teams need logs from every system in one place regardless of which cloud they run on. A SIEM (Security Information and Event Management) system ingests logs from everywhere for threat detection and compliance.

This setup simulates that pattern:
- **Source:** DC01 on Azure generates Windows Security/System logs
- **Transport:** PowerShell + AWS CLI
- **Destination:** AWS S3 (centralised archive)

This demonstrates:
- Cross-cloud IAM and access control
- Log management fundamentals
- Infrastructure-as-code thinking

---

## What Logs to Collect

| Log Source | Windows Event Log | What it captures |
|---|---|---|
| Security | `Security` | Login events, failed logons, policy changes, account lockouts |
| System | `System` | Service starts/stops, driver issues, system errors |
| Application | `Application` | Application errors, warnings, informational events |
| AD Events | `Security` (filtered) | AD user creation, deletion, group changes |

---

## S3 Bucket Structure

```
helpdesk-logs-[yourname]/
├── security-logs/
│   ├── Security-2025-01-15-0800.json
│   └── Security-2025-01-16-0800.json
├── system-logs/
│   └── System-2025-01-15-0800.json
└── application-logs/
    └── Application-2025-01-15-0800.json
```

---

## Run the Log Shipping Script

From PowerShell as Administrator on DC01:

```powershell
cd C:\Scripts

# Ship last 24 hours of Security logs
.\Ship-LogsToS3.ps1 -BucketName "helpdesk-logs-munashe" -LogType "Security"

# Ship all three log types
.\Ship-LogsToS3.ps1 -BucketName "helpdesk-logs-munashe" -LogType "All"
```

---

## Verify Logs in S3

From DC01 (with AWS CLI configured):
```powershell
# List all files in your bucket
aws s3 ls s3://helpdesk-logs-munashe/ --recursive

# Check file size and dates
aws s3 ls s3://helpdesk-logs-munashe/security-logs/
```

Or in the AWS Console:
S3 → click your bucket → browse into `security-logs/` → you should see timestamped `.json` files.

---

## Schedule Automatic Log Shipping

Set up Windows Task Scheduler on DC01 to ship logs daily automatically:

```powershell
# Create a scheduled task to run every day at 2:00 AM
$Action = New-ScheduledTaskAction `
    -Execute "pwsh.exe" `
    -Argument "-NonInteractive -File C:\Scripts\Ship-LogsToS3.ps1 -BucketName helpdesk-logs-munashe -LogType Security"

$Trigger = New-ScheduledTaskTrigger -Daily -At "02:00"

$Settings = New-ScheduledTaskSettingsSet -RunOnlyIfNetworkAvailable

Register-ScheduledTask `
    -TaskName "DailyLogShipToS3" `
    -Action $Action `
    -Trigger $Trigger `
    -Settings $Settings `
    -RunLevel Highest `
    -Force

Write-Host "Scheduled task created — logs will ship daily at 2:00 AM." -ForegroundColor Green
```

---

## S3 Bucket Policy — Least Privilege

Apply a bucket policy so only your IAM user can write to it:

S3 → your bucket → **Permissions** → **Bucket policy** → paste:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyPublicAccess",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::helpdesk-logs-[yourname]",
        "arn:aws:s3:::helpdesk-logs-[yourname]/*"
      ],
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "false"
        }
      }
    }
  ]
}
```

This enforces HTTPS-only access — no unencrypted uploads allowed.

---

## Verification Checklist

- [ ] AWS CLI configured on DC01 with dc01-log-shipper credentials
- [ ] `Ship-LogsToS3.ps1` runs without errors
- [ ] Log files visible in S3 bucket under correct folders
- [ ] Scheduled task created for daily shipping
- [ ] Bucket policy applied (HTTPS only)
- [ ] Screenshot taken of S3 console showing uploaded log files

---

**Next:** [Phase 7 — Helpdesk Scenarios →](07-scenarios.md)
