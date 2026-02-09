
# Jupiter Static Web Application  
**Production-Aligned Container Deployment on AWS ECS using Docker and Amazon ECR**

---

## Overview

This project demonstrates the full lifecycle deployment of a containerized static web application on AWS using Docker, Amazon Elastic Container Registry (ECR), and Amazon Elastic Container Service (ECS) with AWS Fargate.

The objective was to design and operate a production-aligned container workflow that emphasizes immutability, security, scalability, and operational reliability. The solution includes local image validation, private container registry management, multi-AZ networking, controlled public ingress, and automated container orchestration without managing servers.

---

## Architecture

![Architecture Diagram](https://github.com/user-attachments/assets/c72f24e6-721b-48dd-9af8-169b6b905da3)

### Request and Traffic Flow

1. End users access the application through an internet-facing Application Load Balancer (ALB)  
2. The ALB forwards HTTP traffic to ECS tasks using IP-based target registration  
3. ECS runs Docker containers in private subnets using AWS Fargate  
4. Container images are pulled securely from Amazon ECR at runtime  
5. Private subnets access external services through NAT Gateways  

This architecture enforces workload isolation while exposing a single, controlled ingress point to the public internet.

---

## Technology Stack

### Containerization
- Docker  
- Dockerfile (Amazon Linux base image with Apache HTTP Server)

### AWS Services
- Amazon ECS (Fargate)  
- Amazon ECR  
- Application Load Balancer  
- Amazon VPC (Three-Tier Architecture)  
- AWS IAM  
- Amazon CloudWatch  

### Networking
- Public and Private Subnets across multiple Availability Zones  
- Internet Gateway  
- NAT Gateways  
- Security Groups  

---

## Container Build and Local Validation

The static website was containerized using Docker and validated locally prior to promotion to a registry.

- Docker image built from source using a custom Dockerfile  
- Container executed locally with explicit port mapping  
- Application functionality verified via localhost  
- Container lifecycle operations tested (start, stop, removal)  

Local validation ensured predictable runtime behavior before deployment to AWS.

---

## Image Registry and Artifact Management

- Provisioned a private Amazon ECR repository to store container images  
- Applied registry-ready tagging to support versioned deployments  
- Authenticated Docker to ECR using AWS CLI credentials  
- Pushed the image as an immutable deployment artifact  

Amazon ECR serves as the single source of truth for images consumed by ECS services.

---

## Network Infrastructure

A custom Three-Tier VPC was designed and built from scratch to reflect real-world production networking patterns.

### Network Design
- Public subnets for external-facing load balancers  
- Private application subnets for ECS tasks  
- Private data subnets reserved for future backend services  
- Internet Gateway for inbound and outbound public traffic  
- NAT Gateways per Availability Zone for secure outbound access from private subnets  

This design supports high availability, fault tolerance, and strict traffic segmentation.

---

## Security Model

### Identity and Access Management
- `ecsTaskExecutionRole` grants ECS permission to pull images from ECR and publish logs  
- No secrets or credentials embedded in container images or task definitions  

### Network Security
- ALB Security Group allows inbound HTTP traffic from the internet  
- Container Security Group permits inbound traffic only from the ALB  
- No SSH access to containers  
- No public IPs assigned to ECS tasks  

Security was enforced by default, following the principle of least privilege.

---

## Load Balancing and Traffic Management

- Internet-facing Application Load Balancer deployed in public subnets  
- IP-based Target Group configured for ECS Fargate compatibility  
- Health checks enabled to ensure traffic is routed only to healthy tasks  
- ECS dynamically registers and deregisters tasks with the load balancer  

The ALB acts as the single controlled entry point to the application.

---

## Container Orchestration

### ECS Cluster
- Fargate-based ECS cluster with no EC2 instance management  
- Service-linked role initialized via AWS CLI to resolve initial deployment failure  

### Task Definition
- Defines container image, CPU, memory, and port mappings  
- Uses `awsvpc` networking mode for ENI-based isolation  
- Pulls images directly from Amazon ECR  

### ECS Service
- Maintains the desired number of running tasks  
- Automatically replaces failed or unhealthy tasks  
- Integrated with Application Load Balancer and Target Group  
- Service auto scaling enabled based on load  

The ECS Service fulfills a role comparable to an Auto Scaling Group for containerized workloads.

---

## Observability and Operations

- ECS service events provide visibility into deployments and failures  
- Container logs streamed to Amazon CloudWatch  
- Health checks enforce traffic integrity  
- Scaling occurs without application downtime  

Operational visibility and resilience were considered first-class requirements.

---

## Failure Encountered and Resolution

### Issue
ECS cluster creation failed with the error:  
`Unable to assume the service linked role`

### Root Cause
The required ECS service-linked role already existed, resulting in a CloudFormation race condition during initial provisioning.

### Resolution
- Initialized the service-linked role using AWS CLI  
- Deleted the failed CloudFormation stack  
- Re-created the ECS cluster successfully  

This validated infrastructure-level troubleshooting and recovery.

---

## Key Concepts Demonstrated

- Container lifecycle and artifact management  
- Secure promotion of images to a private registry  
- Private networking with controlled public ingress  
- Serverless container orchestration using ECS Fargate  
- Load-balanced traffic routing with health checks  
- IAM least-privilege design  
- Auto scaling and self-healing services  

---

## Professional Takeaway

This project demonstrates the ability to design, deploy, secure, and operate a containerized application using AWS managed services while applying production-aligned DevOps patterns and operational best practices.

---

## Future Enhancements

- Automate image builds and pushes using a CI pipeline  
- Provision infrastructure using Terraform or CloudFormation  
- Enable HTTPS using ACM and Route 53  
- Implement blue-green or rolling deployment strategies  

---
