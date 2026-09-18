# Serverless Registration Application

## About the Project

This project is a **serverless user registration application built using AWS services**.

The frontend is hosted on **Amazon S3**. When a user submits the registration form, the request is sent to **API Gateway**. API Gateway triggers an **AWS Lambda** function, which processes the user details and stores them in an **Amazon RDS MySQL database**.

This project demonstrates how different AWS services can be connected to build a simple serverless web application without managing a traditional server.

## Architecture

```text
User
  |
  v
Amazon S3
(Frontend)
  |
  | HTTP POST
  v
API Gateway
  |
  v
AWS Lambda
  |
  | MySQL
  v
Amazon RDS
(MySQL)
```

## AWS Services Used

* **Amazon S3** – Hosts the frontend website.
* **API Gateway** – Provides the HTTP API endpoint.
* **AWS Lambda** – Processes registration requests.
* **Amazon RDS MySQL** – Stores user registration data.
* **IAM** – Provides permissions for Lambda.
* **CORS** – Allows communication between the frontend and API.

## Project Flow

**S3 → API Gateway → Lambda → RDS MySQL**

User enters registration details → API Gateway receives the request → Lambda processes the data → RDS MySQL stores the registration details.
