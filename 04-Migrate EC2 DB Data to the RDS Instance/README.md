# Migrate EC2 MySQL Database to Amazon RDS

## Project Overview

This project demonstrates how to **migrate a MySQL database running on an EC2 instance to Amazon RDS MySQL using AWS Database Migration Service (DMS).**

The project creates a sample MySQL database on EC2, prepares an RDS MySQL database, and migrates the database tables and data from EC2 to RDS.

## Architecture

```text
                SOURCE
        ┌─────────────────────┐
        │     EC2 Instance    │
        │                     │
        │   MySQL Database    │
        │  company_test_db    │
        │                     │
        │  ┌───────────────┐  │
        │  │ departments   │  │
        │  │ employees     │  │
        │  └───────────────┘  │
        └──────────┬──────────┘
                   │
                   │ MySQL Data
                   ▼
        ┌─────────────────────┐
        │      AWS DMS        │
        │                     │
        │    Full Load        │
        │                     │
        │  EC2 MySQL → RDS    │
        └──────────┬──────────┘
                   │
                   │ Migrated Data
                   ▼
        ┌─────────────────────┐
        │    Amazon RDS       │
        │     MySQL            │
        │                     │
        │  company_test_db    │
        │                     │
        │  ┌───────────────┐  │
        │  │ departments   │  │
        │  │ employees     │  │
        │  └───────────────┘  │
        └─────────────────────┘

        Supporting Services
        ───────────────────
        IAM Roles
        Secrets Manager
        Security Groups
```

## AWS Services Used

* Amazon EC2
* Amazon RDS
* AWS Database Migration Service (DMS)
* AWS IAM
* AWS Secrets Manager
* Amazon CloudWatch
* Security Groups

## Project Flow

```text
1. Launch EC2 Instance
          ↓
2. Install MySQL
          ↓
3. Create Database and Tables
          ↓
4. Insert Sample Data
          ↓
5. Create IAM Roles
          ↓
6. Create RDS MySQL Instance
          ↓
7. Configure Source and Target Credentials
          ↓
8. Start AWS DMS Migration
          ↓
9. Full Load EC2 MySQL → RDS MySQL
          ↓
10. Verify Tables and Data in RDS
```

## Database Used

### Database

`company_test_db`

### Tables

* `departments`
* `employees`

The `employees` table contains a foreign key relationship with the `departments` table.

## Source

The source database runs on:

```text
EC2 Instance
    ↓
MySQL Server
    ↓
company_test_db
```

## Target

The target database runs on:

```text
Amazon RDS
    ↓
MySQL
    ↓
company_test_db
```

## Migration

AWS DMS is used to migrate the database:

```text
EC2 MySQL
    │
    │ Full Load
    ▼
AWS DMS
    │
    ▼
RDS MySQL
```

The migration transfers the database structure and sample data from the EC2 MySQL database to the RDS MySQL database.

## IAM Roles

The project uses IAM roles required by AWS DMS.

Main roles include:

* `dms-vpc-role`
* `dms-cloudwatch-logs-role`

Additional roles can be created by DMS for accessing the source, target, and migration resources.

## Security

Security Groups are configured to allow the required MySQL traffic between:

```text
EC2
 ↓
AWS DMS
 ↓
RDS
```

MySQL uses port:

```text
3306
```

## Verification

After migration, connect to the RDS MySQL database and verify the migrated data.

```sql
SHOW DATABASES;

USE company_test_db;

SHOW TABLES;

SELECT * FROM departments;

SELECT * FROM employees;
```

Expected tables:

```text
departments
employees
```

## Result

The MySQL database originally running on EC2 is successfully migrated to Amazon RDS using AWS DMS.

```text
EC2 MySQL Database
        ↓
     AWS DMS
        ↓
RDS MySQL Database
```

## Key Learning

Through this project, I learned:

* How to install and configure MySQL on EC2
* How to create databases and tables
* How to create an Amazon RDS MySQL database
* How IAM roles are used with AWS DMS
* How to configure source and target databases
* How AWS DMS performs database migration
* How to configure Security Groups for database connectivity
* How to verify migrated database data in RDS

## Technologies

```text
AWS
EC2
RDS
AWS DMS
IAM
Secrets Manager
CloudWatch
MySQL
Linux
```
