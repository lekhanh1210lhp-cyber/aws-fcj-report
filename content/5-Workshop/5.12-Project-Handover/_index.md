---
title: "Project Handover"
date: "2026-07-28"
weight: 12
chapter: false
pre: " <b> 5.12. </b> "
---

# Project Handover

## Deliverables

Provide:

- source repository URL and a clear repository tree;
- `README.md` and `README.vi.md`;
- backend, frontend, and hardware run instructions;
- `.env.example` and `secrets.example.h` with placeholders;
- database migration or initialization instructions;
- test evidence from Section 5.8;
- CloudWatch evidence and clean-up status; and
- demo video link and team contribution summary.

## Operator checklist

1. Configure database, Wi-Fi, device ID, and API endpoint secrets.
2. Confirm EC2, RDS, IAM Role, and Security Group configuration.
3. Start `aws-iot-backend` and verify `/api/health`.
4. Start the frontend and confirm Vite proxy configuration.
5. Power the hardware and inspect Serial Monitor.
6. Run one telemetry and command/ACK smoke test.
7. Review CloudWatch logs, metrics, and alarms.

## Final review

- [ ] All 12 sections have matching English and Vietnamese pages.
- [ ] Commands, endpoints, service names, and screenshots are consistent.
- [ ] No placeholder remains in a claimed working configuration.
- [ ] No secret or private key is committed.
- [ ] The Hugo site builds and navigation works.
- [ ] Each technical section has been reviewed by another team member.

**Expected result:** A new operator can configure, run, validate, monitor, and safely clean up the project without relying on undocumented team knowledge.

This completes the AWS IoT Dashboard workshop.
