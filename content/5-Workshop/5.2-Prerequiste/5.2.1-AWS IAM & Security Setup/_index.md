---
title: "AWS IAM & Security Setup"
date: "2026-06-15"
weight: 1
chapter: false
pre: " <b> 5.2.1 </b> "
---

#### Overview

Security is a foundational pillar when building the Enterprise IoT Cloud Dashboard. Before deploying any resources, the Cloud Engineer must establish strict security protocols. 

Ensure your AWS account is configured with the principle of least privilege. This involves setting up IAM users, groups, and policies, as well as enforcing Multi-Factor Authentication (MFA) for all accounts to prevent unauthorized access. 

#### IAM Configuration

We will perform the initial setup to ensure your environment is secure.

In the search bar, access the [AWS IAM Console](https://us-east-1.console.aws.amazon.com/iam/home?region=us-east-1#/home).

![Access AWS IAM](/images/5-Workshop/5.2-Prerequisite/00_AWS_IAM.jpg)

**Step 1. Access Users and Groups**

- In the left menu of the IAM Console, navigate to **Access management**.
- Click **User groups** to establish role-based access.

!(/images/5-Workshop/5.2-Prerequisite/01_IAM_Groups.jpg)

**Step 2. Define Team Roles**

- Click **Create group**.
- Create distinct groups for the project roles: `CloudEngineers`, `BackendEngineers`, `FrontendEngineers`, and `IoTEngineers`.
- Attach relevant foundational policies to these groups (e.g., EC2 and RDS access for Cloud Engineers).
- Click **Create group**.

!(/images/5-Workshop/5.2-Prerequisite/02_Create_Groups.jpg)

**Step 3. Enforce MFA**

- Navigate to **Users** in the left menu.
- Select an individual user account.
- Go to the **Security credentials** tab.
- Under Multi-factor authentication (MFA), click **Assign MFA device**.
- Follow the prompts to configure a virtual authenticator app. This is a mandatory requirement for all accounts.

!https://encrypted-tbn1.gstatic.com/licensed-image?q=tbn:ANd9GcTgyHugjFkSKqHzugyAAUgrzxlRLW_xPQXsGkIuQf8ZIx7GM3Pttvp0ibjGG6VM1biSq4LY-c1FiwBTrpQ(/images/5-Workshop/5.2-Prerequisite/03_Enforce_MFA.jpg)