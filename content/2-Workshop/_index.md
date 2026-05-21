---
title: "Workshop"
weight: 1
chapter: false
pre: " <b> 2. </b> "
---

# Cloud-Sentinel: Building an AI & Serverless-based SOAR System

### Executive Summary
**Cloud-Sentinel** is a next-generation security solution implementing the **SOAR** (Security Orchestration, Automation, and Response) model. The system leverages **Generative AI** and an **Event-Driven Architecture** on AWS to automate security detection and response workflows cost-effectively.

Unlike traditional automation systems, Cloud-Sentinel builds a team of **AI Agents** running on **Amazon Bedrock** to perform analysis and incident assessment tasks. The system is designed as fully **Serverless** to minimize operational costs, incurring charges only when incidents occur.

---

### Learning Milestones (Agentic Learning Path)

#### 1. IaC Infrastructure Setup (Terraform)
Build the Agents' playground. Use **Terraform** to define the VPC, IAM Roles, and necessary permissions so AI can "read" and "understand" the AWS infrastructure.

#### 2. Detection Activation (Detection Layer)
Connect **GuardDuty** and **EventBridge** to create "neural signals" that wake the **Supervisor Agent** when intrusion signs appear.

#### 3. Building Agent Intelligence (Intelligence Layer)
This is the workshop core. You'll learn to:
- Program the **Supervisor Agent** to orchestrate state machines (Step Functions).
- Build a **RAG Agent** connected to **Pinecone** for security knowledge retrieval.
- Configure an **Advisor Agent** on **Bedrock** to draft remediation plans.

#### 4. Response & Interaction (Action Layer)
Set up approval flows between the **Advisor Agent** and real users via **Telegram**. Execute security actions (Isolate/Block) through **Lambda Triggers**.

#### 5. Monitoring & Recap
Evaluate Agent performance via **CloudWatch**, optimize operational costs (~14.46 USD/month), and learn resource cleanup procedures.

---

### Outcomes
- **IaC skills:** Deploy complex infrastructure with Terraform.
- **AI skills:** Understand and implement RAG workflows for practical problems.
- **Security mindset:** Master modern security operations following the SOAR model.

---

#### Detailed Contents
1. [Module 1: Introduction & Setup](2.1-Workshop-overview/_index.md)
2. [Module 2: Detection & Monitoring](2.2-Detection-Layer/)
3. [Module 3: Intelligence & Orchestration](2.3-Intelligence-Orchestration/)
4. [Module 4: Response & Interaction](2.4-Response-Interaction/)
5. [Module 5: Monitoring & Summary](2.5-Summary-Cleanup/)
