# 🚀 AWS 2-Tier Web Application Deployment

## 📝 1. Project Overview

This project demonstrates how to deploy a **secure and scalable 2-tier web application with vpc** on AWS using infrastructure-as-code with **CloudFormation**.

### 🔧 What This Project Includes:
-  A **custom VPC** split into public and private subnets
-  A **public EC2 instance** that acts as both:
  - A **web server** (running Apache Tomcat)
  - A **bastion host** to access private resources
-  A **private EC2 instance** used to test database connections securely
-  An **RDS MySQL** database in the private subnet
-  A **Java web application** hosted on Apache Tomcat, connected to RDS

### 🎯 Purpose:
- Demonstrate **network isolation** using subnets
- Ensure **secure database access** via a bastion host
- Deploy a full-stack Java app using **CI-like steps** with Maven and Tomcat
- Practice **CloudFormation templating**

---

## 🗺️ 2. Architecture Diagram

> 🧩 **You can create this using draw.io, Lucidchart, or embed an image** in your final report.
```
       [ Your Laptop ]
             |
           SSH
             ↓
  [ Public EC2 (Bastion + App) ]
             |
          SSH (Internal)
             ↓
  [ Private EC2 (DB Tester) ]
             ↓
    [ RDS MySQL in Private Subnet ]
```

It should include:

- A VPC with:
  -  **Public Subnet** (e.g., `us-east-1a`)
  -  **Private Subnet** (e.g., `us-east-1b`)
- EC2 instances:
  -  Public EC2 = App + Bastion Host
  -  Private EC2 = DB Test Client
-  Internet Gateway connected to public route table
-  NAT (optional for outbound access if needed)
- 🗄 RDS MySQL in the private subnet
---

## 🧰 3. Prerequisites

To successfully deploy this project, you need:

-  An AWS account
-  A **key pair** (`.pem`) to SSH into EC2 instances
-  Basic tools:
- Git, Java (JRE), Maven
- Apache Tomcat (manual or script-based setup)
-  Knowledge of:
- Linux shell basics
- MySQL operations
- AWS EC2, VPC, and RDS
- 💻 Terminal or SSH client (e.g., PuTTY for Windows)

---

## 🏗️ 4. VPC Setup Using CloudFormation

You provisioned networking infrastructure with a **CloudFormation YAML template** that creates:

- A VPC (`10.0.0.0/16`)
- Two subnets:
- `10.0.0.0/20` (public)
- `10.0.16.0/20` (private)
- Internet Gateway and public routing
- Route tables for subnet association

> 📄 **Tip**: Include your YAML file in your project folder or repository as `vpc-setup.yaml`.
<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/2f8c0101-245a-464b-9d75-59b619b024c6" />
<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/92bd8489-05dc-4587-a4bc-e65fff0f8a4e" />
<img width="1639" height="552" alt="Image" src="https://github.com/user-attachments/assets/07c5d10f-ef78-4f71-bb7b-0cfc77637bef" />
<img width="1210" height="327" alt="Image" src="https://github.com/user-attachments/assets/a011ecc1-3c01-44a7-97c6-13b3b678a563" />
<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/c40d4ebe-3881-4af5-b648-509db4ed17ed" />
<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/bb187736-d41e-4371-90f4-4d919bf3ddf2" />
---

## 💻 5. EC2 Instance Setup

### 🔓 Public EC2 Instance (App Server & Bastion Host):
- **AMI**: Ubuntu or Amazon Linux
- **Subnet**: Public (`10.0.0.0/20`)
- **Security Group**:
- Inbound:
  - Port 22 (SSH) — From your IP
  - Port 8080 — From anywhere (for Tomcat)
- **Purpose**:
- Hosts the **Java web app**
- Acts as **jump box** to reach private EC2

### 🔐 Private EC2 Instance (Database Tester):
- **Subnet**: Private (`10.0.16.0/20`)
- **Security Group**:
- Port 22 (SSH) — From public EC2 private IP only
- **Purpose**:
- Used to test access to RDS
- Secure — not directly exposed to the internet

<img width="1183" height="290" alt="Image" src="https://github.com/user-attachments/assets/8b69ce49-ea06-4e52-b08f-f127b90930f4" />
---

