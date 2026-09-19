# AWS Serverless Employee Management System

## Project Overview

This project demonstrates a **serverless, event-driven employee management system** using AWS.

The application allows users to add employees through a web interface and view employee records. Employee registration is processed asynchronously using **Amazon SQS** before being stored in **DynamoDB**.

## Architecture

```text
                    USER
                      |
                      v
              Amazon S3 Website
                 (HTML + JS)
                      |
                      v
                API Gateway
                 /        \
                /          \
        POST /register    GET /employees
              |                |
              v                v
     Lambda Producer     Lambda Get Employees
              |                |
              v                v
          SQS Queue        DynamoDB
              |
              v
      Lambda Consumer
              |
              v
          DynamoDB
```

## AWS Services Used

| AWS Service       | Purpose                                  |
| ----------------- | ---------------------------------------- |
| Amazon S3         | Hosts the frontend website               |
| API Gateway       | Provides REST API endpoints              |
| AWS Lambda        | Runs backend application logic           |
| Amazon SQS        | Provides asynchronous message processing |
| Amazon DynamoDB   | Stores employee information              |
| Amazon CloudWatch | Monitoring and Lambda logs               |

## Application Flow

1. User opens the employee management website hosted on **Amazon S3**.
2. User enters employee details and clicks **Add Employee**.
3. Frontend sends a `POST /register` request to **API Gateway**.
4. API Gateway invokes the **employee-producer Lambda**.
5. Producer Lambda sends the employee data to **Amazon SQS**.
6. SQS triggers the **employee-consumer Lambda**.
7. Consumer Lambda processes the message and stores the employee data in **DynamoDB**.
8. Frontend sends a `GET /employees` request to API Gateway.
9. API Gateway invokes the **get-employees Lambda**.
10. Lambda reads employee records from DynamoDB.
11. Employee records are displayed in the frontend.

## DynamoDB

**Table Name:** `employees`

**Partition Key:** `id`

**Data stored:**

```text
id
name
email
department
```

Each employee receives a unique ID using Python `uuid`.

## SQS

**Queue Name:**

```text
employee-queue
```

**Queue Type:**

```text
Standard
```

SQS is used to decouple employee registration from database processing.

This means the Producer Lambda does not directly write employee data to DynamoDB. Instead, it sends the data to SQS, and the Consumer Lambda processes it asynchronously.

## Lambda Functions

### 1. employee-producer

Receives employee information from API Gateway and sends it to SQS.

```text
Frontend
   ↓
API Gateway
   ↓
employee-producer
   ↓
SQS
```

### 2. employee-consumer

Automatically receives messages from SQS and stores employee data in DynamoDB.

```text
SQS
   ↓
employee-consumer
   ↓
DynamoDB
```

### 3. get-employees

Reads employee records from DynamoDB and returns them to the frontend.

```text
Frontend
   ↓
API Gateway
   ↓
get-employees
   ↓
DynamoDB
```

## API Endpoints

### Register Employee

```text
POST /register
```

Used to add a new employee.

### Get Employees

```text
GET /employees
```

Used to retrieve employee records from DynamoDB.

## IAM

Lambda functions use IAM roles to access AWS services.

Required permissions include:

* AWS Lambda execution
* Amazon SQS access
* Amazon DynamoDB access

For a production environment, permissions should be restricted to only the required resources instead of using full-access policies.

## S3 Frontend

The frontend is hosted using **Amazon S3 Static Website Hosting**.

The frontend contains:

```text
index.html
```

Before uploading the file, update the API Gateway URL inside `index.html`.

Example:

```text
https://<api-id>.execute-api.<region>.amazonaws.com/prod
```

## Event-Driven Processing

The main advantage of this architecture is **asynchronous processing**.

```text
User Request
     |
     v
API Gateway
     |
     v
Producer Lambda
     |
     v
SQS Queue
     |
     v
Consumer Lambda
     |
     v
DynamoDB
```

The SQS queue temporarily stores messages until the Consumer Lambda processes them.

If the Consumer Lambda is unavailable, messages can remain in the queue and can be processed when the Lambda trigger is enabled again.

## Testing

After deploying the application:

1. Open the S3 website.
2. Enter employee details.
3. Click **Add Employee**.
4. Check the `employee-queue` in SQS.
5. Check the `employees` table in DynamoDB.
6. Use the employee list option to retrieve employee records.

### Testing SQS Manually

To observe messages in SQS:

1. Disable the SQS trigger from `employee-consumer`.
2. Add new employees from the website.
3. Messages will remain in the SQS queue.
4. Enable the SQS trigger again.
5. Lambda will process the queued messages.
6. Employee data will be stored in DynamoDB.

## Project Benefits

* Serverless architecture
* No server management
* Event-driven processing
* Asynchronous communication using SQS
* Scalable backend using Lambda
* NoSQL database using DynamoDB
* REST API integration
* Simple static frontend using S3
* CloudWatch logging and monitoring

## Technologies

```text
AWS
Python
HTML
JavaScript
Amazon S3
API Gateway
AWS Lambda
Amazon SQS
Amazon DynamoDB
Amazon CloudWatch
```

## Project Architecture Summary

```text
S3
 |
 | POST /register
 v
API Gateway
 |
 v
Lambda Producer
 |
 v
SQS Queue
 |
 v
Lambda Consumer
 |
 v
DynamoDB
 |
 ^
 |
Lambda Get Employees
 |
 ^
 |
API Gateway
 |
 ^
 |
S3 Frontend
```

## Conclusion

This project demonstrates how multiple AWS serverless services can be integrated to build a **scalable event-driven employee management application** without managing traditional servers.
