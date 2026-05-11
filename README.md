# 🚀 AWS 3-Tier Application Deployment on AWS

![AWS](https://img.shields.io/badge/AWS-Cloud-orange)
![React](https://img.shields.io/badge/Frontend-React-blue)
![Node.js](https://img.shields.io/badge/Backend-Node.js-green)
![MySQL](https://img.shields.io/badge/Database-MySQL-blue)
![Nginx](https://img.shields.io/badge/WebServer-Nginx-brightgreen)

A production-style **3-tier web application deployment** project using:

- **React.js** → Frontend
- **Node.js + Express.js** → Backend API
- **MySQL RDS** → Database
- **Nginx** → Reverse Proxy
- **AWS EC2** → Cloud Infrastructure

This project demonstrates how to build and deploy a scalable, secure, and highly available 3-tier architecture in AWS using industry best practices.

---

# 📖 Project Overview

This project is designed to help beginners and freshers understand how real-world applications are deployed in AWS Cloud.

The application is divided into three layers:

| Tier | Technology | Purpose |
|------|-------------|----------|
| 🌐 Web Tier | React.js + Nginx | Frontend UI |
| ⚙️ Application Tier | Node.js + Express.js | Backend APIs & Business Logic |
| 🗄️ Database Tier | MySQL RDS | Database Storage |

---

# 🏗️ AWS 3-Tier Architecture

The below diagram represents the complete AWS infrastructure used in this project including:

- Public & Private Subnets
- Web Tier
- Application Tier
- Database Tier
- Load Balancers
- Auto Scaling Groups
- NAT Gateway
- Bastion Host
- RDS MySQL Database

![AWS Architecture](images/3-tier)

---

# 📌 Architecture Explanation

## 🌐 Web Tier
- Hosted inside public subnets
- Contains React.js application
- Nginx works as reverse proxy
- Connected to External Load Balancer

## ⚙️ Application Tier
- Hosted inside private subnets
- Runs Node.js backend APIs
- Connected through Internal Load Balancer

## 🗄️ Database Tier
- Amazon RDS MySQL deployed in private subnets
- Only accessible from Application Tier

## 🔒 Security
- Security Groups restrict traffic between tiers
- Database is not publicly accessible
- HTTPS enabled using ACM 

---

# ⚙️ Technologies Used

## Frontend
- React.js
- Axios
- HTML5
- CSS3

## Backend
- Node.js
- Express.js
- PM2

## Database
- MySQL RDS

## AWS Services
- VPC
- EC2
- RDS
- IAM
- S3
- ALB
- Auto Scaling
- Route53
- ACM

---

# 📁 Project Structure

```bash
3tier-app-deployment-aws-main/
│
├── images/
│   └── 3-tier-architecture.webp
│
├── application-code/
│
│   ├── app-tier/
│   │   ├── DbConfig.js
│   │   ├── TransactionService.js
│   │   ├── index.js
│   │   ├── package.json
│   │
│   ├── web-tier/
│   │   ├── public/
│   │   ├── src/
│   │   ├── package.json
│   │
│   ├── nginx.conf
│   └── nginx-Without-SSL.conf
│
├── install.sh
└── README.md
```

---

# 🚀 Deployment Flow

1. Create VPC & Subnets
2. Configure Security Groups
3. Create S3 Bucket
4. Create IAM Role
5. Setup RDS Database
6. Deploy Application Tier
7. Configure Internal Load Balancer
8. Deploy Web Tier
9. Configure External Load Balancer
10. Setup HTTPS using ACM
11. Configure Route53
12. Configure Auto Scaling

---

# ☁️ AWS Infrastructure Setup

---

# Step 1️⃣ Create VPC

## Why?

We create a custom VPC to isolate infrastructure securely inside AWS.

## Configuration

| Setting | Value |
|---------|------|
| CIDR Block | 192.168.0.0/16 |
| Availability Zones | 2 |
| Public Subnets | 2 |
| Private Subnets | 4 |
| NAT Gateway | 1 |

---

# Step 2️⃣ Create Security Groups

Security Groups work as virtual firewalls.

| Security Group | Allowed Traffic |
|----------------|-----------------|
| WebALB-SG | HTTP/HTTPS from Internet |
| Web-SG | Access from Web ALB |
| AppALB-SG | Access from Web Tier |
| App-SG | Port 4000 from App ALB |
| Database-SG | MySQL from App Tier |

---

# Step 3️⃣ Create S3 Bucket

Create private S3 bucket:

```bash
3-tier-project-demo
```

Upload:
- application-code/
- install.sh

---

# Step 4️⃣ Create IAM Role

Create IAM Role for EC2 instances.

## Permissions

- AmazonSSMManagedInstanceCore
- S3 Access

This allows:
- Session Manager access
- S3 access without keys

---

# Step 5️⃣ Create RDS MySQL Database

## Create DB Subnet Group

| Setting | Value |
|---------|------|
| Name | tier-Subnet-Group |
| Subnets | DB1 & DB2 |

---

## Create RDS Instance

| Setting | Value |
|---------|------|
| Engine | MySQL |
| Public Access | No |
| Security Group | Database-SG |

Update credentials inside:

```bash
application-code/app-tier/DbConfig.js
```

---

# Step 6️⃣ Setup Application Tier

Launch EC2 instance inside private subnet.

## Install Dependencies

```bash
sudo dnf install -y mariadb105
```

---

## Install Node.js & PM2

```bash
curl -o- https://raw.githubusercontent.com/Anand9340/3tier-app-deployment-aws/main/install.sh | bash

source ~/.bashrc

nvm install 16

nvm use 16

npm install -g pm2
```

---

## Download Application Code

```bash
aws s3 cp s3://3-tier-project-demo/application-code/app-tier/ app-tier --recursive
```

---

## Start Backend Application

```bash
cd app-tier

npm install

pm2 start index.js
```

---

## Verify Health Check

```bash
curl http://localhost:4000/health
```

---

# Step 7️⃣ Create Internal Load Balancer

## Target Group

| Setting | Value |
|---------|------|
| Name | App-TG |
| Port | 4000 |
| Health Check | /health |

---

## Internal ALB

| Setting | Value |
|---------|------|
| Name | app-internal-alb |
| Scheme | Internal |

Update internal ALB DNS inside:

```bash
nginx.conf
```

---

# Step 8️⃣ Setup Web Tier

Launch EC2 instance inside public subnet.

---

## Install Frontend

```bash
aws s3 cp s3://3-tier-project-demo/application-code/web-tier/ web-tier --recursive

cd web-tier

npm install

npm run build
```

---

## Install Nginx

```bash
sudo dnf install -y nginx
```

Replace default config:

```bash
sudo rm /etc/nginx/nginx.conf

sudo aws s3 cp s3://3-tier-project-demo/application-code/nginx.conf /etc/nginx/
```

Restart Nginx:

```bash
sudo systemctl restart nginx
```

---

# Step 9️⃣ Create External Load Balancer

## Web Target Group

| Setting | Value |
|---------|------|
| Name | Web-TG |
| Port | 80 |

---

## External ALB

| Setting | Value |
|---------|------|
| Name | app-external-alb |
| Scheme | Internet-facing |

---

# Step 🔟 Configure HTTPS using ACM

## Why?

HTTPS encrypts communication between users and application.

## Steps

1. Request SSL certificate in ACM
2. Validate domain
3. Attach certificate to HTTPS listener

---

# Step 1️⃣1️⃣ Configure Route53

Point domain to External Load Balancer.

Example:

```text
https://yourdomain.com
```

---

# Step 1️⃣2️⃣ Configure Auto Scaling

## Why?

Auto Scaling automatically adjusts server count based on traffic.

## Benefits

- High Availability
- Scalability
- Cost Optimization

---

# 🔐 Security Best Practices

- Use private subnets for App & Database tiers
- Restrict database access
- Enable HTTPS
- Use IAM Roles
- Configure Security Groups properly

---

# 🧪 Testing Application

## Backend Health Check

```bash
curl http://localhost:4000/health
```

---

## Open Application

```text
https://yourdomain.com
```

---

# 📚 What I Learned

- AWS Networking
- VPC Architecture
- Public vs Private Subnets
- EC2 Deployment
- Internal & External Load Balancers
- Auto Scaling Groups
- RDS Integration
- Reverse Proxy using Nginx
- Route53 & HTTPS Setup

---

# 🚀 Future Improvements

- Docker Containerization
- Kubernetes Deployment
- Terraform Automation
- CI/CD Pipeline
- CloudWatch Monitoring
- CloudFront CDN

---