## 🔐 6. SSH Access via Jump Host

Use **SSH agent forwarding** to access the private EC2 through the public one:

# From your local machine:
```
ssh -A -i my-key.pem ec2-user@<public-ec2-public-ip>
```
# From the public EC2 instance:
```
ssh ec2-user@<private-ec2-private-ip>
```
✅ Ensure:

SSH agent forwarding is enabled

Security groups allow internal traffic

You don't expose your .pem file unnecessarily

---

## 🗄️ 7. RDS MySQL Setup

- **Engine**: MySQL  
- **Endpoint**:
- database-1.c7k4u266e1cr.us-east-1.rds.amazonaws.com

- 
### ✅ Test Connection from Private EC2

```plsql
mysql -h database-1.c7k4u266e1cr.us-east-1.rds.amazonaws.com -u admin -p
```

## 🚀 8. Web Application Deployment
### 🧱 App Build & Setup
```bash
sudo apt update
sudo apt install git -y
git clone https://github.com/shiva7919/aws-rds-java.git
cd aws-rds-java/
sudo apt install openjdk-17-jre-headless -y
sudo apt install maven -y
mvn clean package
```
<img width="945" height="323" alt="Image" src="https://github.com/user-attachments/assets/63948a7f-f6ed-41dc-9835-dfed8aa73fde" />

## 🌐 Apache Tomcat Setup
```bash
wget https://downloads.apache.org/tomcat/tomcat-9/v9.0.107/bin/apache-tomcat-9.0.107.zip
sudo apt install unzip -y
unzip apache-tomcat-9.0.107.zip
cp target/*.war apache-tomcat-9.0.107/webapps/
cd apache-tomcat-9.0.107/bin
chmod 755 *.sh
sh startup.sh
```
<img width="1581" height="715" alt="Image" src="https://github.com/user-attachments/assets/148900aa-5364-41e3-a18d-f030c2310f8e" />
---
### ✅ After starting Tomcat, confirm that port 8080 is open to the internet in your Security Group.

## 🧪 9. Application Testing
🔗 Visit the Application in Browser
```pgsql
http://<public-ec2-public-ip>:8080/aws-rds-java
```

### 📋 Test Pages
-login.jsp
-userRegistration.jsp
<img width="1064" height="451" alt="Image" src="https://github.com/user-attachments/assets/72565bc1-0922-45c5-84b0-a269c65cbaeb" />
💡 These pages interact with the RDS database using JDBC. Ensure DB credentials are set correctly in the app.

## 🔐 10. Security Group Rules Summary
### 🔧 Resource	🔢 Port	🌍 Source	🎯 Purpose
-Public EC2	22	Your IP	SSH Access
-Public EC2	8080	0.0.0.0/0	Web App Access (Tomcat)
-Private EC2	22	Public EC2 Private IP	Jump/Bastion Access
-RDS MySQL	3306	Private EC2 Private IP	MySQL Connectivity
<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/63db55a5-b079-46b5-8f53-a3b5b22015df" />

### 🔒 Always follow least privilege principles when configuring security groups.

## 📁 11. Final Directory Tree
-From the root of your public EC2 instance where the app is cloned, run:
```
aws-rds-java/
├── pom.xml
├── target/
│   └── aws-rds-java.war
└── apache-tomcat-9.0.107/
    └── webapps/
        └── aws-rds-java.war
```
## 🏁 12. Conclusion
In this project, you successfully:
<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/eed5b3f2-1522-44f2-a758-308794019e80" />
<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/674bc31a-0cbe-41bc-8f72-85c1bf6a558e" />
<img width="829" height="534" alt="Image" src="https://github.com/user-attachments/assets/9e2b9a30-ddc3-4be5-9da3-5911cb7207cc" />
<img width="1103" height="663" alt="Image" src="https://github.com/user-attachments/assets/8b6cec7f-ac52-43f8-995e-149ff83fbbe7" />

-✅ Created a secure 2-tier architecture using AWS CloudFormation
-🛠️ Deployed EC2 instances in public and private subnets
-🔐 Configured a jump host for accessing private resources
-🌐 Deployed a Java-based login application on Apache Tomcat
-🔗 Connected the application securely to an RDS MySQL instance
