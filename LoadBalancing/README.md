# AWS ALB + Auto Scaling + Private EC2 Architecture

This project demonstrates a highly available and scalable architecture using AWS services including VPC, Application Load Balancer, Auto Scaling Group, and EC2 instances deployed in private subnets.

---

## 📍 Region
* London (`eu-west-2`)

---

## 🏗️ Infrastructure Setup (Step-by-Step)

### 1. VPC Creation
* Created VPC **Demo-VPC** with CIDR block `10.0.0.0/16`
* This acts as the isolated network for all resources

---

### 2. Subnets Creation
Created 4 subnets across 2 Availability Zones for high availability:
* Public Subnets:
  * `Demo-Public-Subnet-2a-AZ` → `10.0.1.0/24`
  * `Demo-Public-Subnet-2b-AZ` → `10.0.3.0/24`

* Private Subnets:
  * `Demo-Private-Subnet-2a-AZ` → `10.0.2.0/24`
  * `Demo-Private-Subnet-2b-AZ` → `10.0.4.0/24`
---

### 3. Route Tables
Created two route tables:
* **Demo-Public-Subnet-RT**
  * Associated with public subnets
  * Route:
    * `0.0.0.0/0 → Internet Gateway`

* **Demo-Private-Subnet-RT**
  * Associated with private subnets
  * No direct internet access (secure layer)
---

### 4. Internet Gateway
* Created **Demo-IGW**
* Attached to VPC to enable internet access for public subnets
---

## 🔐 Security Groups
### 5. Bastion Security Group (Demo-Bastion-SG)
* Purpose: SSH access to private instances via bastion
| Type | Port | Source    |
| ---- | ---- | --------- |
| SSH  | 22   | 0.0.0.0/0 |
---

### 6. ALB Security Group (Demo-ALB-SG)
* Purpose: Allow public traffic to Load Balancer
| Type | Port | Source    |
| ---- | ---- | --------- |
| HTTP | 80   | 0.0.0.0/0 |
---

### 7. EC2 Private Security Group (Demo-EC2-Private-SG)
* Purpose: Secure backend instances
| Type | Port | Source          |
| ---- | ---- | --------------- |
| HTTP | 80   | Demo-ALB-SG     |
| SSH  | 22   | Demo-Bastion-SG |

👉 Ensures:
* Only ALB can send traffic to EC2
* Only bastion can SSH into EC2
---

## ⚖️ Load Balancing
### 8. Target Group
* Name: `Demo-Target-Group`
* Target Type: Instance
* Protocol: HTTP (Port 80)
* Health Check: HTTP `/`
---

### 9. Application Load Balancer
* Name: `Demo-Public-ALB`
* Scheme: Internet-facing
* Subnets:
  * Public Subnet 2a
  * Public Subnet 2b
* Security Group: `Demo-ALB-SG`

#### Listener:
* Protocol: HTTP (Port 80)
* Default action: Forward to Target Group
---

## 🖥️ Compute Layer
### 10. Launch Template
* Name: `Apache2-Configured-Instance-Template`
* AMI: Ubuntu
* Instance Type: `t2.micro`
* Security Group: `Demo-EC2-Private-SG`
* Public IP: ❌ Disabled (private instances)
* Storage: 8 GB (default)

#### User Data Script:
```bash
#!/bin/bash
sudo apt update -y
sudo apt install apache2 -y
sudo systemctl start apache2
sudo systemctl enable apache2
echo "Healthy Instance" > /var/www/html/index.html
```
---

### 11. Auto Scaling Group (ASG)
* Name: `Demo-ASG`
* Launch Template: Attached
* Subnets: Private (2a & 2b)
* Desired Capacity: 2
* Min: 1
* Max: 5

#### Load Balancing:
* Attached to Target Group

#### Health Checks:
* Enabled ELB health checks
* Grace period: 300 seconds
---

## 🔄 Architecture Flow
```
Internet
   ↓
Application Load Balancer (Public Subnets)
   ↓
Target Group
   ↓
Auto Scaling Group
   ↓
EC2 Instances (Private Subnets)
```
---

## ❗ Issues Faced & Fixes
### 🔴 Issue 1: Target Group showing Unhealthy
**Problem:**
* Instances were running but marked unhealthy

**Root Cause:**
* Application (Apache) was not properly installed or responding
**Fix:**
* Added proper user-data script in Launch Template
* Ensured health check path `/` returns HTTP 200
---

### 🔴 Issue 2: SSH to Private EC2 Failed (Timeout)
**Problem:**
```
connection timed out
```

**Root Cause:**
* Bastion host was in a different VPC
* Security group rules were misconfigured

**Fix:**
* Ensured Bastion and EC2 are in same VPC
* Allowed SSH (22) from Bastion SG in EC2 SG
---

### 🔴 Issue 3: Security Group Misconfiguration
**Problem:**
* Same SG used for ALB and EC2
* ALB not accessible from internet

**Fix:**
* Created separate SGs:
  * ALB SG → allows internet traffic
  * EC2 SG → allows only ALB traffic
---

### 🔴 Issue 4: Wrong Networking Design
**Problem:**
* Public IP enabled on private EC2
* Incorrect SG chaining

**Fix:**
* Disabled public IP for private instances
* Implemented proper layered security model
---

## 🧠 Key Learnings
* ALB communicates with EC2 via Target Group, not directly with ASG
* Security Groups should be layered (ALB → EC2)
* Private EC2 instances should not have public IPs
* Health checks validate application, not just instance status
* Bastion host must be in same VPC for SSH access
---

## 📌 Conclusion
This setup demonstrates a scalable, secure, and highly available AWS architecture using best practices such as:
* Multi-AZ deployment
* Private compute layer
* Load balancing
* Auto scaling
---

## 📷 Architecture Diagram
<img src="./diagrams/AWS-ALB--Auto Scaling--Private-EC2-Architecture.png" />

## 🧑‍💻 Author
**Rishabh Srivastava**
---

## ⭐ If you found this useful
Give this repo a ⭐ and feel free to fork!
