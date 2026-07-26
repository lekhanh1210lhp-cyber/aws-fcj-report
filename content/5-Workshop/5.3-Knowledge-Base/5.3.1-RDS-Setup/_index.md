---
title: "AWS RDS Setup"
date: "2026-06-15"
weight: 1
chapter: false
pre: " <b> 5.3.1 </b> "
---

#### Target

We will use the AWS Management Console to provision a PostgreSQL RDS instance. This process will deploy the database in a private subnet and configure inbound rules to exclusively accept traffic from the EC2 instance to ensure security.

#### Implementation Steps

1.  Log in to the **AWS Management Console** and access the **RDS** service.
2.  In the left-hand menu, select **Databases**.

![Click_Databases](/images/5-Workshop/5.3-RDS-Setup/01_Click_Databases.jpg)

3.  Click the **Create database** button in the top right corner of the screen.

![Create Database](/images/5-Workshop/5.3-RDS-Setup/02_Create_Database.jpg)

**Step 1: Choose a database creation method & Engine**

On the first configuration screen:

1. **Choose a database creation method:** Select `Standard create`.
2. **Engine options:** Select `PostgreSQL`.
3. **Templates:** Choose `Free tier` or `Dev/Test` depending on your budget configuration.

![Configure Engine](/images/5-Workshop/5.3-RDS-Setup/03_Configure_Engine.jpg)

**Step 2: Settings**

Configure the core database details:

1.  **DB instance identifier:** Enter `iot-dashboard-db`
2.  **Master username:** Enter `postgres` (or your preferred admin username).
3.  **Master password:** Enter a strong password and confirm it. Keep this secure for the FastAPI backend connection later.

![Configure Settings](/images/5-Workshop/5.3-RDS-Setup/04_Configure_Settings.jpg)

**Step 3: Connectivity Configuration**

This is the most critical step to secure the database layer:

1.  **Virtual private cloud (VPC):** Select the custom VPC you created in the previous steps.
2.  **Public access:** Select `No` (This ensures the database is deployed in a private subnet).
3.  **VPC security group (firewall):** Choose `Create new` (e.g., `rds-ec2-sg`) or select an existing one.
4.  *Note:* You must configure the inbound rules of this Security Group to allow PostgreSQL traffic (Port 5432) **only from the EC2 instance** running the FastAPI backend.

![Configure Connectivity](/images/5-Workshop/5.3-RDS-Setup/05_Configure_Connectivity.jpg)

**Step 4: Review and Create Database**

1.  Review all configuration information on the summary page.
2.  Ensure the VPC, Subnet, and Public Access (No) configurations are correct.
3.  Scroll to the bottom of the page and click the **Create database** button.

![Step 4](/images/5-Workshop/5.3-RDS-Setup/06_Step_4.jpg)

**Step 5: Wait for Initialization**

After clicking Create, the system will begin provisioning the RDS instance.

- **Wait time:** Approximately **5 - 10 minutes**.
- **Note:** You can navigate away from this page, but wait for completion before connecting the backend.
- **Success:** When the database status changes to **"Available"**, you have completed this step and are ready to initialize the FastAPI backend schemas.

![Step 5](/images/5-Workshop/5.3-RDS-Setup/07_Step_5.jpg)