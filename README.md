# Optimizing Web App High Availability with AWS ALB and EC2 ⚖️

## 📌 Project Overview
This repository contains the architecture and implementation details for a highly available, fault-tolerant two-tier web application architecture. The project demonstrates how an AWS Application Load Balancer (ALB) operating at Layer 7 of the OSI model efficiently routes client traffic across decoupled EC2 compute nodes situated across distinct Availability Zones (AZs).

## 🏗️ Architecture Design
The infrastructure is engineered to prevent single points of failure and gracefully handle variable user traffic:
* **Elastic Load Balancing:** An internet-facing AWS ALB handles incoming public HTTP requests.
* **Multi-AZ Compute:** Two EC2 instances (t2.micro running Amazon Linux) are deployed across separate Availability Zones (AZ-1 and AZ-2).
* **Target Group Optimization:** Traffic is mapped to target group `tg-01` over HTTP Port 80 using HTTP/1.1 protocols.

## 🔒 Layered Security Architecture
To balance user accessibility with robust network hardening, the following security measures were implemented:

### 1. Security Group Configuration
* **Inbound Rules:** Strictly isolated to minimize attack surfaces.
  * **HTTP (Port 80):** Allowed from anywhere (`0.0.0.0/0`) to route public web traffic.
  * **SSH (Port 22):** Restricted access enabled exclusively for remote server administration.
* **Outbound Rules:** Set to allow all destinations by default to permit necessary system updates (`yum update`) and external API dependencies.

### 2. Access Authentication
* Enforced **Public Key Authentication** via asymmetric key-pairs.
* Confidential private keys are maintained client-side, while public keys are securely distributed into the native `authorized_keys` configurations on the EC2 instances.

## 🚀 Deployment & Automation Scripts
To achieve immediate web-hosting capabilities right at boot time, an automated shell script was passed into the **EC2 User Data** payload during instance initialization:

```bash
#!/bin/bash
# Update core system packages
sudo yum update -y

# Install and launch Apache Web Server
sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd

# Inject unique instance metadata to verify load balancing execution
echo "Hello from EC2 Instance - Managed via ALB Routing" | sudo tee /var/www/html/index.html
