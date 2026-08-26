# AWS Blue-Green Deployment using EC2, ALB, Route 53 and ACM

## Project Overview

Implemented a Blue-Green Deployment architecture on AWS using EC2 instances, Application Load Balancer (ALB), Target Groups, Route 53 and AWS Certificate Manager (ACM).

The Blue environment contains the current application version, while the Green environment contains the new application version. Traffic can be switched from Blue to Green after testing and health validation.

## Architecture

![AWS Blue-Green Deployment Architecture](blue-green-architecture.png)

## AWS Services Used

- Amazon EC2
- Application Load Balancer (ALB)
- Target Groups
- Amazon Route 53
- AWS Certificate Manager (ACM)
- HTTP/HTTPS Listeners
- Health Checks

## Environment Setup

### Blue Environment - Current Version

- Blue Server 1 - EC2
- Blue Server 2 - EC2
- Blue Target Group
- Villa Agency application

### Green Environment - New Version

- Green Server 1 - EC2
- Green Server 2 - EC2
- Green Target Group
- Klassy Cafe application

## Deployment Flow

1. User accesses the application using the Route 53 domain.
2. Route 53 resolves the domain to the Application Load Balancer.
3. ALB receives the HTTP/HTTPS request.
4. ALB forwards traffic to the active Target Group.
5. Blue environment serves the current application.
6. Green environment is deployed and tested separately.
7. Health checks verify that Green instances are healthy.
8. Traffic is switched from Blue Target Group to Green Target Group.
9. Green becomes the production environment.
10. Blue can be retained as a backup or removed after validation.

## Blue Server Setup

The Blue servers run the Villa Agency application using Apache HTTP Server.

The deployment script is available in:

`scripts/blue-server.sh`

## Green Server Setup

The Green servers run the Klassy Cafe application using Apache HTTP Server.

The deployment script is available in:

`scripts/green-server.sh`

## HTTPS Configuration

AWS Certificate Manager (ACM) was used to provide an SSL/TLS certificate.

The HTTPS listener was configured on the Application Load Balancer using port 443.

## Health Checks

ALB Target Groups perform health checks on the EC2 instances.

Only healthy instances receive application traffic.

## Blue-Green Deployment Strategy

**Blue = Current production environment**

**Green = New environment**

After deploying and testing the Green environment, traffic is switched from Blue to Green.

This allows application updates with minimal downtime and provides an easy rollback option.

## Project Outcome

Successfully designed and implemented a Blue-Green Deployment architecture using AWS EC2, ALB, Target Groups, Route 53 and ACM.
