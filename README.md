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


<p align="center">
  <img src="Image 2 Public Route.png" alt="Architecture Diagram" width="1000"/>
</p>

*Figure 2: Public route table configuration, routing all outbound traffic (0.0.0.0/0) to the Internet Gateway*

### 6.4 Subnet Configuration

Six subnets were created across two Availability Zones.

| Subnet Name | IPv4 CIDR Block | Availability Zone |
|--------------|----------------|-------------------|
| `the-orchid-vpc-publicsubnet1` | `10.0.1.0/24` | `eu-north-1a` |
| `the-orchid-vpc-publicsubnet2` | `10.0.2.0/24` | `eu-north-1b` |
| `the-orchid-vpc-appsubnet1` | `10.0.10.0/24` | `eu-north-1a` |
| `the-orchid-vpc-appsubnet2` | `10.0.11.0/24` | `eu-north-1b` |
| `the-orchid-vpc-datasubnet1` | `10.0.20.0/24` | `eu-north-1a` |
| `the-orchid-vpc-datasubnet2` | `10.0.21.0/24` | `eu-north-1b` |



The two public subnets were associated with the `public-rt` route table created in **Section 6.3**, allowing resources within those subnets to communicate with the internet through the Internet Gateway.


### 6.5 NAT Gateway

A **NAT Gateway** was deployed in one of the public subnets to provide outbound internet access for resources located in the private application and database subnets.

Because the NAT Gateway is internet-facing, an **Elastic IP (EIP)** was allocated and associated with it.

> **Cost Optimization**
>
> The architecture diagram illustrates **two NAT Gateways** (one per Availability Zone), which is AWS's recommended production architecture for high availability and fault tolerance.
>
> For this project, however, **only one NAT Gateway** was deployed to reduce operating costs. NAT Gateways incur **hourly charges** and are **not included in the AWS Free Tier**, making a single NAT Gateway a practical compromise for a learning environment.

<p align="center">
  <img src="Image 4 NAT gateway.png" alt="Architecture Diagram" width="1000"/>
</p>

*Figure 3: NAT Gateway created in a public subnet with an associated Elastic IP address*

### 6.6 Private Route Table (Main Route Table)

To enable resources in the private subnets to access the internet for software updates, package installations, and other outbound traffic, the VPC's **main route table** (which is implicitly associated with the application and data subnets) was updated with the following route:

| Destination | Target |
|-------------|--------|
| `0.0.0.0/0` | NAT Gateway |

With this route in place, resources in the private subnets can initiate outbound connections through the **NAT Gateway** while remaining inaccessible from the public internet.

> **Note**
>
> Unlike the public subnets, the private subnets do **not** have a direct route to the Internet Gateway. All outbound internet traffic is routed through the NAT Gateway, preserving the security of the private network while still allowing access to external services when required.



### 6.7 Security Groups

Three security groups were created to enforce network access controls at the instance level. Together, they implement a **security group chaining** model, ensuring that each tier accepts traffic only from the layer immediately in front of it.

#### a) Application Load Balancer Security Group

| Setting | Value |
|----------|-------|
| **Name** | `the-orchard-vpc-alb-sg` |
| **Description** | Allow HTTP traffic from the internet to the Application Load Balancer |
| **VPC** | `the-orchard-vpc` |

##### Inbound Rules

| Type | Protocol | Port | Source |
|------|----------|------|--------|
| HTTP | TCP | `80` | `0.0.0.0/0` (Anywhere) |

<p align="center">
  <img src="Image 6 SG.png" alt="Architecture Diagram" width="1000"/>
</p>

*Figure 4: Security group creation in the AWS Console, showing the inbound HTTP rule for the ALB security group*

#### b) Application Security Group

| Setting | Value |
|----------|-------|
| **Name** | `the-orchard-vpc-app-sg` |
| **Description** | Allow HTTP traffic from the Application Load Balancer to the application servers |
| **VPC** | `the-orchard-vpc` |

##### Inbound Rules

| Type | Protocol | Port | Source |
|------|----------|------|--------|
| HTTP | TCP | `80` | `the-orchard-vpc-alb-sg` |

Only the Application Load Balancer is permitted to send HTTP traffic to the EC2 instances. This prevents users from accessing the application servers directly over the internet.

#### c) Database Security Group

| Setting | Value |
|----------|-------|
| **Name** | `the-orchard-vpc-data-sg` |
| **Description** | Allow MySQL traffic from the application servers to the database |
| **VPC** | `the-orchard-vpc` |

