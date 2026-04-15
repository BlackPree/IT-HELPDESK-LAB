# Technical Roadmap: Multi-Cloud Identity & ITSM Integration

This document outlines the high-level technical strategy and step-by-step execution used to bridge Azure and AWS environments.

## Phase 1: Azure Identity Core
**Goal:** Build a functional corporate network environment.
- **Domain Controller:** Deployed Windows Server 2022 on `Standard_B1s`.
- **Static Networking:** Assigned Static Private IP (`10.0.0.4`) to ensure identity persistence.
- **DNS Configuration:** Updated Azure VNet DNS settings to point to the DC's Private IP, allowing the Win10 Client to discover the domain.
- **Domain Join:** Provisioned a Windows 10 Pro VM and successfully joined the `cloudlab.local` forest.

## Phase 2: AWS Monitoring & Storage Layer
**Goal:** Implement cross-cloud security and cost governance.
- **S3 Security:** Created a private bucket with **AES-256 Server-Side Encryption (SSE-S3)**.
- **IAM Hardening:** Created the `AzureLogUploader` user with a custom inline policy restricted to `s3:PutObject`.
- **Cost Governance:** Configured a **CloudWatch Billing Alarm** at a $0.01 threshold to demonstrate professional resource management.

## Phase 3: ITSM (osTicket) Integration
**Goal:** Deploy a service-oriented architecture for end-users.
- **Web Stack:** Installed IIS, PHP 8.x, and MySQL on the Azure Windows Server.
- **Ticket Workflow:** Configured Help Topics (Password Reset, Hardware, Software) and simulated the full ticket lifecycle from the Windows 10 client.

## Phase 4: Connectivity & Teardown
- **ICMP Verification:** Configured Azure NSG and AWS Security Group rules to allow Echo Requests (Ping) between clouds.
- **Resource Lifecycle:** Documented teardown procedures to ensure no "zombie" resources remained in the free-tier accounts.
