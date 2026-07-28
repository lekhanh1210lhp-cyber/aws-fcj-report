---
title: "Event 3"
date: "2026-07-28"
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

# Report on “First Cloud AI Journey: Application Security, Cloud Practitioner Strategy, and SLA Monitoring”

### Purpose of the Event

- Explore how to secure web applications using autonomous AI agents with AWS Security Agent (Frontier Agent).
- Learn a strategic roadmap and practical techniques to conquer the AWS Certified Cloud Practitioner (CLF-C02) exam.
- Understand Service Level Agreements (SLAs) and how to transition from basic infrastructure monitoring to user-centric monitoring.

### Speakers

- **Thinh Nguyen** – DevOps / DevSecOps / Cloud Engineer at Styl Solutions.
- **Ngo Le Tan Huy** – Presenter on AWS Cloud Practitioner Exam Strategy.
- **Nguyễn Huỳnh Sơn** – Infrastructure Support Engineer at Endava, Member of AWS Student Builder Group HUFLIT.

### Key Highlights

## Main Content

1. **Securing Web Apps with AWS Security Agent**
   - Traditional manual penetration testing is time-consuming, expensive ($5k–$20k per engagement), and inconsistent.
   - AWS Security Agent (Frontier Agent) uses autonomous reasoning powered by Amazon Bedrock across the full application lifecycle: Design Review, Code Review, and Active Pentesting.
   - **Design Review:** Analyzes architecture docs or Terraform code against frameworks like PCI DSS, NIST CSF, and AWS Well-Architected (Free Tier: 200 reviews/mo).
   - **Code Review:** Integrates directly into GitHub/GitLab PRs to comment on vulnerabilities, detect secrets, and suggest automated PR fixes (Free Tier: 1,000 PR reviews/mo).
   - **Automated Pentesting:** Executes multi-step exploit chains with verifiable proof, though limitations exist around MFA/biometrics and business-logic flaws.

2. **Strategic Roadmap to AWS Cloud Practitioner (CLF-C02)**
   - **Exam Overview:** 65 multiple-choice questions, 90 minutes (+30 minutes for non-native English speakers), passing score of 700/1000, valid for 3 years.
   - **Domain Weighting:** Cloud Concepts (24%), Security and Compliance (30%), Cloud Technology and Services (34%), and Billing, Pricing, and Support (12%).
   - **Preparation Strategy:** Map services with keywords (e.g., "Decouple" → SQS), review test mistakes thoroughly, and practice hands-on using the AWS Free Tier.
   - **Tips & Tricks:** Use the elimination technique to remove fake or irrelevant services, avoid overthinking simple foundational questions, and flag uncertain questions for later review.

3. **From SLA to Monitoring What Really Matters**
   - Service Level Agreements (SLAs) define expected service levels between providers and customers for accountability and risk management.
   - **The Monitoring Gap:** A "green" infrastructure dashboard (low CPU, 200 OK health checks) does not guarantee a happy user experience if database connections fail during user login.
   - **The Monitoring Pyramid:** Expand visibility across Cloud Provider → Infrastructure → Application → Business Metrics → Customer Experience.
   - **Alerting Flow:** Track custom application metrics (e.g., login failure rate) in CloudWatch and send early alerts via Amazon SNS to Slack or email before customers complain.

### Key Learnings

- **Design & Operational Mindset**
  - "Everything fails all the time, so plan for failure and nothing fails" (Dr. Werner Vogels, CTO of Amazon).
  - Healthy infrastructure metrics alone do not mean a healthy user experience; monitoring must focus on end-to-end customer journeys.

- **Technical Architecture**
  - Autonomous AI security agents complement human security teams by automating PR scans, architecture compliance checks, and exploit verification at a fraction of the cost.
  - Implement top-down monitoring by tracking application-level metrics and business outcomes alongside server health.

- **Exam & Learning Strategy**
  - Studying for foundational certifications requires understanding core service use cases and eliminating obvious distractor answers rather than memorizing complex configurations.
  - Recognize the limitations of automated security tools, such as authentication blocks (MFA, mTLS) and complex business logic context.

### Application to Work

- **In infrastructure & operations**:
  - Shift from pure infrastructure monitoring to user-centric metric tracking (e.g., login success rate, checkout completion).
  - Configure custom CloudWatch Alarms connected to SNS topics to receive proactive Slack or email notifications during operational anomalies.

- **In software development & AI**:
  - Integrate automated code security reviews into CI/CD pipelines to catch secrets and code vulnerabilities early in Pull Requests.
  - Utilize free resources like AWS Skill Builder and AWS Free Tier to build hands-on experience alongside theoretical certification preparation.

- **In security & compliance**:
  - Use automated design security reviews on Terraform code to ensure infrastructure compliance with PCI DSS, NIST CSF, and AWS Well-Architected standards.
  - Apply the Shared Responsibility Model in daily tasks: AWS manages security *of* the cloud, while developers are responsible for security *in* the cloud and customer satisfaction.

#### Event Photos

<img src="/images/4-EventParticipated/image_3.jpg" alt="Event 3" width="600"/>