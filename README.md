# Cloud-Native Hybrid Helpdesk Ecosystem (Azure + AWS)

<p align="center">
  <img src="https://img.shields.io/badge/Azure-0089D6?style=for-the-badge&logo=microsoft-azure&logoColor=white" />
  <img src="https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white" />
  <img src="https://img.shields.io/badge/Windows_Server-0078D6?style=for-the-badge&logo=microsoft&logoColor=white" />
  <img src="https://img.shields.io/badge/PowerShell-5391FE?style=for-the-badge&logo=powershell&logoColor=white" />
</p>

---

## Hi, I'm Munashe 👋

An aspiring Cloud & Security Engineer. This repository documents a **Zero-Cost Hybrid Cloud Helpdesk Lab**. I built this to demonstrate proficiency in Active Directory, Multi-Cloud integration (Azure & AWS), and automated ITSM workflows.

### 🎯 Objective
To deploy a fully functional IT Support environment using only **Free Tier** resources, ensuring secure identity management and automated ticketing.

---

## 🏗️ Architecture Overview

![Architecture Diagram](DOCUMENTATION_LINK_OR_IMAGE_PATH_HERE)

The project bridges two major cloud providers to simulate a real-world enterprise environment:

* **Azure Environment:**
    * **Windows Server 2022:** Functions as the Primary Domain Controller (AD DS, DNS, DHCP).
    * **Windows 10 Client:** Domain-joined workstation for end-user simulation.
    * **PowerShell 7:** Used for bulk user creation and system automation.
* **AWS Environment:**
    * **EC2 Jump Box:** A Linux-based entry point for secure management.
    * **S3 Bucket:** Acts as a centralized log archive for long-term storage.
    * **CloudWatch:** Monitoring and billing alerts to ensure "Zero Cost" compliance.
* **ITSM Layer:**
    * **osTicket (Open Source):** Handles ticket lifecycle, SLAs, and automated email/webhook notifications.

---

## 🚀 Key Features

- **Automated User Onboarding:** PowerShell scripts to populate Active Directory with hundreds of test users instantly.
- **Cross-Cloud Logging:** Shipping system logs from Azure VMs to AWS S3 for centralized auditing.
- **Network Security:** Configured Network Security Groups (NSG) and Security Groups to restrict traffic to "Least Privilege."
- **Ticketing Workflow:** End-to-end flow from user issue submission to technician resolution.

---

## 📂 Project Structure

| Folder | Description |
| :--- | :--- |
| `Scripts/` | PowerShell scripts for AD setup and automation. |
| `Documentation/` | Detailed step-by-step guides and configuration screenshots. |
| `Configs/` | IAM policies and osTicket configuration snippets. |

---

## 🤝 Connect with Me

* **LinkedIn:** [Munashe Nyarwendo](https://www.linkedin.com/in/munashenyarwendo/)

---
*This project was built for educational purposes to demonstrate cloud architectural skills.*
