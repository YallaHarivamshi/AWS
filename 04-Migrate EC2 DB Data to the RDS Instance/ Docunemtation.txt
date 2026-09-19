Migrate EC2 DB Data to the RDS Instance
=======================================


Flow
----

Create a EC2 instance and install MySQL database and Create sample tables
Create a MySQL RDS DB instance
Create IAM ROLES
Migrate EC2 data to RDS 


Lets Start
==========


Step-1
-------

Launch Linux EC2 instance(RedHat, t3.micro), increase the volume to 20 Gb as we will create and install MySQL DB. 

Login to the EC2 instance.

Install the Mysql on the EC2:-
--------------------------------

sudo -s

sudo yum update -y

sudo dnf install mysql8.4-server -y

sudo systemctl enable --now mysqld.service

sudo systemctl status mysqld

Run secure installation:

sudo mysql_secure_installation

VALIDATE PASSWORD COMPONENT: yes
Password validation policy: LOW
Root password: welcome123
Do you want to continue the password? YES
Remove anonymous users? NO
Disallow root login remotely? NO
Remove test database and access to it? NO
Reload privilege tables now? Yes

Connect to local MySQL:
--------------
mysql -u root -p
Password: welcome123


CREATE DATABASE IF NOT EXISTS company_test_db;

USE company_test_db;

CREATE TABLE departments (
department_id INT AUTO_INCREMENT PRIMARY KEY,
department_name VARCHAR(50) NOT NULL UNIQUE,
location VARCHAR(50) DEFAULT 'Main Campus'	
);


CREATE TABLE employees (
employee_id INT AUTO_INCREMENT PRIMARY KEY,
first_name VARCHAR(50) NOT NULL,
last_name VARCHAR(50) NOT NULL,
email VARCHAR(100) UNIQUE,
hire_date DATE NOT NULL,
salary DECIMAL(10, 2) NOT NULL,
is_active BOOLEAN DEFAULT TRUE,
department_id INT,
FOREIGN KEY (department_id)
REFERENCES departments(department_id)
ON DELETE SET NULL
);


INSERT INTO departments (department_name, location) VALUES
('Engineering', 'Building A'),
('Data Science', 'Building A'),
('Human Resources', 'Building B'),
('Marketing', 'Remote'),
('Finance', 'Building B');



INSERT INTO employees
(first_name, last_name, email, hire_date, salary, is_active, department_id)
VALUES
('Alice', 'Smith', 'alice.smith@example.com', '2022-03-15', 95000.00, TRUE, 1),
('Bob', 'Johnson', 'bob.johnson@example.com', '2021-06-20', 105000.00, TRUE, 1),
('Charlie', 'Brown', 'charlie.brown@example.com', '2023-01-10', 88000.00, TRUE, 2),
('Diana', 'Prince', 'diana.prince@example.com', '2020-11-05', 120000.00, TRUE, 2),
('Evan', 'Wright', 'evan.wright@example.com', '2019-05-12', 65000.00, TRUE, 3),
('Fiona', 'Gallagher', 'fiona.g@example.com', '2024-02-01', 55000.00, FALSE, 4),
('George', 'Miller', 'george.m@example.com', '2022-08-24', 72000.00, TRUE, 4),
('Hannah', 'Abbott', 'hannah.a@example.com', '2021-09-18', 85000.00, TRUE, 5),
('Ian', 'Malcolm', 'ian.m@example.com', '2023-07-19', 98000.00, TRUE, NULL);


SELECT e.employee_id, e.first_name, e.last_name,
d.department_name, e.salary
FROM employees e
INNER JOIN departments d
ON e.department_id = d.department_id;


SELECT d.department_name,
COUNT(e.employee_id) AS total_staff,
ROUND(AVG(e.salary), 2) AS avg_salary
FROM departments d
LEFT JOIN employees e
ON d.department_id = e.department_id
GROUP BY d.department_name;



Important
----------
SELECT user,host FROM mysql.user;

this show
root             | localhost

it should be % meaning, not only localhost, others also need to connect especially DMS


RENAME USER 'root'@'localhost' TO 'root'@'%';

GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;

FLUSH PRIVILEGES;


SELECT user,host FROM mysql.user;

Step-2
-------

Before creating RDS , go to IAM and create a role [we might have already same role created, if not create]

Create the 2 roles manually from the IAM Console.

Keep Same Role Names

TE = DMS
Permissions= AmazonDMSVPCManagementRole

Set the Role name to exactly: dms-vpc-role


TE= DMS
Permissions = AmazonDMSCloudWatchLogsRole, CloudWatchFullAccess

Role Name = dms-cloudwatch-logs-role



Step 3:
------

Create a Inline Policy 

{
"Version": "2012-10-17",
"Statement": [
{
"Effect": "Allow",
"Action": "iam:PassRole",
"Resource": "arn:aws:iam::*:role/dms-vpc-role"
}
]
}

Name: DMSPassRolePolicy

Attach this Policy to dms-vpc-role


Step-4
------

Setup RDS instance
------------------
Engine: MySQL
username: admin
password: root123456
Public Access: NO
Setup EC2 connection:
- Select “Connect to an EC2 compute resource”
- Choose the EC2 instance running MySQL
- Security Groups: dont select default, it will create new
Defualt database: NO - keep empty
Backups : NO

If required: connect to RDS from ec2 instance, later we do to check the migrated tables
mysql -h mysqdb.cdbmlufgqkjd.ap-south-1.rds.amazonaws.com -P 3306 -u admin -p

Step-5
-------

Migrate Database
------------

Select RDS Instance --> Migrate data to this database

Location: EC2 instance and select EC2
secret: Create and use a secret --> username: root, password: welcome123  [this is ec2 instance MySQL root password]

IAM role for secret --> Create and use a new IAM role --> IAM role name --> may18-source-role	
Security Socket Layer (SSL) mode --> NONE

Target
======
Create and use a new secret --> username: admin, password: root123456  [this is rds db instance admin password]
Create and use a new IAM role --> IAM role name --> may18-target-role


Configure data migration
==========================
full load
Create and use a new IAM role --> may18-dms-role

Create 

If you get any error like " Access denied when trying to access your VPC settings. Please check your service access role trust relationships"

Ignore the error and Re-Create it 


Security groups
--------------
Go to ec2-rds-1 and allow MySQL traffic from ip-sg-ec2-rds-mip-1779078965930 [DMS created security group]

EC2 instance has ec2-rds-1, this should receive the traffic from DMS created SG on MySQL hence add a new rule in ec2-rds-1 


connect to RDS and check the databases and tables
-------------

mysql -h mysqdb.cdbmlufgqkjd.ap-south-1.rds.amazonaws.com -P 3306 -u admin -p

show databases;
use company_test_db;

show tables;

select * from departments;
select * from employees;


Clean up
---------

Delete RDS DB instance
Terminate EC2 instance
Delete roles: may18-target-role, may18-source-role, may18-dms-role
Delete Secrets
