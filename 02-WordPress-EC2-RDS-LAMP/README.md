# WordPress Website Hosting on AWS EC2 with RDS – LAMP Stack

## Project Overview

This project demonstrates hosting a WordPress website on Amazon EC2 using the LAMP stack.

LAMP stands for:

- Linux – Amazon Linux 2023
- Apache – Web Server
- MySQL – Amazon RDS MySQL
- PHP – Application Runtime

The project also demonstrates a scalable architecture using Application Load Balancer, Auto Scaling Group, Launch Template, AMI, Route 53, and Blue-Green Deployment concepts.


## Architecture Flow

```text
                         User
                           |
                           v
                        Internet
                           |
                           v
                    Application Load Balancer
                           |
                 +---------+---------+
                 |                   |
                 v                   v
             Blue EC2            Green EC2
             WordPress            WordPress
             LAMP Stack           LAMP Stack
                 |                   |
                 +---------+---------+
                           |
                           v
                    Amazon RDS MySQL
                    WordPress Database
