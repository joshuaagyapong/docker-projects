# Jupiter Static Web Application  
Containerized Deployment on AWS ECS using Docker and Amazon ECR

---

## Overview

This project demonstrates the end-to-end deployment of a containerized static web application on AWS using Docker, Amazon ECR, and Amazon ECS with Fargate.

The goal was to design a production-aligned container workflow that includes local validation, secure image storage, private networking, load-balanced traffic routing, and automated container orchestration without managing servers.

---

## Architecture
![Architecture Diagram](https://github.com/user-attachments/assets/c72f24e6-721b-48dd-9af8-169b6b905da3)


### Traffic Flow

1. Users access the application through an internet-facing Application Load Balancer  
2. The ALB forwards HTTP traffic to ECS tasks using IP-based target registration  
3. ECS runs Docker containers in private subnets using AWS Fargate  
4. Container images are pulled securely from Amazon ECR  
5. Private subnets access the internet through NAT Gateways  

This architecture isolates workloads while exposing only a controlled public entry point.

---

## Technology Stack

**Containerization**
- Docker
- Dockerfile (Amazon Linux + Apache)

**AWS Services**
- Amazon ECS (Fargate)
- Amazon ECR
- Application Load Balancer
- VPC (Three-Tier Architecture)
- IAM
- CloudWatch

**Networking**
- Public and Private Subnets across multiple Availability Zones
- Internet Gateway
- NAT Gateways
- Security Groups

---

## Container Build and Local Validation

The static website was packaged into a Docker image and validated locally before registry promotion.

- Image built from source using Docker
- Container executed locally with explicit port mapping
- Application verified via localhost
- Container lifecycle tested (start, stop, cleanup)

This ensured runtime correctness prior to deployment.

---

## Image Registry and Artifact Management

- Created a private Amazon ECR repository
- Tagged the Docker image following a registry-ready naming convention
- Authenticated Docker to ECR using AWS CLI
- Pushed the image as an immutable deployment artifact

ECR serves as the authoritative source for container images consumed by ECS.

---

## Network Infrastructure

A Three-Tier VPC was built from scratch to reflect production-grade networking.

**Design**
- Public subnets for load balancers
- Private application subnets for ECS tasks
- Private data subnets reserved for backend services
- Internet Gateway for inbound access
- NAT Gateways per Availability Zone for outbound access

This design supports high availability and strict traffic isolation.

---

## Security Model

**IAM**
- `ecsTaskExecutionRole` grants permission to pull images from ECR and publish logs
- No credentials embedded in images or containers

**Security Groups**
- ALB Security Group allows inbound HTTP traffic from the internet
- Container Security Group allows inbound traffic only from the ALB
- No SSH access
- No public IPs assigned to ECS tasks

Security controls were applied by default, not retrofitted.

---

## Load Balancing and Traffic Management

- Internet-facing Application Load Balancer deployed in public subnets
- IP-based Target Group configured for ECS Fargate compatibility
- Health checks enabled to route traffic only to healthy tasks
- Dynamic task registration handled automatically by ECS

The ALB acts as the single ingress point.

---

## Container Orchestration

**ECS Cluster**
- Fargate-based cluster with no EC2 management
- Service-linked role initialized via AWS CLI to resolve deployment failure

**Task Definition**
- Defines container image, CPU, memory, and port mappings
- Uses `awsvpc` networking mode
- Pulls images directly from ECR

**ECS Service**
- Maintains desired task count
- Automatically replaces failed tasks
- Integrated with Application Load Balancer
- Service auto scaling enabled

The ECS Service functions similarly to an Auto Scaling Group for containers.

---

## Observability and Operations

- ECS service events provide deployment visibility
- Container logs streamed to CloudWatch
- Health checks enforce traffic integrity
- Scaling occurs without service interruption

Operational visibility was designed into the system.

---

## Failure Encountered and Resolution

**Issue**  
ECS cluster creation failed with:  
`Unable to assume the service linked role`

**Root Cause**  
A service-linked role already existed, causing a CloudFormation race condition.

**Resolution**
- Initialized the role using AWS CLI
- Deleted the failed stack
- Re-created the cluster successfully

This validated infrastructure-level troubleshooting.

---

## Key Concepts Demonstrated

- Container lifecycle management
- Artifact promotion to private registry
- Private networking with controlled ingress
- Serverless container orchestration
- Load-balanced traffic routing
- IAM least-privilege access
- Auto scaling and self-healing services

---

## Professional Takeaway

This project demonstrates the ability to design, deploy, secure, and operate a containerized application using AWS managed services with production-aligned patterns.

---

## Future Enhancements

- Automate image builds and pushes using CI pipelines
- Provision infrastructure using Terraform
- Add HTTPS using ACM and Route 53
- Implement blue-green or rolling deployments

---
