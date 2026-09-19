# 3-Tier Architecture Application Deployment on AWS

## Project Overview

This project demonstrates the deployment of a **3-Tier Web Application on AWS** using a custom VPC, Web Tier, Application Tier, and Database Tier.

The application uses **Nginx Web Servers**, **Node.js Application Servers**, and **Amazon RDS MySQL**. Load Balancers and Auto Scaling Groups are used to provide availability and scalability.

## Architecture

```text
                         USER
                           |
                           v
                    Route 53 / HTTPS
                           |
                           v
                    CloudFront
                           |
                           v
              External Application Load Balancer
                           |
                           v
                 +-------------------+
                 |     WEB TIER      |
                 |   Nginx Servers   |
                 |    Web ASG         |
                 +-------------------+
                           |
                           v
              Internal Application Load Balancer
                           |
                           v
                 +-------------------+
                 |      APP TIER     |
                 |  Node.js + PM2    |
                 |    App ASG         |
                 +-------------------+
                           |
                           v
                 +-------------------+
                 |    DATABASE TIER  |
                 |    RDS MySQL      |
                 +-------------------+
```

## AWS Services Used

| AWS Service               | Purpose                             |
| ------------------------- | ----------------------------------- |
| Amazon VPC                | Creates isolated network            |
| EC2                       | Hosts Web and Application servers   |
| Application Load Balancer | Distributes traffic                 |
| Auto Scaling              | Automatically manages EC2 instances |
| Amazon RDS MySQL          | Stores application data             |
| Amazon S3                 | Stores application code             |
| IAM                       | Provides permissions to EC2         |
| AWS Systems Manager       | Connects to private EC2 instances   |
| ACM                       | Provides HTTPS certificate          |
| Route 53                  | Provides custom domain              |
| CloudFront                | CDN and application delivery        |
| Nginx                     | Web server and reverse proxy        |
| Node.js                   | Application backend                 |
| PM2                       | Runs Node.js application            |
| CloudWatch                | Monitoring and logs                 |

## VPC Architecture

**VPC CIDR:**

```text
192.168.0.0/16
```

### Subnets

```text
Public Subnets
├── Public1
└── Public2

Private Subnets
├── APP1
├── APP2
├── DB1
└── DB2
```

The architecture is deployed across **two Availability Zones**.

A **NAT Gateway** is used to provide outbound internet connectivity for resources in private subnets.

## Security Groups

Five Security Groups are created:

```text
WebALB-SG
     ↓
Web-SG
     ↓
AppALB-SG
     ↓
App-SG
     ↓
Database-SG
```

### Traffic Flow

```text
Internet
   ↓
WebALB-SG
   ↓
Web-SG
   ↓
AppALB-SG
   ↓
App-SG
   ↓
Database-SG
```

This controls which tier is allowed to communicate with the next tier.

## Three Tiers

### 1. Web Tier

The Web Tier contains:

* Amazon EC2
* Nginx
* Frontend application
* External Application Load Balancer
* Web Auto Scaling Group

The External ALB receives requests from users and distributes them to Web Servers.

```text
User
  ↓
External ALB
  ↓
Web Server
  ↓
Nginx
```

### 2. Application Tier

The Application Tier contains:

* Amazon EC2
* Node.js
* PM2
* Internal Application Load Balancer
* Application Auto Scaling Group

The Application Tier processes application requests and communicates with the database.

```text
Web Tier
   ↓
Internal ALB
   ↓
Node.js Application
   ↓
RDS MySQL
```

The application runs on:

```text
Port: 4000
```

Health check:

```text
/health
```

### 3. Database Tier

The Database Tier uses **Amazon RDS for MySQL**.

The database is deployed in private subnets and is not publicly accessible.

```text
App Server
    |
    v
RDS MySQL
```

## S3 Application Code Storage

Amazon S3 is used as a private storage location for the application code.

Example structure:

```text
S3 Bucket
│
└── application-code
    │
    ├── app-tier
    │   ├── index.js
    │   ├── DbConfig.js
    │   └── package.json
    │
    └── web-tier
        ├── frontend files
        └── package.json
```

The EC2 instances download the required application files from S3.

## Application Server Setup

The Application Server runs:

```text
Node.js
PM2
MySQL Client
```

The application listens on:

```text
Port 4000
```

PM2 is used to keep the Node.js application running as a service.

## Database

Database used:

