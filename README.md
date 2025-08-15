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
## 5) Auto Scaling Group (ASG)

Create an ASG spanning the private subnets (**min=2, desired=2, max=4**).

![Step 5](images/9auto scalling group.png)  
![Step 5](images/10auto scalling group.png)  

---

## 6) Bastion Host (in Public Subnet)

Launch a small EC2 (e.g., **t2.micro**) in a public subnet with a public IP to SSH into private EC2.

![Step 6](images/11 bastion host.png)  
![Step 6](images/12 bastion host.png)  

---

## 7) Security Groups

- **ALB SG:** Inbound `80 (HTTP)` from `0.0.0.0/0`; outbound to targets.  
- **EC2/ASG SG:** Inbound `8000` only from ALB SG; allow SSH `22` only from Bastion SG.  
- **Bastion SG:** Inbound `22` from your IP.  

![Step 7](images/19 security group.png)  

---

## 8) Target Group & Register Targets

Create a target group (**HTTP port 8000**) and register the ASG instances (or let ASG attach automatically).

![Step 8](images/13 target group.png)  
![Step 8](images/14target group.png)  
![Step 8](images/15target group.png)  
![Step 8](images/16 target group.png)  

---

## 9) Application Load Balancer (ALB)

Create an ALB in the two public subnets and add a listener on **port 80** forwarding to the target group.

![Step 9](images/17 load balancer.png)  
![Step 9](images/18 load balancer.png)  
![Step 9](images/20 load balancer.png)  

---

## 🔟 Test via Bastion + Private EC2 (SSH)

From your local machine → Bastion (public) → Private EC2 (via private IP):

![Step 10](images/21 ssh.connection.png)  

```bash
# Fix key permissions locally (required by SSH)
chmod 400 aws_login.pem

# SSH to Bastion (public IP/DNS)
ssh -i aws_login.pem ubuntu@<bastion-public-ip>

# From Bastion, SSH to a Private EC2 (use its private IP)
ssh -i aws_login.pem ubuntu@10.0.x.x

```bash

