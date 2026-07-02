# The Orchard Restaurant

## 1. Project Overview

This project is a self-directed AWS build completed for hands-on practice. It recreates a realistic business scenario in which a restaurant chain needs a web application to run a public recipe submission campaign. The application is designed to remain highly available, even if an individual server or an entire Availability Zone experiences a failure.

The infrastructure is deployed across **two Availability Zones** within a **custom-built Virtual Private Cloud (VPC)** containing **six subnets**. Incoming traffic is distributed through an **Application Load Balancer (ALB)**, while **Amazon EC2 Auto Scaling** automatically adjusts compute capacity based on application demand.

Customer recipe submissions are securely stored in an **Amazon RDS for MySQL** database. Application source code and deployment assets are hosted in **Amazon S3**, and all AWS service interactions follow the **Principle of Least Privilege** through carefully configured **IAM roles** and **security groups**.

### Key Features

- Multi-AZ architecture for high availability
- Custom Amazon VPC with six subnets
- Application Load Balancer (ALB) for traffic distribution
- Amazon EC2 Auto Scaling for automatic scaling
- Amazon RDS MySQL for persistent data storage
- Amazon S3 for application source code storage
- IAM roles implementing the Principle of Least Privilege
- Security Groups providing network-level access control

## Skills Demonstrated

| **Category** | **Technologies & Concepts** |
|--------------|-----------------------------|
| **Networking** | Custom VPC, subnetting, route tables, Internet Gateway, NAT Gateway |
| **Compute & Scaling** | Amazon EC2, Launch Templates, bootstrapping with user data, Auto Scaling |
| **High Availability** | Multi-AZ architecture, Application Load Balancer (ALB), health checks, failover testing |
| **Security & IAM** | Least-privilege IAM roles and policies, security group chaining, AWS Systems Manager Session Manager (no SSH or port 22 exposure) |
| **Database** | Amazon RDS for MySQL, DB subnet groups, private data-tier isolation |
| **Storage** | Amazon S3 for decoupled application source file storage |

## 2. Architecture Diagram

<p align="center">
  <img src="The Vegan Project.png" alt="Architecture Diagram" width="1000"/>
</p>

*Figure 1: High-level architecture. The application runs across two Availability Zones, sitting behind an Application Load Balancer in a public subnet. NAT Gateways in the public subnets give the private application and data subnets outbound internet access. IAM roles and security groups control which resources can talk to each other.*













