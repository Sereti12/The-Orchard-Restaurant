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

The diagram shows six subnets split across two Availability Zones: two public subnets (Internet Gateway and NAT Gateway), two application subnets (EC2 instances behind the load balancer), and two data subnets (the RDS database). This split keeps the database completely unreachable from the internet, while still letting the application tier reach out for updates and dependencies through the NAT Gateway.

## 3. Business Background

**The Orchard** is a fictional chain of coffee shops and restaurants in the United Kingdom offering a vegetarian and plant-based menu. During a planning meeting, the marketing team identified an opportunity to expand the brand's customer base through a community-driven campaign.

Their proposal was to invite members of the public to submit their favourite vegan recipes through a website. Each participant may submit **one recipe**, which is reviewed by the kitchen team. The winning recipe is published on the company website and added to the restaurant's menu.

The campaign is designed to:

- Incentivise customers to follow the company's social media channels
- Increase brand awareness
- Grow the email subscriber list
- Increase sales
- Improve customer engagement

### Why a Server-Based Architecture?

Although the IT department recognised the benefits of a serverless architecture, this project intentionally uses an **Amazon EC2-based** design for three primary reasons:

- Full control over the operating system and installed software
- Support for long-running background processes
- More predictable costs under sustained workloads

Serverless services are still used where appropriate—most notably **Amazon S3** for storing application files.


## 4. Architecture Components

This section explains the purpose of every major component in the architecture before walking through the implementation.

### 4.1 Network — Amazon VPC

A custom **Amazon VPC** hosts every resource in the project. The VPC spans **two Availability Zones** and contains **six subnets**, allowing public-facing resources, application servers, and the database tier to remain isolated at the network level.

### 4.2 Subnets

The architecture uses six subnets:

| Subnet Type | Purpose |
|-------------|---------|
| **Public (2)** | Hosts the Application Load Balancer and NAT Gateway. Reachable from the internet through the Internet Gateway. |
| **Application (2)** | Hosts the EC2 web servers. Not directly accessible from the internet. |
| **Data (2)** | Hosts the Amazon RDS database. Accessible only from the application layer. |

This follows AWS's standard **three-tier subnet architecture**.

### 4.3 Internet Gateway

The **Internet Gateway (IGW)** is attached to the VPC and enables communication between resources inside the VPC and the public internet.

Without an Internet Gateway, resources inside the VPC cannot communicate with the internet.

### 4.4 NAT Gateway

A **NAT Gateway**, deployed in a public subnet, allows resources in private subnets to initiate outbound internet connections while preventing unsolicited inbound traffic.

Typical use cases include:

- Downloading operating system updates
- Installing software packages
- Retrieving security patches

### 4.5 Compute — Amazon EC2, Launch Templates & Auto Scaling

Two Linux web servers run across separate Availability Zones.

A **Launch Template** defines:

- Amazon Machine Image (AMI)
- Instance type
- Security group
- IAM role
- User data bootstrap script

An **Auto Scaling Group (ASG)** uses this template to:

- Maintain the desired number of EC2 instances
- Replace unhealthy instances automatically
- Distribute instances across both Availability Zones

### 4.6 Load Balancing — Application Load Balancer

An internet-facing **Application Load Balancer (ALB)** serves as the single entry point into the application.

It:

- Receives all incoming HTTP requests
- Performs health checks
- Routes traffic only to healthy EC2 instances
- Distributes traffic across multiple Availability Zones

### 4.7 Storage — Amazon S3

Amazon S3 stores the application's source files, including:

- HTML
- CSS
- JavaScript
- PHP

Keeping application code separate from compute resources allows new EC2 instances to automatically retrieve the latest version during startup, making the architecture more loosely coupled.

### 4.8 Database — Amazon RDS (MySQL)

An **Amazon RDS for MySQL** instance stores every recipe submitted through the application.

The architecture supports a **Multi-AZ deployment**, allowing a standby database in another Availability Zone to automatically take over if the primary database becomes unavailable.

### 4.9 Security Groups

Three security groups provide instance-level firewall protection using a **security group chaining** approach.

| Security Group | Allowed Traffic |
|----------------|-----------------|
| **ALB Security Group** | HTTP (Port 80) from the internet (`0.0.0.0/0`) |
| **Application Security Group** | HTTP (Port 80) only from the ALB Security Group |
| **Database Security Group** | MySQL (Port 3306) only from the Application Security Group |

This ensures that each layer only accepts traffic from the layer immediately in front of it.

### 4.10 IAM Roles and Policies

IAM roles provide temporary credentials to AWS resources while IAM policies define the permissions granted.

This project follows the **Principle of Least Privilege**.

The EC2 instances are assigned an IAM role containing:

- **Custom S3 access policy** – Allows instances to retrieve application files from Amazon S3 during boot.
- **AmazonSSMManagedInstanceCore** – Enables communication with AWS Systems Manager.

### 4.11 AWS Systems Manager — Session Manager

AWS Systems Manager **Session Manager** provides secure browser-based shell access to EC2 instances without requiring:

- SSH (Port 22)
- SSH key pairs
- Bastion hosts

Because the EC2 instances reside in private subnets, Session Manager is the only administrative access method used.

Benefits include:

- IAM-based authentication
- No inbound management ports
- Session logging and auditing
- Reduced attack surface


## 5. Prerequisites

Before completing this project, you should have:

- An active AWS account
- Access to the AWS Management Console
- Intermediate-to-advanced familiarity with core AWS services
- Working knowledge of IP addressing and CIDR notation
- Familiarity with IAM roles and IAM policies

## 6. Implementation Walkthrough

This section documents the infrastructure exactly as it was built in the AWS Management Console.

### 6.1 VPC and Subnet Configuration

An Amazon VPC was created with the following configuration.

| Setting | Value |
|----------|-------|
| **Name** | `the-orchard-vpc` |
| **IPv4 CIDR Block** | `10.0.0.0/16` |

The `/16` CIDR block provides approximately **65,536 IP addresses**, leaving significant room for future expansion beyond the six subnets used in this project.

### 6.2 Internet Gateway

An Internet Gateway was created and attached to the VPC.

| Setting | Value |
|----------|-------|
| **Name** | `the-orchard-vpc-igw` |

Its purpose is to enable communication between public resources in the VPC and the internet.

### 6.3 Route Tables

Every VPC includes a default **main route table**, which initially allows only internal VPC communication.

A second route table was created for the public subnets.

| Setting | Value |
|----------|-------|
| **Name** | `public-rt` |
| **Destination** | `0.0.0.0/0` |
| **Target** | `the-orchard-vpc-igw` (Internet Gateway) |

This route allows resources in the public subnets to communicate with the internet.











