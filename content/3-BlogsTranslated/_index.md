---
title: "Translated Blogs"
date: "2025-09-09"
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

### [Blog 1 - How PostNL processes billions of IoT events with Amazon Managed Service for Apache Flink](3.1-blog1/)

This blog details how **PostNL** modernized its IoT data stream processing platform by migrating to **Amazon Managed Service for Apache Flink** to handle billions of IoT events from tracked assets. It highlights the challenges of real-time IoT data at scale, particularly handling late events and event time semantics. By utilizing Flink's **ProcessFunction API** for fine-grained control, PostNL successfully built a scalable, robust, and cost-effective stream processing solution for its logistics operations.

### [Blog 2 - Optimize industrial IoT analytics with Amazon Data Firehose and Amazon S3 Tables with Apache Iceberg](3.2-blog2/)

This blog demonstrates how to build a scalable, low-code edge-to-cloud data ingestion framework for **industrial IoT (IIoT)** analytics. It explains how to collect real-time sensor data using **AWS IoT Greengrass** at the edge, stream it through **Amazon Data Firehose**, and optimize storage using **Amazon S3 Tables** with the **Apache Iceberg** format. This architecture enables efficient, performant, and cost-effective querying and analysis via **Amazon Athena** without requiring complex infrastructure setup.

### [Blog 3 - Process millions of observability events with Apache Flink and write directly to Prometheus](3.3-blog3/)

This blog introduces a new **Apache Flink connector for Prometheus**, enabling Flink applications to write preprocessed time-series data directly to **Amazon Managed Service for Prometheus**. It explores how preprocessing raw observability events from highly distributed assets (like IoT devices and connected cars) with **Amazon Managed Service for Apache Flink** helps reduce data cardinality and frequency. This approach allows organizations to build more efficient and scalable real-time dashboards and alerts in **Amazon Managed Grafana**.