```text
MySQL
```

Example database:

```text
webappdb
```

Example table:

```text
transactions
```

The table stores transaction information such as:

```text
id
amount
description
```

## Internal Load Balancer

An **Internal Application Load Balancer** is created for the Application Tier.

```text
Web Server
     |
     v
Internal ALB
     |
     +--------> App Server 1
     |
     +--------> App Server 2
```

The internal ALB is accessible only inside the VPC.

## External Load Balancer

An **Internet-facing Application Load Balancer** is created for the Web Tier.

```text
Internet
    |
    v
External ALB
    |
    +--------> Web Server 1
    |
    +--------> Web Server 2
```

## Auto Scaling

Auto Scaling Groups are configured for both Web and Application Tiers.

### Application ASG

```text
Minimum: 2
Desired: 4
Maximum: 6
CPU Target: 70%
```

### Web ASG

```text
Minimum: 2
Desired: 4
Maximum: 6
CPU Target: 70%
```

Auto Scaling automatically launches or terminates EC2 instances based on application demand.

## HTTPS

AWS Certificate Manager is used to create an SSL/TLS certificate.

HTTPS traffic is terminated at the External Load Balancer.

```text
User
  |
  | HTTPS
  v
External ALB
  |
  | HTTP
  v
Web Servers
```

## Route 53

Amazon Route 53 is used to connect the application to a custom domain.

Example:

```text
https://boom.reyazawstrainer.com
```

Route 53 points the domain to the application infrastructure.

## CloudFront

Amazon CloudFront is implemented to provide:

* Content delivery
* Lower latency
* Edge caching
* Improved application performance

Final traffic flow:

```text
User
  |
  v
Route 53
  |
  v
CloudFront
  |
  v
External ALB
  |
  v
Web Tier
  |
  v
Internal ALB
  |
  v
Application Tier
  |
  v
RDS MySQL
```

## Complete Request Flow

```text
1. User opens the application URL
                |
                v
2. Route 53 resolves the domain
                |
                v
3. CloudFront receives the request
                |
                v
4. External ALB receives traffic
                |
                v
5. Web ASG provides available Web Server
                |
                v
6. Nginx forwards API requests
                |
                v
7. Internal ALB receives the request
                |
                v
8. App ASG provides an Application Server
                |
                v
9. Node.js processes the request
                |
                v
10. RDS MySQL stores or retrieves data
                |
                v
11. Response returns to the user
```

## High Availability

The architecture uses two Availability Zones.

```text
              AWS Region
                  |
        +---------+---------+
        |                   |
       AZ-1                AZ-2
        |                   |
   Web Server 1        Web Server 2
   App Server 1        App Server 2
   DB Subnet 1        DB Subnet 2
```

Multiple instances and Load Balancers help distribute traffic across available resources.

## Auto Scaling Flow

```text
              Application Load
                    |
                    v
              Auto Scaling
                    |
          +---------+---------+
          |                   |
      EC2 Instance        EC2 Instance
          |                   |
          +---------+---------+
                    |
              High CPU Usage
                    |
                    v
             Launch Instance
```

## Project Structure

```text
3-tier-project/
│
├── application-code/
│   │
│   ├── app-tier/
│   │   ├── index.js
│   │   ├── DbConfig.js
│   │   └── package.json
│   │
│   └── web-tier/
│       ├── frontend files
│       └── package.json
│
├── install.sh
│
└── README.md
```

## Key Features

* Custom AWS VPC
* Three-tier architecture
* Public and private subnets
* Web and Application Auto Scaling
* External and Internal Load Balancers
* Nginx Web Server
* Node.js Application
* PM2 Process Manager
* Private RDS MySQL
* Private S3 application-code storage
* IAM roles
* AWS Systems Manager
* HTTPS using ACM
* Custom domain using Route 53
* CloudFront CDN
* Multi-AZ deployment
* Health checks and automatic scaling

## Technologies

```text
AWS
EC2
VPC
ALB
Auto Scaling
S3
RDS MySQL
IAM
SSM
ACM
Route 53
CloudFront
Nginx
Node.js
PM2
```

## Project Outcome

This project demonstrates how to deploy a **secure, scalable, and highly available 3-Tier application on AWS**, separating the Web, Application, and Database layers while using Load Balancers, Auto Scaling, HTTPS, Route 53, and CloudFront for production-style infrastructure.
