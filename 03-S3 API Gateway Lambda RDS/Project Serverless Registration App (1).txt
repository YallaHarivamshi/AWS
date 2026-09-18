Project: Serverless Registration Application
===========================================

This project implements a serverless registration system using AWS services. The frontend is hosted as a static website on S3, allowing users to enter their registration details. When a user submits the form, the data is sent via API Gateway, which acts as a secure HTTP endpoint. AWS Lambda handles the backend logic, processing the request and storing user data in Amazon RDS (MySQL) database.

Key Services & Features:

•	S3: Hosts the static HTML/CSS/JS frontend.
•	API Gateway: Provides a HTTP API endpoint to securely connect the frontend to Lambda.
•	Lambda : Executes backend logic for processing registration data.
•	RDS (MySQL): Stores user registration information persistently.
•	CORS Configuration: Enables the frontend to communicate with API Gateway.


This architecture demonstrates a fully serverless flow, connecting a web application to a relational database without managing servers, making it scalable and cost-efficient.

Create RDS DB Instance
======================

Create a Publicly access RDS(MySQL), sample DB = regdb - no backup required

Copy the RDS Endpoint and access from the tool (MYSQL Workbench)

Use the regdb created at the time of RDS set-up

USE regdb;
CREATE TABLE users ( id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(100), email VARCHAR(100), password VARCHAR(100) );


Create IAM ROLE
==============

Lambda needs to talk to RDS, so better TE=Lambda, Permissions=Admin, RoleName=reg-role


Install python on Windows
-------------------------

https://www.python.org/ftp/python/3.14.5/python-3.14.5-amd64.exe

Search in windows Environment Variables --> user variables --> select Path --> edit --> new and add below paths

C:\Users\Reyaz\AppData\Local\Programs\Python\Python314\
C:\Users\Reyaz\AppData\Local\Programs\Python\Python314\Scripts

Open cmd --> python --version and pip --version

mkdir Lambda

pip install pymysql -t python
pip install mysql-connector-python -t .

Open Win explorer and go to the path and zip by selecting all files

pymysql
pymysql-1.1.3.dist-info
MySQL
mysql_connector_python-9.7.0.dist-info
_mysql_connector.cp314-win_amd64




Create Lambda Function
=====================
Go to the lambda service and create a function.
Create function
Choose :Author from scratch
Fill the details:
Function name: regUser
Runtime: Python 3.12 (or 3.11)
Architecture: x86_64
Permissions: Select role : reg-role
Create function

Add environment variable to store the RDS connection details.
------------------------------
Configuration → Environment Variables → Edit
DB_HOST = your-rds-endpoint
DB_USER = admin
DB_PASSWORD = yourpassword
DB_NAME = regdb


Go to Code 
---------
Select update --> update from a zip file and upload MySQL.zip 
and click on create and create a file and paste the below code --> Deploy



import json
import mysql.connector
import os

def lambda_handler(event, context):

    headers = {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Headers': '*',
        'Access-Control-Allow-Methods': 'OPTIONS,POST,GET'
    }

    try:

        if event['requestContext']['http']['method'] == 'OPTIONS':
            return {
                'statusCode': 200,
                'headers': headers,
                'body': json.dumps('CORS OK')
            }

        body = json.loads(event['body'])

        connection = mysql.connector.connect(
            host=os.environ['DB_HOST'],
            user=os.environ['DB_USER'],
            password=os.environ['DB_PASSWORD'],
            database=os.environ['DB_NAME']
        )

        cursor = connection.cursor()

        query = """
        INSERT INTO users(name,email,password)
        VALUES(%s,%s,%s)
        """

        values = (
            body['name'],
            body['email'],
            body['password']
        )

        cursor.execute(query, values)

        connection.commit()

        return {
            'statusCode': 200,
            'headers': headers,
            'body': json.dumps({
                'message': 'User registered successfully'
            })
        }

    except Exception as e:

        print(str(e))

        return {
            'statusCode': 500,
            'headers': headers,
            'body': json.dumps({
                'error': str(e)
            })
        }


Do a quick test --> Click on test --> and see the output if it success or not


Create API Gateway
-----------------
Go to API Gateway and create HTTP API

Name: regapi

Add Integration --> Lambda --> select Lambda function
Add route --> Method: POST, resource path : /register, integration target: regUser

remove default
Add stage --> regstage --> off auto-deploy

Left Side --> CORS --> configure
Access-Control-Allow-Origin : *  ADD
Access-Control-Allow-Headers: content-type
Access-Control-Allow-Methods: POST , OPTIONS
SAVE

On TOP -- Deploy --> Select Stage and Deploy
Left side --> stage --> Copy the URL https://f8ww35p4fj.execute-api.ap-south-1.amazonaws.com/regstage


Edit the index.html and replace the above URL

Create a S3 bucket
-----------------
Create a Public S3 public bucket , name: registration-frontend-13 
Upload index.html --> make public
Enable static Website hosting

Open the URL and start accessing


Now verify in the Database use SQL Workbench

USE regdb;

select * from users









