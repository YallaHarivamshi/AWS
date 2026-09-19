# Serverless Employee Management Application

## Project Overview

This project demonstrates a **serverless employee management application using AWS**.

The application uses **Amazon S3** for the frontend, **API Gateway** to expose REST APIs, **AWS Lambda** for backend processing, and **Amazon DynamoDB** to store employee data.

## Architecture

```text
                    USER
                      |
                      v
             +----------------+
             |   S3 Website   |
             | HTML + JS      |
             +-------+--------+
                     |
                     | HTTP GET / POST
                     v
             +----------------+
             |  API Gateway   |
             | REST API       |
             +-------+--------+
                     |
              +------+------+
              |             |
             GET           POST
              |             |
              v             v
       +-------------+ +------------------+
       | getEmployee | | insertEmployeeData|
       |   Lambda    | |     Lambda       |
       +------+------+ +--------+---------+
              |                 |
              +--------+--------+
                       |
                       v
              +------------------+
              |    DynamoDB      |
              |   employeeData   |
              +------------------+
                       |
                       v
                Employee Data
```

## AWS Services Used

| AWS Service | Purpose                        |
| ----------- | ------------------------------ |
| Amazon S3   | Hosts the frontend website     |
| API Gateway | Provides REST API endpoints    |
| AWS Lambda  | Handles backend operations     |
| DynamoDB    | Stores employee information    |
| IAM         | Provides permissions to Lambda |

## Application Flow

```text
User
 ↓
S3 Static Website
 ↓
API Gateway
 ↓
Lambda
 ↓
DynamoDB
```

### GET Request

Used to retrieve employee data.

```text
User
 ↓
S3 Website
 ↓
API Gateway GET
 ↓
getEmployee Lambda
 ↓
DynamoDB
 ↓
Employee Data
```

### POST Request

Used to insert employee data.

```text
User
 ↓
S3 Website
 ↓
API Gateway POST
 ↓
insertEmployeeData Lambda
 ↓
DynamoDB
 ↓
Employee Data Stored
```

## DynamoDB Configuration

Table name:

```text
employeeData
```

Primary Key:

```text
employeeid
```

Data Type:

```text
String
```

## Lambda Functions

Two Lambda functions are used:

### 1. getEmployee

Responsible for retrieving employee information from DynamoDB.

```text
API Gateway
     ↓
GET
     ↓
getEmployee
     ↓
DynamoDB
```

### 2. insertEmployeeData

Responsible for inserting employee information into DynamoDB.

```text
API Gateway
     ↓
POST
     ↓
insertEmployeeData
     ↓
DynamoDB
```

## IAM Role

An IAM role is created with:

```text
Trusted Entity:
Lambda
```

The required DynamoDB permissions are attached to the role.

This role is attached to the Lambda functions so they can access the DynamoDB table.

## API Gateway

API type:

```text
REST API
```

API name:

```text
employee
```

Endpoint type:

```text
Edge-Optimized
```

Methods:

```text
GET
POST
```

Lambda integrations:

```text
GET  → getEmployee
POST → insertEmployeeData
```

CORS is enabled for both:

```text
GET
POST
```

Deployment stage:

```text
employeeapi
```

## Frontend

The frontend contains:

```text
index.html
script.js
```

These files are uploaded to an **Amazon S3 bucket**.

The S3 bucket is configured for **static website hosting**.

The JavaScript frontend communicates with API Gateway using the deployed API endpoint.

```text
S3 Website
    |
    | API Request
    v
API Gateway
```

## Complete Architecture Flow

```text
                USER
                  |
                  v
        +-------------------+
        |   Amazon S3       |
        | Static Website    |
        | HTML + JavaScript |
        +---------+---------+
                  |
                  | GET / POST
                  v
        +-------------------+
        |   API Gateway     |
        |   REST API        |
        +---------+---------+
                  |
          +-------+-------+
          |               |
         GET             POST
          |               |
          v               v
 +----------------+ +---------------------+
 | getEmployee    | | insertEmployeeData   |
 | Lambda         | | Lambda              |
 +-------+--------+ +---------+-----------+
          |                  |
          +--------+---------+
                   |
                   v
        +-------------------+
        |    DynamoDB       |
        |   employeeData    |
        |                   |
        | PK: employeeid    |
        +-------------------+
```

## Project Steps

1. Create DynamoDB table `employeeData`.
2. Create IAM role for Lambda with DynamoDB permissions.
3. Create `getEmployee` Lambda function.
4. Create `insertEmployeeData` Lambda function.
5. Create API Gateway REST API named `employee`.
6. Create GET method for `getEmployee`.
7. Create POST method for `insertEmployeeData`.
8. Enable CORS for GET and POST.
9. Deploy the API using the `employeeapi` stage.
10. Update the API URL in `script.js`.
11. Create a public S3 bucket.
12. Upload `index.html` and `script.js`.
13. Enable S3 static website hosting.
14. Configure the bucket policy for website access.
15. Open the S3 website and submit employee data.

## Technologies Used

```text
AWS
Amazon S3
API Gateway
AWS Lambda
Amazon DynamoDB
AWS IAM
HTML
JavaScript
REST API
```

## Key Learning

Through this project, I learned:

* How to create a DynamoDB table
* How Lambda functions interact with DynamoDB
* How to create REST APIs using API Gateway
* How to connect GET and POST methods to Lambda
* How to enable CORS
* How to deploy an API using API Gateway stages
* How to host a static website using S3
* How a serverless application works without managing servers

## Final Result

A simple **serverless employee management application** was created using:

```text
S3
 ↓
API Gateway
 ↓
Lambda
 ↓
DynamoDB
```

The frontend is hosted on S3, API Gateway handles requests, Lambda processes the requests, and DynamoDB stores the employee data.
