---
title: "System Security & Stress Testing"
date: "2026-06-15"
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Target

One of the biggest challenges for an Enterprise IoT Cloud Dashboard is handling high-frequency telemetry data securely. Before finalizing the project, we must harden the cloud infrastructure and ensure the backend can handle enterprise loads. 

In this section, we will simulate the following scenario:

1.  Implement basic rate limiting in FastAPI to prevent DDoS or spam telemetry.
2.  Configure the IoT Simulator to send high-frequency data to the EC2 backend.
3.  Monitor the EC2 CPU and CloudWatch logs to verify system stability under heavy load.

#### Implementation Steps

**Step 1: Implement API Rate Limiting**

We need to confirm that our FastAPI backend can defend against request spamming.

1.  Access your EC2 instance or local backend source code.
2.  Configure the rate limiting logic in your FastAPI application.
    - _Example:_ Limit incoming telemetry data to `100 requests per minute` per IP.
3.  Restart the `systemctl` backend service to apply the new security rules.

![API Rate Limiting Setup](/images/5-Workshop/5.6-Stress-Testing/01_Rate_Limit.png)

**Step 2: Configure IoT Simulator for Stress Testing**

We will modify the Python script to act as a stress-testing tool.

1.  On your computer, open the Python Simulator project.
2.  Adjust the threading and delay configurations in the script to generate high-frequency data.
    ```python
    # Example snippet for high-frequency test
    import threading
    import time

    def send_spam_telemetry():
        while True:
            # Send POST request rapidly
            post_telemetry()
            time.sleep(0.01) # Very short delay for stress testing
    ```
3.  Save the changes to the simulator script.

**Step 3: Execute the Stress Test**

Now, we will bombard the backend with high-frequency data to see how it reacts.

1.  Run the updated Python Simulator from your Terminal.
2.  **Observe the result in the console:**
    - Initially, requests will return `200 OK`.
    - Once the rate limit is exceeded, the backend should automatically reject the spam telemetry, returning `429 Too Many Requests` or dropping the connections gracefully.

![Stress Testing Execution](/images/5-Workshop/5.6-Stress-Testing/02_Stress_Test.png)

**Step 4: Monitor EC2 CPU & CloudWatch (The Verification)**

The system must remain stable despite the high-frequency data attack.

1.  Access the **AWS Console**, navigate to **EC2**, and select your backend instance.
2.  Go to the **Monitoring** tab and review the **CPU Utilization** graph.
    - Ensure the CPU spikes but does not crash the instance (properly sized at t2.micro/t3.micro).
3.  Switch to the **CloudWatch Console** -> Select **Log groups**.
4.  Verify that CloudWatch logs accurately capture all API successes (200) and gracefully handled errors (429/500).

![CloudWatch CPU Monitoring](/images/5-Workshop/5.6-Stress-Testing/03_CloudWatch_CPU.png)

#### Conclusion

You have successfully stress-tested and secured the IoT Dashboard!

- **Infrastructure Secured:** The backend is protected against basic DDoS vulnerabilities and spam telemetry.
- **Enterprise Load Ready:** The system handled the high-frequency data efficiently.
- **Code Freeze:** With stability verified, you are now ready for the final Code Freeze and project hand-off.