---
title: "Workshop"
weight: 1
chapter: false
pre: " <b> 2. </b> "
---

# Cloud-Sentinel: Building an AI & Serverless-based SOAR System

### Executive Summary

**Cloud-Sentinel** is a next-generation security solution applying the **SOAR** (Security Orchestration, Automation, and Response) model. The system leverages the power of **Generative AI** and **Event-Driven Architecture** on AWS to automate security incident detection and response processes with optimized costs.

Unlike conventional automation systems, Cloud-Sentinel builds a team of **AI Agents** operating on **Amazon Bedrock** to replace human intervention in analyzing and evaluating security incidents. The system is designed entirely on a **Serverless** model to ensure the lowest operational costs, incurring charges only when an actual incident occurs.

-----

### Agentic Learning Path

#### 1\. IaC Infrastructure Setup (Terraform)

Build the "playground" for the Agents. Use **Terraform** to define VPCs, IAM Roles, and the necessary permissions so that the AI can "read" and "understand" the AWS infrastructure.

#### 2\. Activating the Senses (Detection Layer)

Connect **GuardDuty** and **EventBridge** to create "neural signals," waking up the **Supervisor Agent** the moment a sign of intrusion is detected.

#### 3\. Building Agent Intelligence (Intelligence Layer)

This is the core of the workshop. You will learn how to:

  * Program the **Supervisor Agent** to coordinate state machines (Step Functions).
  * Build a **RAG Agent** connected to **Pinecone** to retrieve security knowledge.
  * Configure the **Advisor Agent** on **Bedrock** to draft solutions.

#### 4\. Response & Interaction (Action Layer)

Establish an approval flow between the **Advisor Agent** and real users via **Telegram**. Execute security commands (Isolate/Block) through **Lambda Triggers**.

#### 5\. Monitoring & Summary

Evaluate the effectiveness of the Agents via **CloudWatch**, optimize operational costs (\~14.46 USD/month), and follow instructions on how to clean up resources.

-----

### Learning Outcomes

  * **IaC Skills:** Proficiency in deploying complex infrastructure using Terraform.
  * **AI Skills:** Understand and implement RAG flows to apply to real-world problems.
  * **Security Mindset:** Master modern security operations following the SOAR model.

-----

#### Detailed Content

1.  [Module 1: Introduction & Setup](https://www.google.com/search?q=2.1-Workshop-overview)
2.  [Module 2: Monitoring & Detection](https://www.google.com/search?q=2.2-Detection-Layer)
3.  [Module 3: Intelligence & Orchestration](https://www.google.com/search?q=2.3-Intelligence-Orchestration)
4.  [Module 4: Response & Interaction](https://www.google.com/search?q=2.4-Response-Interaction)
5.  [Module 5: Monitoring & Summary](https://www.google.com/search?q=2.5-Summary-Cleanup)