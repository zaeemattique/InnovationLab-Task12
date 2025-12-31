# Deploying a Node.js Application on AWS ECS using GitHub Actions

## Overview

This project demonstrates a complete CI/CD workflow for deploying a **Node.js application** on **AWS Elastic Container Service (ECS) with Fargate**, using **Docker**, **Amazon ECR**, and **GitHub Actions**.  
The pipeline automatically builds a Docker image, pushes it to ECR, and deploys it to ECS whenever code is pushed to the `main` branch.

---

## Architecture

![alt text](https://raw.githubusercontent.com/zaeemattique/InnovationLab-Task12/refs/heads/main/Task12%20Architecture%20Diagram.jpg)

The deployment architecture consists of:

- Custom VPC with public and private subnets across multiple Availability Zones
- Internet Gateway and NAT Gateways for outbound connectivity
- Application Load Balancer (ALB) for traffic distribution
- Amazon ECR for Docker image storage
- Amazon ECS (Fargate) for container orchestration
- GitHub Actions for CI/CD automation

---

## Networking Infrastructure

- **VPC**
  - CIDR: `10.0.0.0/16`

- **Subnets**
  - Public Subnet A (`us-west-2a`) – `10.0.1.0/24`
  - Private Subnet A (`us-west-2a`) – `10.0.2.0/24`
  - Public Subnet B (`us-west-2b`) – `10.0.3.0/24`
  - Private Subnet B (`us-west-2b`) – `10.0.4.0/24`

- **Gateways & Routing**
  - Internet Gateway attached to VPC
  - NAT Gateway in each public subnet
  - Public route table routes traffic to IGW
  - Private route tables route traffic to NAT Gateways

---

## Application & Docker Configuration

- Simple Node.js application listening on **port 5000**
- `node_modules` removed from repository (installed during build)
- Dockerfile configuration:
  - Base image: `node:18-alpine`
  - Working directory: `/usr/src/app`
  - Install dependencies using `npm install`
  - Copy application files
  - Expose port `5000`
  - Start app using `node index.js`

---

## AWS Credentials Management

- AWS credentials stored securely as **GitHub Repository Secrets**
  - `AWS_ACCESS_KEY_ID`
  - `AWS_SECRET_ACCESS_KEY`
- Secrets are injected into the workflow using `configure-aws-credentials@v2`
- Prevents credentials from appearing in logs or source code

---

## Load Balancer & Target Group

- **Target Group**
  - Type: IP
  - Protocol: HTTP
  - Port: 5000
  - Health checks: Default

- **Application Load Balancer**
  - Internet-facing
  - IPv4
  - Listener: HTTP on port 5000
  - Routes traffic to ECS target group
  - Security Group allows inbound traffic on port 5000

---

## Amazon ECR

- Repository created to store Docker images
- Image settings:
  - Mutable tags
  - AES-256 encryption
- Images are tagged using the **Git commit SHA** for traceability

---

## ECS Configuration

### ECS Cluster
- Launch type: **Fargate only**
- Container Insights enabled

### Task Definition
- Family name defined for versioning
- CPU: 2 vCPU
- Memory: 4 GB
- Container:
  - Name: `NodeJS-App`
  - Image pulled from ECR
  - Port mapping: `5000/TCP`
- Task execution and task roles configured

### ECS Service
- Desired tasks: 2
- Deployment type: Rolling updates
- Integrated with ALB and target group
- Automatically registers/deregisters targets

---

## GitHub Actions CI/CD Pipeline

The workflow is triggered on every push to the `main` branch and performs the following steps:

1. Checkout repository code
2. Configure AWS credentials
3. Authenticate Docker with Amazon ECR
4. Build Docker image using the Dockerfile
5. Push image to ECR
6. Fetch the currently running ECS task definition
7. Remove unsupported fields (e.g., `enableFaultInjection`)
8. Inject the new image into the task definition
9. Register a new task definition revision
10. Deploy the new revision to the ECS service
11. Wait for service stability

---

## Deployment & Testing

- ECS service maintains desired task count during deployments
- Application is accessed using the **ALB DNS name**
- Health checks ensure only healthy containers receive traffic

---

## Rollback Strategy

Two rollback methods are supported:

### Fast Rollback (AWS Console)
- Update ECS service
- Select a previous task definition revision
- Force deployment

### Clean Rollback (Git-Based)
- Revert to a previous stable commit
- Push changes to `main`
- GitHub Actions automatically redeploys the application

---

## Key Learnings & Observations

- ECS target groups initially show no registered targets (Fargate registers them dynamically)
- Task definitions can be reused and versioned efficiently
- GitHub Actions provides a clean alternative to CodePipeline
- Docker image tagging using commit SHAs ensures full traceability
- Infrastructure can be managed either manually or via IaC tools like Terraform

---

## Technologies Used

- AWS ECS (Fargate)
- Amazon ECR
- Application Load Balancer
- Docker
- GitHub Actions
- Node.js
- AWS CLI

---

## Author

**Zaeem Attique Ashar**  
Cloud Intern
