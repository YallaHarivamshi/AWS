# EC2 Blue-Green Deployment with Application Load Balancer

## Project Overview

This project demonstrates a Blue-Green Deployment architecture using AWS EC2, Application Load Balancer (ALB), Route 53, and AWS Certificate Manager (ACM).

The application is deployed in two environments:

- Blue Environment - Current/Actual Production Environment
- Green Environment - New Environment

The Application Load Balancer distributes traffic to the active environment. During deployment, traffic can be switched from Blue to Green without directly changing the Route 53 DNS configuration.

## AWS Services Used

- Amazon EC2
- Application Load Balancer (ALB)
- Target Groups
- Amazon Route 53
- AWS Certificate Manager (ACM)
- HTTP/HTTPS
- Linux
- Apache HTTP Server

## Architecture

The architecture contains:

### Blue Environment

- Blue-Server1 - EC2
- Blue-Server2 - EC2
- Blue Target Group

### Green Environment

- Green-Server1 - EC2
- Green-Server2 - EC2
- Green Target Group

### Traffic Flow

User
→ Route 53
→ Application Load Balancer
→ Active Target Group
→ EC2 Servers

## Blue-Green Deployment

Blue represents the currently active production environment.

Green represents the new application environment.

After testing the Green environment, the ALB target group can be switched from Blue to Green.

This allows application deployment with minimal downtime and provides an easier rollback mechanism.

## EC2 Configuration

### Blue Environment

The Blue servers host the Villa Agency application.

### Green Environment

The Green servers host the Restaurant/Klassy Cafe application.

## HTTP and HTTPS

The Application Load Balancer uses:

- HTTP - Port 80
- HTTPS - Port 443

HTTPS is configured using AWS Certificate Manager (ACM).

## SSL Certificate

A public certificate is requested from ACM for the application domain.

DNS validation is performed using Route 53.

The certificate is then associated with the HTTPS listener of the Application Load Balancer.

## Result

The project demonstrates:

- EC2 web server deployment
- Application Load Balancer configuration
- Target group configuration
- Route 53 DNS configuration
- HTTPS configuration using ACM
- Blue-Green deployment strategy
- Traffic switching between environments
