# 🏗️ AWS-ALB Project (VPC, ALB, NAT, Private Instances, Auto Scaling)

## 📘 Project Overview

This project demonstrates the creation of a **highly available AWS infrastructure** using the **AWS Management Console**.  
It deploys a **two-tier architecture** with:
- A custom **VPC**,
- **Public** and **Private Subnets** across two Availability Zones,
- **Internet Gateway** and **NAT Gateways** for controlled connectivity,
- An **Application Load Balancer (ALB)** that routes traffic to **private EC2 instances**,
- An optional **Auto Scaling Group (ASG)** for elasticity.

---

## 🧠 Architecture Summary

**Region:** `us-east-1` (N. Virginia)  
**Availability Zones:** `us-east-1a`, `us-east-1b`  

| Layer | Components |
|-------|-------------|
| Networking | VPC, Subnets, Route Tables, IGW, NAT Gateway |
| Security | Security Groups for ALB and EC2 |
| Compute | Launch Template, EC2 Instances, Auto Scaling Group |
| Load Balancing | Application Load Balancer (ALB), Target Group |

---

## ⚙️ Step-by-Step Setup (AWS Console)

---

### 🧩 Step 1: Create a Custom VPC
1. Navigate to **VPC Console → Your VPCs → Create VPC**
2. Choose **VPC only**
3. Set:
   - **Name tag:** `aws-alb-vpc`
   - **IPv4 CIDR block:** `10.0.0.0/16`
   - **Tenancy:** Default
4. Click **Create VPC**

---

### 🌐 Step 2: Create Subnets

| Subnet Name | Type | AZ | CIDR Block |
|--------------|------|----|------------|
| public-sn-1a | Public | us-east-1a | 10.0.1.0/24 |
| public-sn-1b | Public | us-east-1b | 10.0.2.0/24 |
| private-sn-1a | Private | us-east-1a | 10.0.3.0/24 |
| private-sn-1b | Private | us-east-1b | 10.0.4.0/24 |

**Steps:**
1. Go to **VPC → Subnets → Create Subnet**
2. Choose **VPC: aws-alb-vpc**
3. Add all 4 subnets as shown
4. After creation, select `public-sn-1a` and `public-sn-1b`
   - Click **Actions → Modify auto-assign IP settings**
   - Enable **Auto-assign public IPv4 address**

---

### 🚪 Step 3: Create and Attach Internet Gateway
1. Go to **VPC → Internet Gateways → Create Internet Gateway**
   - Name: `aws-alb-igw`
2. Click **Attach to VPC → aws-alb-vpc**

---

### 🛣️ Step 4: Create Route Tables

#### 4.1 Public Route Table
1. **VPC → Route Tables → Create Route Table**
   - Name: `aws-alb-public-rt`
   - VPC: `aws-alb-vpc`
2. Click **Routes → Edit routes → Add route**
   - Destination: `0.0.0.0/0`
   - Target: **Internet Gateway (aws-alb-igw)**
3. Go to **Subnet Associations → Edit**
   - Associate: `public-sn-1a`, `public-sn-1b`

#### 4.2 Private Route Tables
Create **two private route tables** for NAT access later:
- `aws-alb-private-rt-1a` → For subnet `private-sn-1a`
- `aws-alb-private-rt-1b` → For subnet `private-sn-1b`

---

### 🌉 Step 5: Create NAT Gateways
NAT Gateways allow private instances to access the internet.

1. Go to **VPC → NAT Gateways → Create NAT Gateway**
2. **NAT-GW-1a:**
   - Name: `natgw-1a`
   - Subnet: `public-sn-1a`
   - Elastic IP: Allocate new
3. **NAT-GW-1b:**
   - Name: `natgw-1b`
   - Subnet: `public-sn-1b`
   - Elastic IP: Allocate new
4. Click **Create NAT Gateway**
5. Wait until both show **Available**

---

### 🗺️ Step 6: Update Private Route Tables

#### For `aws-alb-private-rt-1a`
1. Go to **Routes → Edit routes**
2. Add:
   - Destination: `0.0.0.0/0`
   - Target: NAT Gateway → `natgw-1a`
3. Associate with **private-sn-1a**

#### For `aws-alb-private-rt-1b`
1. Same as above, but target → `natgw-1b`
2. Associate with **private-sn-1b**

---

### 🔒 Step 7: Create Security Groups

#### 7.1 ALB Security Group (`alb-sg`)
- **Inbound Rules:**
  - Type: HTTP, Port: 80, Source: 0.0.0.0/0
- **Outbound Rules:**
  - Allow all

#### 7.2 Private Instance Security Group (`private-sg`)
- **Inbound Rules:**
  - HTTP (80) → from `alb-sg`
  - SSH (22) → from your IP (My IP)
- **Outbound Rules:**
  - Allow all

---

### 💻 Step 8: Create Launch Template
This defines the EC2 instance configuration.

1. Go to **EC2 → Launch Templates → Create Launch Template**
2. Set:
   - **Name:** `aws-alb-template`
   - **AMI:** Amazon Linux 2
   - **Instance Type:** `t2.micro`
   - **Key Pair:** Existing or new key
   - **Security Group:** `private-sg`
3. Under **Advanced Details → User data**, add:

```bash
#!/bin/bash
sudo yum update -y
sudo yum install -y httpd
echo "<h1>Hello from $(hostname -f) in private subnet</h1>" > /var/www/html/index.html
sudo systemctl enable httpd
sudo systemctl start httpd
```

## 💻 Step 9: Launch Private EC2 Instances
1. Go to **EC2 → Launch Instance → From Template**
2. Choose:
   - **Launch Template:** `aws-alb-template`
   - **Subnet:** `private-sn-1a`
3. Launch instance.
4. Repeat for **private-sn-1b** subnet.

Now you have **two EC2 instances** in private subnets — one per Availability Zone.

---

### 🎯 Step 10: Create Target Group
1. Go to **EC2 → Target Groups → Create Target Group**
2. Type: **Instances**
3. Name: `aws-alb-tg`
4. Protocol: **HTTP | Port 80**
5. VPC: `aws-alb-vpc`
6. Register both private EC2 instances
7. Click **Create target group**

---

### ⚖️ Step 11: Create Application Load Balancer (ALB)
1. Go to **EC2 → Load Balancers → Create Load Balancer → Application Load Balancer**
2. **Name:** `aws-alb`
3. **Scheme:** Internet-facing  
4. **IP Type:** IPv4  
5. **VPC:** `aws-alb-vpc`
6. **Availability Zones:**  
   - `public-sn-1a`  
   - `public-sn-1b`
7. **Security Group:** `alb-sg`
8. **Listener:** HTTP (80) → Forward to `aws-alb-tg`
9. Click **Create Load Balancer**
10. Wait until the ALB becomes **Active**

---

### 🔁 Step 12 (Optional): Create Auto Scaling Group (ASG)
1. Go to **EC2 → Auto Scaling Groups → Create Auto Scaling Group**
2. Choose **Launch Template → aws-alb-template**
3. Select VPC: `aws-alb-vpc`
4. Choose private subnets: `private-sn-1a`, `private-sn-1b`
5. Attach to existing **Target Group:** `aws-alb-tg`
6. Set capacity:
   - Desired: `2`
   - Minimum: `1`
   - Maximum: `4`
7. Click **Create Auto Scaling Group**

This enables automatic scaling and high availability for your backend web servers.

---

### 🌍 Step 13: Test the Setup
1. Go to **EC2 → Load Balancers**
2. Copy your **ALB DNS name**, e.g.:
