---
title: "VPC & Networking Configuration"
date: "2026-06-15"
weight: 2
chapter: false
pre: " <b> 5.2.2 </b> "
---

#### Overview

Initialize the Virtual Private Cloud (VPC) to serve as the secure network foundation for the Enterprise IoT Cloud Dashboard. This acts as the isolated environment where the PostgreSQL RDS and EC2 backend will operate securely. You will design the VPC, public/private subnets, Internet Gateway, and Route Tables to establish the core architecture.

#### Network Preparation

We will configure the VPC and provision the foundational compute resources for the backend.

**Step 1. Design VPC and Networking**

- Access the **VPC** service from the search bar.
- **AWS Region:** Select `United States (N. Virginia us-east-1)`.

![Access VPC](/images/5-Workshop/5.2-Prerequisite/04_VPC.jpg)

- Click **Create VPC**.

![Create VPC](/images/5-Workshop/5.2-Prerequisite/05_VPC_Create.jpg)

- Configure Networking information:
    - Design the VPC with public and private subnets.
    - Configure the Internet Gateway to allow external traffic.
    - Set up Route Tables to manage network traffic flow.

![Configure VPC](/images/5-Workshop/5.2-Prerequisite/06_Configure_VPC.jpg)

- Scroll to the bottom of the page, Click **Create VPC**.

![Finished Create VPC](/images/5-Workshop/5.2-Prerequisite/07_Finished_Create_VPC.jpg)

- Check that the VPC has been created successfully.

![Create VPC Successful](/images/5-Workshop/5.2-Prerequisite/08_Create_VPC_Successful.jpg)

**Step 2. EC2 Provisioning & Security Groups**

- Once the network is ready, we need to provision the compute layer for the FastAPI backend.

- Navigate to the **EC2** service.
![Click_EC2](/images/5-Workshop/5.2-Prerequisite/09_Click_EC2.jpg)

- Click **Launch instances**.
![Launch_Instance](/images/5-Workshop/5.2-Prerequisite/10_Launch_Instance.jpg)

- In the Launch interface:
    - Select an Ubuntu EC2 instance for the FastAPI backend.
    - Attach an Elastic IP to ensure a static public endpoint.
    - Configure Security Groups to specifically allow HTTP and SSH access.
- Scroll to the bottom of the page, Click **Launch instance**.

![Configure EC2](/images/5-Workshop/5.2-Prerequisite/11_Configure_EC2.jpg)

- When you see the success notification, Click **Close**.

![Launch successfully](/images/5-Workshop/5.2-Prerequisite/12_Successfully.jpg)