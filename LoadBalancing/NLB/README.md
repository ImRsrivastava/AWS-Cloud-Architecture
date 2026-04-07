# AWS NLB + Auto Scaling + Private EC2 Architecture

This project demonstrates a highly available and scalable architecture using AWS services including VPC, Network Load Balancer, Auto Scaling Group, and EC2 instances deployed in private subnets.

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
### 5. EC2 Private Security Group (Demo-EC2-Private-SG)
```
* Purpose: Secure backend instances
| Type | Port | Source      |
| ---- | ---- | ----------- |
| HTTP | 80   | 10.0.0.0/16 |
```

👉 Ensures:
* Only traffic from within VPC (including NLB) can reach EC2

⚠️ Note:
* Network Load Balancer does **NOT use Security Groups**
---

## ⚖️ Load Balancing
### 6. Target Group
* Name: `Demo-Target-Group`
* Target Type: Instance
* Protocol: TCP (Port 80)
* Health Check: TCP (Port 80)
---

### 7. Network Load Balancer
* Name: `Demo-NLB`
* Scheme: Internet-facing
* Subnets:
  * Public Subnet 2a
  * Public Subnet 2b

#### Listener:
* Protocol: TCP (Port 80)
* Default action: Forward to Target Group

⚠️ Note:
* NLB does not support Security Groups
---

## 🖥️ Compute Layer
### 8. Launch Template
* Name: `Apache2-Configured-Instance-Template`
* AMI: Custom AMI (Apache pre-installed)
* Instance Type: `t2.micro`
* Security Group: `Demo-EC2-Private-SG`
* Public IP: ❌ Disabled (private instances)
* Storage: 8 GB (default)

#### User Data Script:
```bash
#!/bin/bash
echo "Instance: $(hostname) | IP: $(hostname -I)" > /var/www/html/index.html
```
---

### 9. Auto Scaling Group (ASG)
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
Network Load Balancer (Public Subnets)
   ↓
Target Group
   ↓
Auto Scaling Group
   ↓
EC2 Instances (Private Subnets via ASG)
```
---

## ❗ Issues Faced & Fixes
### 🔴 Issue 1: Apache not installing on private instance
**Problem:**
* User-data script failed

**Root Cause:**
* No internet access (no NAT Gateway)

**Fix:**
* Created custom AMI with Apache pre-installed.
---

### 🔴 Issue 2: Target group showing unhealthy
**Problem:**
* Instances marked unhealthy

**Root Cause:**
* Apache not running / port 80 not open

**Fix:**
* Ensured Apache is installed and running properly
---

### 🔴 Issue 3: Same response from all instances
**Problem:**
* Load balancing not visible

**Root Cause:**
* Same static content baked into AMI

**Fix:**
* Used user-data to generate dynamic content:
```bash
#!/bin/bash
$(hostname) and $(hostname -I)
```
---

### 🔴 Issue 4: NLB not switching traffic like ALB
**Problem:**
* Traffic always hitting same instance

**Root Cause:**
* NLB uses connection-based routing

**Fix:**
* Forced new connections:
```bash
#!/bin/bash
curl -H "Connection: close"
```
---

## 🧠 Key Learnings
* NLB works at Layer 4 (TCP), not Layer 7.
* NLB does not use Security Groups.
* Traffic is distributed per connection, not per request.
* Auto Scaling Group manages instances, not traffic routing.
* Private EC2 instances should not have public IPs.
* Health checks ensure application availability.
---

## 📌 Conclusion
This setup demonstrates a scalable, secure, and highly available AWS architecture using:
* Multi-AZ deployment
* Private compute layer
* Network Load Balancer (Layer 4)
* Auto Scaling Group
* Custom AMI for private environments
---

## 📷 Architecture Diagram
<img src="./diagrams/AWS-NLB--Auto Scaling--Private-EC2-Architecture.png" />
---

## 🧑‍💻 Author
Rishabh Srivastava

---

## ⭐ If you found this useful
Give this repo a ⭐ and feel free to fork!