##### Inbound Rules

| Type | Protocol | Port | Source |
|------|----------|------|--------|
| MySQL/Aurora | TCP | `3306` | `the-orchard-vpc-app-sg` |

The database accepts connections **only** from EC2 instances associated with the application security group, ensuring that it remains isolated from both the public internet and other resources within the VPC.

---

### 6.8 Amazon S3 Bucket for Application Source Code

An Amazon S3 bucket was created to store the application's source code and deployment assets.

| Setting | Value |
|----------|-------|
| **Name** | `the-orchid-app-sourcecode` |
| **Block Public Access** | Enabled |
| **Versioning** | Disabled |

Public access blocking was enabled because the application source code and other objects stored in the bucket are confidential and should not be accessible directly from the internet.

> **Versioning Note**
>
> Versioning was intentionally left **disabled** for this learning project to minimise storage costs. While Amazon S3 Versioning itself has no additional charge, storing multiple versions of objects increases storage consumption over time.
>
> In a production environment, **versioning should be enabled**. It preserves previous versions of objects, providing protection against accidental deletion, unintended overwrites, application errors, and simplifying data recovery when changes need to be rolled back.

<p align="center">
  <img src="Image 7 app sourcecode.png" alt="Architecture Diagram" width="1000"/>
</p>

*Figure 5: Application source files and assets uploaded to the S3 bucket.*

### 6.9 IAM Roles and Policies

An IAM role is required so that when the Auto Scaling Group launches new EC2 instances, each instance can fetch the application source code from S3 and communicate with Systems Manager. The role has two policies attached: an S3 access policy, and the AWS-managed `AmazonSSMManagedInstanceCore` policy for Session Manager.

The Systems Manager policy was attached using the AWS-managed role designed for this purpose. In the IAM console, under **Roles**, choosing **AWS service** and then **EC2 role for Systems Manager** pre-selects the `AmazonSSMManagedInstanceCore` policy.

Rather than granting broad administrative permissions, this policy provides only the permissions required for the Systems Manager agent running on the EC2 instance to register with AWS Systems Manager, establish secure Session Manager connections, receive management commands, and report instance information back to AWS.

This allows administrators to manage the instances securely without exposing SSH (port 22) to the internet or maintaining SSH key pairs, while aligning with AWS security best practices by using temporary IAM credentials instead of long-lived access keys.

<p align="center">
  <img src="Image 10 IAM roles.png" alt="Architecture Diagram" width="1000"/>
</p>

*Figure 6: The completed IAM role with both policies — S3 access and AmazonSSMManagedInstanceCore — attached.*

### 6.10 Database Creation and Configuration

The database stores every recipe and piece of information submitted by site visitors. Before creating a highly available RDS instance, a **DB subnet group** is required—a named collection of subnets in the VPC that RDS can place the database and its standby copy into. It is essential for Multi-AZ deployments.

| Setting | Value |
|----------|-------|
| **Name** | `the-orchard-db-subnet-group` |
| **Description** | Subnet group for the RDS database |
| **VPC** | `the-orchard-vpc` |
| **Availability Zones** | `eu-north-1a`, `eu-north-1b` |
| **Subnets** | `the-orchid-vpc-datasubnet1`, `the-orchid-vpc-datasubnet2` |

The RDS database instance itself was then created with the following configuration:

| Setting | Value |
|----------|-------|
| **Name** | `the-orchard-db` |
| **Engine** | MySQL |
| **Template** | Free Tier |
| **Availability & Durability** | Single DB instance |
| **Credentials** | Self-managed username and password |
| **Instance Class** | Burstable classes — `db.t3.micro` |
| **VPC** | `the-orchard-vpc` |
| **Subnet Group** | `the-orchard-db-subnet-group` |
| **Public Access** | Disabled |
| **Security Group** | `the-orchard-vpc-data-sg` |

> **Free Tier Constraint**
>
> The **Free Tier** template does not support a **Dev/Test** environment or a **Multi-AZ** deployment—only a **Single DB Instance** is available. Recent changes to AWS account eligibility rules made the Dev/Test tier unavailable for this account, so the Free Tier template was used instead.
>
> In a production environment, the **Dev/Test** or **Production** template should be used with a **Multi-AZ DB instance** (or **Multi-AZ DB cluster**) to match the architecture shown in the diagram.


### 6.11 Target Group and Application Load Balancer

