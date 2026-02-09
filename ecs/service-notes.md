# ECS Service Design Notes – Jupiter Application

## Service Overview

The Jupiter application is deployed as an Amazon ECS Service using AWS Fargate.  
The service is responsible for maintaining availability, integrating with the Application Load Balancer, and automatically replacing failed containers.

The ECS Service fulfills a role similar to an Auto Scaling Group for containerized workloads.

---

## Cluster and Launch Type

- Cluster: `jupiter-cluster`
- Launch Type: AWS Fargate

Fargate was selected to eliminate EC2 instance management and reduce operational overhead while maintaining fine-grained control over networking and scaling.

---

## Networking Configuration

- VPC: Dev-VPC (Three-Tier Architecture)
- Subnets: Private Application Subnets across two Availability Zones
- Public IP Assignment: Disabled

Running tasks in private subnets ensures containers are not directly exposed to the internet.  
All inbound traffic is routed through the Application Load Balancer.

---

## Load Balancer Integration

- Load Balancer Type: Application Load Balancer
- Listener: HTTP :80
- Target Group Type: IP

The IP-based target group is required for ECS Fargate, as tasks receive dynamic ENIs rather than fixed EC2 instance IDs.

The ALB acts as the single ingress point for the application.

---

## Security Groups

- ALB Security Group:
  - Allows inbound HTTP traffic from the internet

- Container Security Group:
  - Allows inbound traffic only from the ALB Security Group
  - No SSH access
  - No public exposure

This enforces least-privilege network access and reduces the attack surface.

---

## Task Placement and Availability

- Scheduling Strategy: Replica
- Desired Tasks: 1
- Minimum Tasks: 1
- Maximum Tasks: 2

The service ensures at least one task is always running and automatically replaces failed tasks.

---

## Auto Scaling Behavior

- Scaling Type: ECS Service Auto Scaling
- Metric: CPU utilization
- Target Value: 70%

Auto scaling allows the service to respond to increased load without manual intervention while maintaining cost control.

---

## Health Checks and Resilience

- Health checks are performed via the Application Load Balancer
- Unhealthy tasks are automatically deregistered and replaced
- Rolling deployments ensure zero downtime during updates

This design supports self-healing and high availability.

---

## Operational Notes

- New deployments are triggered by updating the task definition revision
- Containers are immutable; changes require a new image build
- Logs are streamed to Amazon CloudWatch for visibility and troubleshooting

---

## Design Rationale Summary

This ECS Service was designed to:
- Avoid direct public exposure of containers
- Enforce separation of concerns between build and runtime
- Support horizontal scaling
- Enable automated recovery from failures
- Align with production-grade DevOps patterns
