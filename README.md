# Production-Ready AWS VPC Architecture (Public & Private Subnets)

> Highly-available VPC with public/private subnets, NAT, ALB, Auto Scaling, and a Bastion Host. Backend app runs on **port 8000** in private subnets and is reached via **Application Load Balancer**.

![Architecture](images/arch.png)  
![Architecture](images/architecture-diagram.png)

---

## 📌 What You’ll Build
- **Custom VPC** across **2 Availability Zones**
- **Public subnets** (ALB + NAT Gateways)
- **Private subnets** (EC2 in an **Auto Scaling Group**)
- **Bastion Host** for secure SSH into private EC2
- **Security Groups** with least-privilege rules
- App served on **port 8000** behind ALB

---

## 🚦 Prereqs
- AWS Account + IAM permissions (VPC, EC2, ALB, Auto Scaling, EIP)
- AWS CLI configured: `aws configure`
- Key pair downloaded (`.pem`)

---

## 🧭 Step-by-Step (with your screenshots)

### 1) Create VPC
Create a VPC (e.g., `10.0.0.0/16`) with 2 public and 2 private subnets (spread across AZs).

![Step 1](images/1create vpc.png)  
![Step 1](images/create vpc (2).png)

---

### 2) Internet Gateway & NAT Gateway
- Attach **Internet Gateway** to the VPC.
- Create **NAT Gateway** in each public subnet (or one NAT for cost).

![Step 2](images/3.png)  
![Step 2](images/5.png)  
![Step 2](images/6.png)  
![Step 2](images/7.png)  
![Step 2](images/8.png)

---

### 3) Route Tables
- Public subnets route `0.0.0.0/0` → **Internet Gateway**  
- Private subnets route `0.0.0.0/0` → **NAT Gateway**

![Step 3](images/5.png)  
![Step 3](images/6.png)

---

### 4) Launch Template (user data starts the app on :8000)
Create a Launch Template that installs Python and serves a simple page on **port 8000**.

![Step 4](images/4 create launch template.png)

**Suggested User Data**
```bash
#!/bin/bash
apt-get update -y || yum update -y
apt-get install -y python3 || yum install -y python3
cd /home/ubuntu || cd /home/ec2-user
echo "<h1>Private EC2 behind ALB (port 8000)</h1>" > index.html
nohup python3 -m http.server 8000 >/var/log/http8000.log 2>&1 &
