---
title: "System Security & Stress Testing"
date: "2026-06-15"
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Objectives

One of the biggest challenges for the Enterprise IoT Cloud Dashboard is handling high-frequency telemetry data safely. Before finalizing the project, we must harden the infrastructure and ensure the backend remains stable under enterprise-level traffic.

In this section, we will simulate the following scenarios:

1. Apply basic API rate limiting to reduce abuse and DDoS-like traffic.
2. Increase the frequency of telemetry requests to test the backend under load.
3. Monitor EC2 metrics and CloudWatch logs to confirm the service remains healthy.

#### Step 1: Apply rate limiting

We will configure the FastAPI application so that it rejects excessive requests from a single source.

1. Open the backend source code and add middleware or dependency-based rate limiting.
2. Set a practical threshold such as `100 requests/minute` per IP.
3. Restart the FastAPI service so the rules take effect.

<!-- Insert screenshot: rate limiting logic in backend service -->
> Placeholder for screenshot: FastAPI rate limiting configuration.

#### Step 2: Generate high-frequency traffic

A simple Python simulator can be used to create sustained traffic.

```python
import time

while True:
    post_telemetry()
    time.sleep(0.01)
```

This allows the team to observe how the API behaves when request volume increases sharply.

#### Step 3: Execute the stress test

Run the updated simulator and observe the system response.

1. Start the simulator from the terminal.
2. Watch whether requests return `200 OK` at first and then `429 Too Many Requests` after the threshold is exceeded.
3. Confirm the application remains available and logs the events properly.

<!-- Insert screenshot: terminal output or Postman results during stress test -->
> Placeholder for screenshot: stress-test output from the simulator.

#### Step 4: Observe AWS monitoring metrics

The final validation step is to inspect the monitoring layer.

1. Open the EC2 instance in the AWS Console.
2. Review the **Monitoring** tab for CPU utilization and network activity.
3. Open CloudWatch and check that logs reflect both successful requests and handled overload events.

<!-- Insert screenshot: CloudWatch dashboard or EC2 monitoring chart -->
> Placeholder for screenshot: CloudWatch metrics for CPU and API errors.

#### Conclusion

This step demonstrates the system's resilience under stress and confirms that the backend and monitoring stack are ready for practical deployment.