The load balancer is the single point of contact for client traffic. It distributes incoming requests across multiple targets—in this case, the EC2 instances in both Availability Zones—which improves fault tolerance and keeps the application available if an individual instance becomes unavailable.

A target group was created first to define which resources the load balancer forwards traffic to.

| Setting | Value |
|----------|-------|
| **Target Type** | Instances |
| **Name** | `the-orchard-tg` |
| **VPC** | `the-orchard-vpc` |
| **Protocol Version** | HTTP1 |
| **Health Check Path** | `/health.html` |
| **Healthy Threshold** | 3 |
| **Unhealthy Threshold** | 2 |
| **Interval** | 10 seconds |

An internet-facing **Application Load Balancer** was then created in `the-orchard-vpc`, listening for incoming traffic and forwarding requests to the `the-orchard-tg` target group.

<p align="center">
  <img src="Image 11 ALB.png" alt="Architecture Diagram" width="1000"/>
</p>

*Figure 7: Application Load Balancer configuration — internet-facing, deployed in the-orchard-vpc.*

| Setting | Value |
|----------|-------|
| **Subnets** | `the-orchid-vpc-publicsubnet1`, `the-orchid-vpc-publicsubnet2` |
| **Security Group** | `the-orchard-vpc-alb-sg` |
| **Listener** | HTTP :80 → `the-orchard-tg` |

---

## 6.12 Launch Template

Before configuring Auto Scaling, a **Launch Template** was created to define exactly how new instances should be configured on launch.

| Setting | Value |
|----------|-------|
| **Name** | `the-orchard-app-lt` |
| **AMI** | Amazon Linux 2023 |
| **Instance Type** | `t2.micro` |
| **Security Group** | `the-orchard-vpc-app-sg` |
| **IAM Instance Profile** | The IAM role from **Section 6.9** (S3 read access + Systems Manager) |

The following user data script runs automatically on first boot, installing and configuring the web stack before pulling the application files from Amazon S3:

```bash
#!/bin/bash

# Update the package repository
dnf update -y

# Install the necessary packages
dnf install -y httpd php php-mysqli mariadb105

# Start and enable the Apache service
systemctl start httpd
systemctl enable httpd

# Add the ec2-user to the apache group
usermod -a -G apache ec2-user

# Set ownership and permissions for the web directory
chown -R ec2-user:apache /var/www
chmod 2775 /var/www

# Set permissions for directories and files within /var/www
find /var/www -type d -exec chmod 2775 {} \;
find /var/www -type f -exec chmod 0664 {} \;

# Copy files from the S3 bucket to the web directory
aws s3 cp s3://s3://the-orchid-app-sourcecode /var/www/html --recursive
```

---

## 6.13 Auto Scaling Group

An Auto Scaling Group was configured using the Launch Template above.

<p align="center">
  <img src="Img 12 Lt.png" alt="Architecture Diagram" width="1000"/>
</p>

*Figure 11: Auto Scaling Group configuration, referencing the-orchard-app-lt launch template..*

The Auto Scaling Group was configured with the following settings:

| Setting | Value |
|----------|-------|
| **VPC** | `the-orchard-vpc` |
| **Subnets** | `the-orchid-vpc-appsubnet1`, `the-orchid-vpc-appsubnet2` |
| **Target Group** | `the-orchard-tg` |
| **Health Checks** | ELB health checks enabled |

<p align="center">
  <img src="Img 13 autoscaling.png" alt="Architecture Diagram" width="1000"/>
</p>

*Figure 11: Auto Scaling Group csuccessfully created.*

## 7. Testing and Validation

With the DNS name of the load balancer and the health-check file path appended, the application loaded successfully, confirming that the Application Load Balancer, Target Group, and Auto Scaling Group were all configured and integrated correctly.

<p align="center">
  <img src="Img 14 appload.png" alt="Architecture Diagram" width="1000"/>
</p>

*Figure 12: The application loading successfully through the Application Load Balancer's DNS name.*

Several test recipe submissions were made through the site to mimic real customer activity. 

<p align="center">
  <img src="img 15 submission.png" alt="Architecture Diagram" width="1000"/>
</p>

*Figure 13: A test recipe submission through the application's front end.*

After submitting, the data was retrieved back from the RDS database and displayed on the page, confirming the application, database, and IAM permissions were all working end to end:

<p align="center">
  <img src="img 15 submission.png" alt="Architecture Diagram" width="1000"/>
</p>

*Figure 13: A test recipe submission through the application's front end.*




















