# 🚀 FastAPI Production Deployment on AWS (ALB + ASG + RDS)
---

## 📌 Overview
This project demonstrates a **production-style deployment** of a FastAPI application on AWS using:

* VPC with Public & Private Subnets (Multi-AZ)
* Application Load Balancer (ALB)
* Auto Scaling Group (ASG)
* EC2 (Private Instances)
* Bastion Host (Public Instance)
* RDS MySQL (Private)
* Nginx + Gunicorn (App Layer)
---

## 🏗️ Architecture
<img src="./diagrams/AWS-3-Tier-Architecture-(FastAPI + ALB + RDS).png" />

### 🔄 Traffic Flow
1. User sends request to ALB over **HTTP/HTTPS**
2. ALB forwards request to EC2 instances over **HTTP (port 80)**
3. Nginx proxies request to Gunicorn
4. Gunicorn runs FastAPI application
5. FastAPI communicates with RDS over **MySQL (port 3306)**

### Flow Representation
```
User → ALB (HTTP/HTTPS) → EC2 (HTTP 80 via Nginx) → Gunicorn → FastAPI → RDS (MySQL 3306)
```
---

## 🌐 Infrastructure Setup
### 1. VPC
* CIDR: `172.16.0.0/16`
---

### 2. Subnets
```
| AZ | Type    | CIDR          |
| -- | ------- | ------------- |
| 1a | Public  | 172.16.1.0/24 |
| 1a | Private | 172.16.2.0/24 |
| 1b | Public  | 172.16.3.0/24 |
| 1b | Private | 172.16.4.0/24 |
```
---

### 3. Networking
* Internet Gateway attached to VPC
* NAT Gateways:
  * One per AZ (for high availability)
* Route Tables:
  * Public Subnets → Internet Gateway
  * Private Subnets → NAT Gateway
---

## 🔐 Security Groups
### 🟢 Bastion SG
* SSH (22) → Your IP (/32)
**Purpose:**
* Secure SSH access to private EC2 instances
* No direct public access to application servers
---

### 🌐 ALB SG
* HTTP (80) → 0.0.0.0/0
* HTTPS (443) → 0.0.0.0/0
---

### 🖥️ Private EC2 SG
* SSH (22) → Bastion SG
* HTTP (80) → ALB SG
---

### 🗄️ RDS SG
* MySQL (3306) → Private EC2 SG
---

## 🗄️ Database (RDS)
* MySQL (Single-AZ)
* Hosted in private subnets
* Public access: Disabled
* Accessible only via Security Groups
---

## ⚙️ Launch Template (Auto Deployment)
Includes:
* Python, pip, nginx installation
* Application cloning from GitHub
* Virtual environment setup
* Gunicorn setup
* Nginx reverse proxy configuration
* `.env` file creation
* Systemd service for auto-start
---

## 🔁 Auto Scaling Group
* Desired Capacity: 2
* Minimum: 1
* Maximum: 5
* Multi-AZ deployment
---

## ⚖️ Load Balancer (ALB)
* Internet-facing
* Distributes traffic across multiple AZs
* Target Group uses **HTTP (port 80)**
---

## 🧪 Health Check Configuration
### ❌ Initial Problem
* Health check path: `/`
* Result:
  * 404 (route not found)
  * 502 (application crash)
---

### ✅ Fix
Created a dedicated health endpoint:
```python
@app.get("/health")
def health():
    return {"status": "ok"}
```

Updated Target Group:
```
Path: /health
```
---

# 🐛 Issues Faced & Fixes
## 1. ❌ 502 Bad Gateway (ALB)
**Cause:**
* Gunicorn not running or app crashing

**Fix:**
```bash
curl http://127.0.0.1:8000
```
---

## 2. ❌ NAT Gateway Misconfiguration
**Cause:**
* NAT placed in private subnet

**Fix:**
* Moved NAT Gateway to public subnet
* Attached Elastic IP
---

## 3. ❌ No Internet in Private Subnet
**Cause:**
* Incorrect route table

**Fix:**
* Added NAT Gateway route (0.0.0.0/0)
---

## 4. ❌ Permission Issues (.env / venv)
**Fix:**
```bash
sudo chown -R ubuntu:ubuntu /var/www/html/e-commerce-fastapi
chmod 644 .env
```
---

## 5. ❌ .env Not Loading
**Fix:**
```bash
EnvironmentFile=/path/to/.env
```
---

## 6. ❌ Database Connection Error
**Cause:**
* Using localhost instead of RDS

**Fix:**
```
mysql+pymysql://user:password@<RDS-ENDPOINT>:3306/db
```
---

## 7. ❌ MySQL Access Denied
**Fix:**
```sql
GRANT ALL PRIVILEGES ON db.* TO 'user'@'%';
FLUSH PRIVILEGES;
```

---

## 8. ❌ Missing Dependency (pymysql)
**Fix:**
```bash
pip install pymysql
```

---

## 9. ❌ Swagger `/docs` Not Loading
**Cause:**
* Backend errors affecting `/openapi.json`

**Fix:**
* Fixed database + model issues

---

## 10. ❌ Inconsistent ASG Instances
**Cause:**
* Manual setup differences

**Fix:**
* Moved all setup into Launch Template (UserData)

---

## 11. ❌ Wrong Health Check Endpoint
**Fix:**
* Use `/health` instead of `/docs`

---

# 🔍 Debugging Commands
```bash
curl http://127.0.0.1
curl http://127.0.0.1:8000
journalctl -u fastapi -f
sudo tail -f /var/log/nginx/error.log
sudo systemctl status fastapi
nc -zv <RDS-ENDPOINT> 3306
```

---

# ✅ Final Outcome
* ✅ Highly available multi-AZ architecture
* ✅ Auto Scaling enabled
* ✅ Secure private networking
* ✅ Load balanced FastAPI application
* ✅ RDS integration working
* ✅ Fully automated deployment

---

# 🙌 Conclusion
This project demonstrates a **real-world production deployment** of a FastAPI application on AWS, covering networking, security, scalability, and debugging.

Traffic enters through an internet-facing ALB over HTTP/HTTPS, is routed to private EC2 instances over HTTP (port 80), and those instances communicate with RDS over MySQL (port 3306), with access strictly controlled using security groups.

---

## 🧑‍💻 Author
**Rishabh Srivastava**

---

## ⭐ If you found this useful
Give this repo a ⭐ and feel free to fork!
