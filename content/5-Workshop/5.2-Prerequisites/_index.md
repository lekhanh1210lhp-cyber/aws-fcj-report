---
title: "Prerequisites"
date: "2026-07-28"
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

# Prerequisites

## AWS and local environment

Prepare an AWS account and select **Asia Pacific (Singapore), `ap-southeast-1`**. Your identity needs permission to create or inspect VPC, EC2, RDS, IAM roles, and CloudWatch resources. Use least privilege whenever the workshop provides a narrower option.

Install Git, VS Code, Python, Node.js/npm, and PlatformIO on Windows. The EC2 host will need Python 3, `pip`, Git, and the PostgreSQL client.

Verify the local tools:

```powershell
git --version
python --version
node --version
npm --version
pio --version
```

## Hardware and knowledge

Prepare a YOLO UNO, DHT20, light sensor, fan, relay or light, curtain servo, jumper wires, and a suitable power supply. You should understand basic REST requests, PostgreSQL, Security Groups, environment variables, and Linux `systemd`.

## Checklist

- [ ] AWS Console opens in `ap-southeast-1`.
- [ ] Required local commands return a version.
- [ ] Hardware and cables are available.
- [ ] Source repositories can be cloned.
- [ ] No credentials are stored in tracked files.

**Expected result:** The team can begin AWS provisioning without pausing for missing tools or access.

## Troubleshooting

- If `python` or `pio` is not recognized, reopen the terminal after installation and check `PATH`.
- If AWS actions are denied, capture the denied action name and ask the account administrator for the minimum required permission.

Next: [review the architecture and data flows](../5.3-Architecture-and-Service-Design/).
