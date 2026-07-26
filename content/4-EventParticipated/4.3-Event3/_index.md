---
title: "Event 3"
date: "2025-09-09"
weight: 1
chapter: false
pre: " <b> 4.3. </b> "
---

# Report: “WORKSHOP: DATA SCIENCE ON AWS”

### Event Objectives

- Share AWS AI services
- Demonstrate deploying AI models using Amazon SageMaker
- Show how to deploy models and access them via APIs

### Speakers

- **Van Hoang Kha** - Cloud Solutions Architect, AWS User Group Leader
- **Bach Doan Vuong** - Cloud Developer Engineer, AWS Community Builder

### Highlights

#### **Introduction & the importance of Cloud in Data Science**

- Discussed the role of cloud computing in supporting data processing, training, and deploying AI models at scale.
- Cloud vs. On-premise:

  - Cloud: flexible scalability, rapid deployment, lower operating costs, easy integration.
  - On-premise: high upfront investment, harder to scale, complex maintenance.

- Cloud (especially **AWS**) provides a strong foundation for the Data Science pipeline — from collection, storage, and processing to training and deploying AI models.

#### **AI Layers on AWS**

AWS organizes the AI ecosystem into **three layers**, helping users choose the right level of management based on skills and goals:

**1. AI Services (Fully Managed Layer)**

> _For users who want to apply AI without deep Machine Learning knowledge._

- Fully managed AI services that have been pre-trained by AWS.
- Users can call APIs to use them directly in applications.
- Examples:

  - **Amazon Comprehend:** Natural language processing (NLP)
  - **Amazon Translate:** Neural machine translation
  - **Amazon Textract:** Extract data from documents and invoices
  - **Amazon Rekognition:** Image and video recognition
  - **Amazon Polly:** Text-to-speech
  - **Amazon Bedrock:** Access to foundation models (e.g., Claude, Titan, Mistral)

👉 Benefits: Fast deployment, no model training needed, cost scales with usage.

**2. ML Services (Semi-managed Layer)**

> _For Data Scientists and ML Engineers who want to build, train, and deploy ML models with more customization._

- **Amazon SageMaker** is at the center of this layer: it provides tools to build, train, and deploy ML models.
- Key features:

  - **Data Wrangler:** Visual data cleaning and processing.
  - **Feature Store:** Manage features across models.
  - **AutoML (SageMaker Autopilot):** Automated model training.
  - **Model Registry & Monitoring:** Track and manage models after deployment.

👉 Benefits: Full control over the ML pipeline, customizable algorithms, training environments, and deployment workflows.

**3. AI Infrastructure (Self-managed Layer)**

> _For organizations or experts who want to fully manage AI/ML infrastructure to optimize cost or performance._

- Users can build training environments using core AWS infrastructure services:

  - **Amazon EC2 / GPU Instances (P5, G6, Inferentia):** Train large custom models.
  - **Amazon EKS / ECS:** Run ML workloads in containers or Kubernetes.
  - **AWS Lambda:** Small-scale data processing or serverless inference.
  - **Amazon S3 / EFS:** Store data and models.

👉 Benefits: Maximum flexibility and control, but requires higher technical expertise.

#### Popular AWS AI Services to Support Students During Model Training

**1. Amazon SageMaker**

- Integrated development environment (SageMaker Studio) for the full ML lifecycle:

  - Data preparation
  - Model training
  - Result tracking
  - Deploying endpoints for API inference

- Supports AutoML, GPU training, model monitoring, and CI/CD for AI models.

**2. Amazon Comprehend**

- NLP service to analyze, understand, and classify natural language.

- Main capabilities:

  - Sentiment analysis
  - Entity recognition
  - Text classification
  - Automated labeling
  - Language detection

- Use cases:

  - Intelligent document processing
  - Bulk email analysis to detect positive/negative responses
  - Customer sentiment and behavioral analysis
  - Contact center analytics
  - Information extraction and validation

**3. Amazon Translate**

- Neural machine translation service.
- Supports over 75 languages with high accuracy and easy integration.
- Applications:

  - Multilingual websites
  - Automatic content translation in apps
  - Multilingual chatbot support and analytics

