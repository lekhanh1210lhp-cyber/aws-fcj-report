---
title: "Testing REST API (Data Ingestion)"
date: "2026-06-15"
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

#### Objectives

Once the FastAPI backend is connected to PostgreSQL, it is essential to verify that real requests can be accepted, validated, and stored correctly. This section simulates the role of IoT devices sending telemetry data to the API.

We will focus on two main aspects:

1. **Data ingestion:** Confirm that valid requests are accepted and saved to the database.
2. **Data validation:** Ensure malformed or incomplete payloads are rejected with clear error messages.

#### Preparation

Before testing, make sure the backend service is running on the EC2 instance and that the database connection is working. The API endpoint should be accessible from your local machine.

<!-- Insert screenshot: Postman request setup pointing to EC2 public IP -->
> Placeholder for screenshot: Postman request configured for the telemetry endpoint.

#### Step 1: Send a valid payload

Use Postman to submit a sample telemetry request.

1. Open Postman and create a new request.
2. Set the method to **POST**.
3. Enter the EC2 endpoint, for example `http://<EC2-Elastic-IP>:8000/telemetry`.
4. In the body tab, select **raw** and **JSON**, then send the following payload:

```json
{
  "building_id": "HN_01",
  "temperature": 25.4,
  "humidity": 60,
  "light": 450,
  "device_status": "active"
}
```

Expected result: the backend should respond with `200 OK` or `201 Created`, and the record should be stored in the database.

#### Step 2: Send an invalid payload

To verify validation, send a request with an invalid field type.

```json
{
  "building_id": "HN_01",
  "temperature": "too_hot",
  "humidity": 60
}
```

Expected result: the service should reject the request with `422 Unprocessable Entity` and return a validation error.

#### Step 3: Verify through CloudWatch

After sending both requests, inspect the backend logs in CloudWatch.

1. Open the AWS Console and navigate to **CloudWatch**.
2. Open the log group for the EC2 backend.
3. Confirm that successful requests and validation failures are being recorded.

<!-- Insert screenshot: CloudWatch logs showing successful and rejected API requests -->
> Placeholder for screenshot: CloudWatch log view for telemetry requests.

#### Conclusion

This step validates the core behavior of the IoT API: it should accept correct telemetry data, reject invalid input, and leave a clear trace in the monitoring system.