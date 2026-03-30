# Cloud Computing Overview (AWS & Azure)

## Overview
This document introduces cloud computing on AWS and Azure for beginners. It focuses on practical concepts, core services, service models, simple demos, and real-world examples to prepare students to build and run apps in the cloud.

---

## Simple Definition
Cloud computing is renting IT resources over the internet instead of owning them.

**Analogy:**
- On-premises → Buying a house  
- Cloud → Renting an apartment  

You:
- Skip buying hardware  
- Pay only for what you use  
- Scale or move easily  

---

## Why Cloud?

### Problem
To run a website, you need:
- Servers
- Storage
- Databases
- Networking
- Security
- Maintenance

On-premises setup:
- Time-consuming
- Expensive
- Requires expertise

### Solution
AWS and Azure provide all these as on-demand services:
- Provision infrastructure in minutes
- Built-in networking and security
- No hardware maintenance

---

## Core Cloud Services

### 1. Networking
Connect resources securely and route traffic.

**AWS Services:**
- Amazon VPC
- Elastic Load Balancing
- Amazon Route 53
- Amazon CloudFront

**Azure Services:**
- Azure Virtual Network (VNet)
- Azure Load Balancer / Application Gateway
- Azure DNS
- Azure Front Door / Azure CDN

**Example:**
- DNS routes users to your site  
- Load balancer distributes traffic  
- CDN caches content closer to users  

---

### 2. IAM (Identity and Access Management)
Control access to resources.

**AWS:**
- AWS IAM
- AWS IAM Identity Center (SSO)
- Amazon Cognito

**Azure:**
- Microsoft Entra ID (Azure AD)
- Azure RBAC
- Azure AD B2C

**Example:**
- Developer → Read-only storage access  
- Users → Login via Cognito / Azure AD  

---

### 3. Compute
Where applications run.

**AWS:**
- Amazon EC2
- AWS Lambda
- ECS / EKS

**Azure:**
- Azure Virtual Machines
- Azure Functions
- AKS / Container Instances

**Example:**
- Web API → EC2 / VM  
- Image processing → Lambda / Functions  

---

### 4. Storage & Databases
Store files and application data.

**AWS:**
- Amazon S3
- Amazon RDS
- DynamoDB

**Azure:**
- Azure Blob Storage
- Azure SQL Database
- Cosmos DB

**Example:**
- Images → S3 / Blob  
- Orders → RDS / SQL Database  
- Sessions → DynamoDB / Cosmos DB  

---

### 5. Operations
Monitoring, logging, automation.

**AWS:**
- CloudWatch
- CloudTrail
- AWS Config
- Systems Manager

**Azure:**
- Azure Monitor
- Log Analytics
- Application Insights
- Azure Policy
- Azure Automation

**Example:**
- Alert if CPU > 80%  
- Track configuration changes  
- Automate patching  

---

## Cloud Service Models

### IaaS (Infrastructure as a Service)
- You manage: OS, runtime, app  
- Provider manages: hardware  

**Examples:**
- AWS EC2  
- Azure Virtual Machines  

**Analogy:** Renting land  

---

### PaaS (Platform as a Service)
- You manage: app & data  
- Provider manages: OS, runtime, scaling  

**Examples:**
- AWS Elastic Beanstalk  
- Azure App Service  

**Analogy:** Renting a house  

---

### SaaS (Software as a Service)
- Provider manages everything  
- You use the application  

**Examples:**
- Microsoft 365  
- Salesforce  

**Analogy:** Staying in a hotel  

---

## Live Demo Ideas

### Option A: Launch a Server
- Create EC2 / Azure VM  
- Choose OS  
- Start and connect  

**Point:** Server created in minutes  

---

### Option B: Upload a File
- Upload to S3 / Blob Storage  
- Copy public URL  

**Point:** File accessible globally  

---

## Key Benefits
- Pay as you go  
- Scalable on demand  
- High availability  
- No hardware maintenance  
- Global access  

---

## Real-Life Use Cases

- **Netflix** → Global video streaming  
- **Banking Apps** → Secure transactions  
- **E-commerce** → Scalable websites and checkout systems  

---
