---
title : "Introduction"
weight : 1 
chapter : false
pre : " <b> 2.1. </b> "
---

# Cloud-Sentinel

## 1. Cloud-Sentinel Project Overview

**Cloud-Sentinel** is a modern **SOAR (Security Orchestration, Automation and Response)** system designed to protect critical infrastructures (such as Banking/Finance) on the AWS platform. The system operates on an **Event-Driven** mechanism combined with **Multi-Agent AI** to solve the challenge of: **Fast Detection - Intelligent Analysis - Automated Response**.

### Problems Solved & Technical Value

  * **Challenge:** Transforming security operation processes from manual to automated to minimize the burden on administrators. Eliminating false alarms through cross-evaluation processes between AI Agents.
  * **Technical Solution:** We solve this problem by deploying an intelligent orchestration flow using **AWS Step Functions** combined with artificial intelligence from **Amazon Bedrock**. The system provides real-time incident analysis capabilities and automatically executes protective actions after specialist approval.

-----

## 2. Design Strategy & Cost Optimization

The project is designed following a fully **Serverless** model to optimize operational costs, ensuring you only pay for what you actually use (Pay-as-you-go).

  * **Estimated Total Cost:** \~14.46 USD/month.
  * **Maintenance Cost when no incidents occur:** Near zero thanks to the event-driven architecture.

### Optimization Highlights:

  * **Amazon GuardDuty:** Focuses only on analyzing EC2 VPC Flow Logs (estimated 6GB/month) instead of scanning all raw logs to control the budget at approximately \~6.90 USD.
  * **Cost-effective AI:** Uses Amazon Bedrock's **On-Demand** model for Agents, ensuring costs are only incurred when actual incident analysis requests occur.
  * **Transparent Infrastructure:** Using **Terraform** helps accurately list all resources and projected costs before deployment, completely eliminating "zombie" resources.

-----

## 3. Prerequisites

To ensure the experimentation process goes smoothly and cost-effectively, you need to prepare the following:

### A. Programming Tools & Infrastructure Management

  * **AWS CLI v2:** Installed and configured with an `access_key`/`secret_key` that has **AdministratorAccess** permissions.
  * **Terraform CLI:** Version 1.5 or higher to manage infrastructure professionally and consistently.
  * **Python 3.11+:** Necessary for packaging Lambda Runtimes and building logic for the AI Agents.

### B. AWS Service Configuration

  * **Amazon Bedrock:** Access the AWS Console in the **Singapore (ap-southeast-1)** region, go to the **Model Access** section, and activate permissions for **Claude** or **Llama** models.
  * **S3 Backend:** Manually create an S3 Bucket (e.g., `cloud-sentinel-tfstate-yourname`) to store the `terraform.tfstate` file, helping to keep the infrastructure state secure.

### C. External Integration

  * **Telegram:** Create a Bot via `@BotFather` and save the **Bot Token** to serve the Action Layer.
  * **Pinecone:** Register for an account (Free Tier) and obtain an **API Key** to serve the knowledge storage layer (Vector Database).

-----

## 4. Infrastructure Initialization with Terraform

In this module, participants will learn how to deploy a complete security infrastructure using Terraform. Using Terraform brings high consistency and reusability to the Cloud-Sentinel project.

**Basic commands to be used:**

1.  `terraform init`: Initialize the project and connect to the S3 Backend.
2.  `terraform plan`: Preview changes and estimate resources to be created.
3.  `terraform apply`: Execute the infrastructure deployment to AWS.

![Agentic Workflow](/25FALL/OJT/AWS_S2/cloud-sentinel/public/images/High_Level_System_Architecture.gif)