---
title: "Results, Challenges, and Future Improvements"
date: "2026-07-28"
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

# Results, Challenges, and Future Improvements

## Results

The implemented workflow connects sensor telemetry, persistent PostgreSQL history, a React dashboard, physical device control, command ACK, and CloudWatch observability. Claim a result only when the matching evidence from Section 5.8 exists.

## Challenges and resolutions

| Challenge | Resolution |
|---|---|
| Singular/plural endpoint mismatch | Use the OpenAPI contract and one canonical path |
| Uvicorn works but `systemd` fails | Check user, working directory, environment file, and `journalctl` |
| Vite proxy or CORS issue | Use relative `/api/...` URLs and validate the proxy target |
| EC2 public IP changes | Update configuration; use a stable endpoint only when intentionally provisioned |
| Commands execute repeatedly | Persist the last command ID and ACK only after success |
| CloudWatch Agent has no data | Check IAM Role, paths, permissions, dimensions, and agent status |

## Limitations

The workshop uses one EC2 instance, HTTP during development, no application authentication, and no guaranteed static public IP. It therefore does not claim high availability, production-grade security, or measured latency guarantees.

## Future improvements

- HTTPS and a managed domain;
- user authentication and authorization;
- stable deployment configuration and CI/CD;
- scalable backend/database design based on measured demand; and
- more advanced automation after a safe rule-based baseline is validated.

**Expected result:** Results, limitations, and future work are supported by evidence and use accurate technical language.

Next: [prepare the project handover](../5.12-Project-Handover/).