**4. Amazon Textract**

- Automatically extract text and structured data from images, documents, and forms.
- Used for processes like record digitization, invoice processing, and automatic data entry.

#### AWS Data Science Pipeline Overview

1. Data collection & storage: Amazon S3, AWS Data Exchange
2. Data preprocessing: AWS Glue, Lambda, Athena
3. Model training: SageMaker (train, tune, evaluate)
4. Model deployment: SageMaker Endpoint / Lambda + API Gateway
5. Monitoring & optimization: CloudWatch, Model Monitor

#### **Demo 1: Designing an AI Training Workflow with a Drag-and-Drop Interface (No-Code/Low-Code)**

- **Goal:** Show how to build an AI training pipeline without heavy coding.
- **Tools:** Amazon SageMaker Studio / SageMaker Canvas
- **Demo steps:**

  1. Prepare the dataset and upload it to Amazon S3.
  2. Use SageMaker's drag-and-drop interface to:

     - Choose data sources, training algorithms, and parameters.
     - Design a pipeline including data cleaning, training, validation, and deployment steps.

  3. Visually monitor training progress and model results (accuracy, confusion matrix, metrics, etc.).

- **Key message:** Students and developers can quickly create AI workflows without complex code, speeding up research and experimentation.

#### **Demo 2: Deploying an AI Service and Accessing It Via API/Website**

- **Goal:** Demonstrate how to deploy an AI model so users can access it in practice.
- **Tools:** Amazon SageMaker Endpoint, API Gateway, and Lambda.
- **Demo steps:**

  1. Deploy the trained model to a SageMaker Endpoint.
  2. Integrate the endpoint with API Gateway to create a public REST API.
  3. Provide a web route or API URL for users to send requests (e.g., submit text for sentiment analysis or translation).
  4. Show how to present results visually (UI demo or Postman/API test).

- **Key message:** Demonstrates how AWS supports moving AI from research to production — easy to share, scale, and commercialize.

#### Discussion: Performance & Cost (Cloud vs. On-premise)

| Criteria                  | Cloud (AWS)                           | On-premise                  |
| ------------------------- | ------------------------------------- | --------------------------- |
| **Scalability**           | Easily scale resources as needed      | Limited by fixed hardware   |
| **Cost**                  | Pay-as-you-go                         | High upfront investment     |
| **Deployment**            | Automated, fast                       | Manual, time-consuming      |
| **Maintenance**           | Managed by AWS                        | User is responsible         |
| **Suitable for students** | ✅ Free Tier available, easy to learn | ❌ Harder to access, costly |

#### Conclusion

- AWS provides a comprehensive AI ecosystem from infrastructure to application layers, suitable for everyone — from students learning AI to enterprises deploying at scale.

### Event Experience

Attending the workshop “AI Services on AWS for Data Science” was very valuable. It helped me better understand the role of cloud in Data Science and how AWS supports training, deploying, and accessing AI models.

#### Key takeaways from expert speakers

- Speakers emphasized the importance of cloud in data processing and model training.
- Gained a clear understanding of the three AI layers on AWS: AI-managed services, ML services (SageMaker), and AI frameworks.

#### Hands-on technical experience

- **Demo 1:** Designed an AI workflow using SageMaker Canvas drag-and-drop to train models without code.
- **Demo 2:** Deployed an AI model as a service accessible via API or link.

#### Using modern tools

- Learned about key AI services: **Amazon Comprehend**, **Translate**, and **Textract**.
- Understood how these services support NLP, machine translation, and intelligent data extraction.

#### Networking and discussion

- Interacted with experts and fellow students interested in AI & Cloud.
- Discussed cost, performance (Cloud vs On-premise), and how to optimize SageMaker usage.

#### Lessons learned

- Cloud is a foundational platform for modern Data Science workflows.
- AWS provides tools for every AI skill level — from no-code to fully managed deployments.
- Gained clearer knowledge of how to bring AI models into real products using AWS services.

#### Some photos from the event