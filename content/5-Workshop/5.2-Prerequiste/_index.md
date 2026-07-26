---
title: "Prerequisite"
date: "2026-06-15"
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

#### Objectives

Before building the Enterprise IoT Cloud Dashboard, we need to establish a solid foundation. Similar to preparing ingredients before cooking, this section ensures your AWS environment is ready with the necessary security protocols and core architecture.

In this section, we will complete 3 key initialization objectives:

1.  **Set up the AWS environment:** Finalize the 5-layer architecture diagram and assign strict roles to team members.
2.  **Establish IAM security protocols:** Ensure the Cloud Engineer sets up IAM users, groups, policies, and enforces MFA for all accounts.
3.  **Initialize Repository:** Set up the Git repository and define branching strategies following Agile/Scrum standards.

#### Key Components

In this preparation section, we will interact with the following components:

- **AWS IAM (Identity and Access Management):** The service used to manage users, groups, policies, and enforce MFA for system-wide security.
- **Amazon VPC (Virtual Private Cloud):** The foundational networking service where we will design the VPC, public/private subnets, Internet Gateway, and Route Tables.
- **Git Repository:** The version control system initialized to define branching strategies and prepare deployment scripts.

#### Implementation Steps

1.  [AWS IAM & Security Setup](5.2.1-IAM-Setup/)
2.  [VPC & Networking Configuration](5.2.2-VPC-Networking/)