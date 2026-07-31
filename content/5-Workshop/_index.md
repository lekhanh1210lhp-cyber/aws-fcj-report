---
title: "Workshop"
date: "2026-07-28"
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

This hands-on workshop builds the documented end-to-end path for one physical device, `room_01`: YOLO UNO reads a DHT20 and an analog light sensor, FastAPI stores telemetry and commands in Amazon RDS for PostgreSQL, React displays the data and creates commands, and the device acknowledges completed fan, light, and curtain actions.

## Objectives and final result

By the end of the workshop, you will be able to:

- deploy a FastAPI backend on Amazon EC2 with an EBS root volume;
- connect EC2 privately to Amazon RDS for PostgreSQL in a DB Subnet Group;
- integrate YOLO UNO telemetry, command polling, execution, and ACK;
- connect a local React + Vite + TypeScript dashboard;
- validate `Pending` to `Executed` command transitions; and
- collect EC2, RDS, and backend operational evidence in CloudWatch.

The expected result is a reproducible prototype for `room_01`, not an enterprise BMS. Allow **8-12 focused hours** for the workshop after the application source and AWS account are ready.

## Workshop map

1. [5.1 Workshop Overview](5.1-Workshop-overview/)
2. [5.2 Prerequisites](5.2-Prerequisites/)
3. [5.3 Architecture and Service Design](5.3-Architecture-and-Service-Design/)
4. [5.4 AWS Infrastructure Setup](5.4-AWS-Infrastructure-Setup/)
5. [5.5 Backend Deployment and Database Integration](5.5-Backend-and-Database/)
6. [5.6 Hardware Integration](5.6-Hardware-Integration/)
7. [5.7 Frontend Integration](5.7-Frontend-Integration/)
8. [5.8 End-to-End Testing and Validation](5.8-End-to-End-Testing/)
9. [5.9 Monitoring with CloudWatch](5.9-CloudWatch-Monitoring/)
10. [5.10 Cost, Security and Clean-up](5.10-Cost-Security-Cleanup/)
11. [5.11 Results, Challenges and Future Improvements](5.11-Results-Challenges-Future/)
12. [5.12 Project Handover](5.12-Project-Handover/)

## Architecture

![AWS IoT Monitoring and Control Dashboard architecture](/images/5-Workshop/5.3-architecture/aws_diagram.png)

*Figure 5-1. The source-repository architecture places the local dashboard user, React frontend, and YOLO UNO outside AWS; EC2 and RDS inside the VPC; and IAM and CloudWatch as AWS account/regional services outside the VPC boundary.*

The deployed AWS services are **Amazon EC2, Amazon EBS, Amazon RDS for PostgreSQL, Amazon VPC, subnets, Security Groups, an AWS IAM Role, Amazon CloudWatch, and CloudWatch Alarms**. AWS IoT Core, Lambda, API Gateway, S3, SNS, ECS/ECR, Cognito, CloudFront, and DynamoDB are not part of the current architecture.

Start with [Workshop Overview](5.1-Workshop-overview/).
