# AWS VPC with ALB, NAT, and Private Instances

This project creates a **highly available AWS infrastructure** across two Availability Zones using the AWS Management Console.

## 🏗️ Architecture

- **VPC:** 10.0.0.0/16  
- **Subnets:** 2 public + 2 private (across us-east-1a and us-east-1b)  
- **Internet Gateway** for ALB access  
- **NAT Gateways** in each public subnet for private instances  
- **Application Load Balancer (ALB)** routing HTTP traffic  
- **Private EC2 instances** running Apache web server  
- **Target Group** to forward requests to instances  
- **Security Groups** controlling access  

## 🧩 Components

| Component | Description |
|------------|--------------|
| **VPC** | Custom VPC (10.0.0.0/16) |
| **Public Subnets** | Host ALB and NAT gateways |
| **Private Subnets** | Host EC2 instances |
| **IGW** | Internet access for ALB |
| **NAT GW** | Outbound internet access for private instances |
| **ALB** | Distributes traffic to target group |
| **Target Group** | Links ALB to EC2 instances |
| **EC2** | Apac
