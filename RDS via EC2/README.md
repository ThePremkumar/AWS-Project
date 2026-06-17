![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Amazon RDS](https://img.shields.io/badge/Amazon%20RDS-527FFF?style=for-the-badge&logo=amazonrds&logoColor=white)
![Amazon EC2](https://img.shields.io/badge/Amazon%20EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![Security Groups](https://img.shields.io/badge/Security%20Groups-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white)
![License: MIT](https://img.shields.io/badge/License-MIT-22c55e?style=for-the-badge)
![Status: Active](https://img.shields.io/badge/Status-Active-brightgreen?style=for-the-badge)
![Made with ❤️ in India](https://img.shields.io/badge/Made%20with%20%E2%9D%A4%EF%B8%8F%20in-India-blue?style=for-the-badge)

---

# 🗄️ RDS via EC2: Connecting Amazon RDS (MySQL) Through an EC2 Instance

> **A hands-on AWS project demonstrating how to provision, connect, and manage an Amazon RDS MySQL database through an EC2 instance — including multi-database management, automated backups, and read replicas.**

---

## 📑 Table of Contents

- [Overview](#-overview)
- [Architecture Diagram](#-architecture-diagram)
- [AWS Services Used](#-aws-services-used)
- [Key Features](#-key-features)
- [Prerequisites](#-prerequisites)
- [Project Structure](#-project-structure)
- [Setup & Deployment](#-setup--deployment)
- [How It Works](#-how-it-works)
- [Database Schemas Demonstrated](#-database-schemas-demonstrated)
- [Security Highlights](#-security-highlights)
- [Testing & Validation](#-testing--validation)
- [Screenshots](#-screenshots)
- [RDS Backup & Snapshots](#-rds-backup--snapshots)
- [Read Replica](#-read-replica)
- [Common Issues & Troubleshooting](#-common-issues--troubleshooting)
- [Cleanup / Destroy](#-cleanup--destroy)
- [Future Improvements](#-future-improvements)
- [Contributing](#-contributing)
- [License](#-license)
- [Author & Contact](#-author--contact)

---

## 📌 Overview

### What This Project Does

This project provisions an **Amazon RDS MySQL instance** inside a private subnet and establishes a secure connection from an **EC2 instance** acting as the client/application layer. The EC2 instance connects to the RDS endpoint using the `mysql` CLI client.

Once connected, five real-world database schemas are created and queried to validate full database functionality:

| Database | Domain |
|---|---|
| Student Management System | Education |
| Employee Management System | HR / Enterprise |
| Library Management System | Operations |
| Hospital Management System | Healthcare |
| Online Shopping Database | E-Commerce |

The project also validates two critical RDS production features:
- **Automated Backup & Snapshots** — Manual snapshot creation and verification.
- **Read Replica** — Provisioning a read replica from the primary RDS instance to demonstrate horizontal read scaling.

### Why It Was Built / Real-World Use Case

In production AWS environments, databases are **never exposed to the public internet**. Amazon RDS is deployed inside private subnets, and application servers (EC2, ECS, Lambda) connect to it via private DNS endpoints. This project simulates and validates this exact pattern:

- **Application–Database Separation**: The EC2 instance acts as the application server; the RDS instance acts as the managed database tier.
- **Managed Database Service**: Instead of self-managing MySQL on EC2, Amazon RDS handles automated patching, backups, and failover.
- **Security Group Chaining**: The RDS security group permits port `3306` exclusively from the EC2 security group — not from any public IP.

### Key Problem It Solves

Exposing MySQL directly on a public IP is a critical security vulnerability. This project enforces the principle that:
> **EC2 → RDS (via SG reference)** is the standard, secure AWS database connectivity pattern — replacing the insecure anti-pattern of **Internet → MySQL:3306**.

---

## 🏗️ Architecture Diagram

```
        ┌──────────────────────────────────────────────────────────────────┐
        │                 DEVELOPER WORKSTATION                           │
        │           SSH Client → EC2 Public IP                            │
        └─────────────────────────┬────────────────────────────────────────┘
                                  │
                                  │ SSH Port 22
                                  ▼
 ┌────────────────────────────────────────────────────────────────────────────┐
 │  AWS VPC                                                                   │
 │                                                                            │
 │  ┌──────────────────────────────────────────────────────────────────────┐  │
 │  │  Public Subnet                                                       │  │
 │  │                                                                      │  │
 │  │  ┌─────────────────────────────────────────────────────────────────┐ │  │
 │  │  │  EC2 Instance (Application / MySQL Client)                      │ │  │
 │  │  │  Security Group: EC2-SG                                         │ │  │
 │  │  │  Inbound: Port 22 from Your IP                                  │ │  │
 │  │  └──────────────────────────┬──────────────────────────────────────┘ │  │
 │  └─────────────────────────────┼────────────────────────────────────────┘  │
 │                                │                                           │
 │                                │ MySQL Port 3306                           │
 │                                ▼ (Source: EC2-SG)                          │
 │  ┌──────────────────────────────────────────────────────────────────────┐  │
 │  │  Private Subnet                                                      │  │
 │  │                                                                      │  │
 │  │  ┌─────────────────────────────────────────────────────────────────┐ │  │
 │  │  │  Amazon RDS (MySQL 8.x)                                         │ │  │
 │  │  │  Security Group: RDS-SG                                         │ │  │
 │  │  │  Inbound: Port 3306 from EC2-SG only                            │ │  │
 │  │  │                                                                 │ │  │
 │  │  │  ┌────────────────────┐    ┌─────────────────────────────────┐  │ │  │
 │  │  │  │  Primary RDS (R/W) │───▶│  Read Replica RDS (Read-Only)   │  │ │  │
 │  │  │  └────────────────────┘    └─────────────────────────────────┘  │ │  │
 │  │  └─────────────────────────────────────────────────────────────────┘ │  │
 │  └──────────────────────────────────────────────────────────────────────┘  │
 └────────────────────────────────────────────────────────────────────────────┘
```

### Traffic Flow

1. **Developer → EC2**: SSH into the EC2 instance using your `.pem` key pair over port `22`.
2. **EC2 → RDS**: The EC2 instance connects to the RDS MySQL endpoint using the `mysql` CLI on port `3306`. The RDS Security Group allows this because the source is the EC2 Security Group.
3. **RDS → Read Replica**: The primary RDS instance asynchronously replicates writes to the Read Replica, which handles read-only workloads.

---

## ☁️ AWS Services Used

| Service | Purpose |
|---|---|
| **Amazon EC2** | Application-layer client; used as a MySQL CLI host |
| **Amazon RDS (MySQL)** | Fully managed relational database engine |
| **Amazon VPC** | Logical networking boundary (public + private subnets) |
| **Security Groups** | Stateful firewall — port `3306` allowed only from EC2-SG |
| **RDS Automated Backups** | Point-in-time recovery and manual snapshot support |
| **RDS Read Replica** | Horizontal read scaling from the primary instance |
| **Internet Gateway** | Provides internet access to the public subnet (for EC2 SSH) |

---

## ✨ Key Features

- 🔗 **EC2-to-RDS Connectivity**: Validated end-to-end MySQL connection via EC2 using the RDS endpoint DNS name.
- 🗃️ **Multi-Database Management**: Five real-world database schemas created and queried on the same RDS instance.
- 🔒 **Security Group Chaining**: RDS Security Group references the EC2 Security Group as source — no public IP allowed.
- 📸 **Manual Snapshot**: RDS backup snapshot created and verified in the AWS console.
- 📖 **Read Replica Provisioning**: A read replica is created from the primary instance to demonstrate AWS's horizontal scaling capability.
- 🛡️ **Private Subnet Isolation**: The RDS instance has no public IP and cannot be reached from the internet.

---

## 🛠️ Prerequisites

| Requirement | Details |
|---|---|
| **AWS Account** | Standard / Free Tier |
| **EC2 Key Pair (.pem)** | Required for SSH access to the EC2 instance |
| **MySQL Client** | Installed on the EC2 instance (`sudo dnf install mysql -y`) |
| **AWS CLI** | v2.x (optional — for CLI-based provisioning) |

---

## 📂 Project Structure

```
AWS-Project/
└── RDS via EC2/
    ├── 01 RDS_Dashboard.png                      # RDS instance overview in AWS Console
    ├── 02 EC2_dashboard.png                      # EC2 instance running as client
    ├── 03 connection.png                         # Successful mysql CLI connection to RDS endpoint
    ├── 04 Student Management System output.png   # CREATE TABLE + INSERT + SELECT demo
    ├── 05 Employee Management System output.png  # HR schema demo
    ├── 06 Library Management System output.png   # Library schema demo
    ├── 07 Hospital Management System output.png  # Healthcare schema demo
    ├── 08 Online Shopping Database output.png    # E-Commerce schema demo
    ├── 09 RDS Backup Snapshot.png                # Manual snapshot creation
    ├── 10 RDS Backup Snapshot list.png           # Snapshot list view
    ├── 11 Read Replica Demo.png                  # Read replica provisioned and active
    └── README.md                                 # This file
```

---

## 🚀 Setup & Deployment

### Step 1: Create Security Groups

```bash
# Create EC2 Security Group
aws ec2 create-security-group \
  --group-name "EC2-SG" \
  --description "EC2 Application Instance Security Group" \
  --vpc-id "<YOUR-VPC-ID>"

# Allow SSH from your IP
aws ec2 authorize-security-group-ingress \
  --group-id "<EC2-SG-ID>" \
  --protocol tcp \
  --port 22 \
  --cidr "<YOUR-IP>/32"

# Create RDS Security Group
aws ec2 create-security-group \
  --group-name "RDS-SG" \
  --description "RDS MySQL Security Group" \
  --vpc-id "<YOUR-VPC-ID>"

# Allow MySQL (3306) ONLY from EC2-SG
aws ec2 authorize-security-group-ingress \
  --group-id "<RDS-SG-ID>" \
  --protocol tcp \
  --port 3306 \
  --source-group "<EC2-SG-ID>"
```

### Step 2: Create an RDS Subnet Group

The RDS instance requires a **DB Subnet Group** spanning at least two Availability Zones.

```bash
aws rds create-db-subnet-group \
  --db-subnet-group-name "rds-private-subnet-group" \
  --db-subnet-group-description "Private subnets for RDS" \
  --subnet-ids "<PRIVATE-SUBNET-AZ-A>" "<PRIVATE-SUBNET-AZ-B>"
```

### Step 3: Launch the RDS MySQL Instance

```bash
aws rds create-db-instance \
  --db-instance-identifier "prem-rds-mysql" \
  --db-instance-class db.t3.micro \
  --engine mysql \
  --master-username admin \
  --master-user-password "<YOUR-PASSWORD>" \
  --allocated-storage 20 \
  --vpc-security-group-ids "<RDS-SG-ID>" \
  --db-subnet-group-name "rds-private-subnet-group" \
  --no-publicly-accessible \
  --backup-retention-period 7 \
  --region us-east-1
```

> ⏳ RDS provisioning takes approximately 5–10 minutes. Wait for the instance status to change to `available`.

### Step 4: Launch the EC2 Instance

```bash
aws ec2 run-instances \
  --image-id ami-04b70fa74e45c3917 \
  --count 1 \
  --instance-type t3.micro \
  --key-name "aws-new" \
  --security-group-ids "<EC2-SG-ID>" \
  --subnet-id "<PUBLIC-SUBNET-ID>" \
  --associate-public-ip-address \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=RDS-Client-EC2}]' \
  --region us-east-1
```

### Step 5: Install MySQL Client on EC2

```bash
# SSH into EC2
ssh -i "aws-new.pem" ec2-user@<EC2-PUBLIC-IP>

# Install MySQL client
sudo dnf install mysql -y
```

### Step 6: Connect to RDS from EC2

```bash
mysql -h <RDS-ENDPOINT> -u admin -p
# Enter your password when prompted
```

*Verify the connection:*
```sql
SHOW DATABASES;
SELECT VERSION();
```

---

## ⚙️ How It Works

### Security Group Reference (Port 3306)

The RDS Security Group (`RDS-SG`) is configured with a **Security Group Reference** rule — the source is the EC2 Security Group ID (`EC2-SG`), not any IP range. This means:

- ✅ Any EC2 instance associated with `EC2-SG` can connect to MySQL on port `3306`.
- ❌ Direct connections from the internet, your laptop, or any other source are dropped silently.
- 🔄 If the EC2 instance is replaced (e.g., by Auto Scaling), the new instance inherits `EC2-SG` and automatically has access — no rule changes needed.

### RDS Endpoint DNS Resolution

Amazon RDS exposes a **DNS endpoint** (e.g., `prem-rds-mysql.xxxxxxxx.us-east-1.rds.amazonaws.com`) that resolves to the private IP of the RDS instance. The EC2 instance resolves this via the VPC's internal DNS resolver (Route 53 Resolver). No public IP is ever involved.

---

## 🗃️ Database Schemas Demonstrated

All five databases were created and tested on the same RDS instance to demonstrate multi-tenant database management.

### 1. 🎓 Student Management System
```sql
CREATE DATABASE StudentDB;
USE StudentDB;
CREATE TABLE students (
    student_id INT PRIMARY KEY AUTO_INCREMENT,
    name       VARCHAR(100) NOT NULL,
    email      VARCHAR(100) UNIQUE,
    course     VARCHAR(50),
    enrolled   DATE
);
INSERT INTO students (name, email, course, enrolled)
VALUES ('Prem Kumar', 'prem@example.com', 'Cloud Computing', '2024-01-15');
SELECT * FROM students;
```

### 2. 👔 Employee Management System
```sql
CREATE DATABASE EmployeeDB;
USE EmployeeDB;
CREATE TABLE employees (
    emp_id     INT PRIMARY KEY AUTO_INCREMENT,
    name       VARCHAR(100) NOT NULL,
    department VARCHAR(50),
    salary     DECIMAL(10,2),
    join_date  DATE
);
INSERT INTO employees (name, department, salary, join_date)
VALUES ('Prem Kumar', 'DevOps', 75000.00, '2024-03-01');
SELECT * FROM employees;
```

### 3. 📚 Library Management System
```sql
CREATE DATABASE LibraryDB;
USE LibraryDB;
CREATE TABLE books (
    book_id   INT PRIMARY KEY AUTO_INCREMENT,
    title     VARCHAR(200) NOT NULL,
    author    VARCHAR(100),
    isbn      VARCHAR(20) UNIQUE,
    available BOOLEAN DEFAULT TRUE
);
INSERT INTO books (title, author, isbn)
VALUES ('AWS Solutions Architect Guide', 'Jon Bonso', '978-1-XXXXXX');
SELECT * FROM books;
```

### 4. 🏥 Hospital Management System
```sql
CREATE DATABASE HospitalDB;
USE HospitalDB;
CREATE TABLE patients (
    patient_id INT PRIMARY KEY AUTO_INCREMENT,
    name       VARCHAR(100) NOT NULL,
    age        INT,
    diagnosis  VARCHAR(200),
    admitted   DATE
);
INSERT INTO patients (name, age, diagnosis, admitted)
VALUES ('Test Patient', 35, 'Routine Checkup', '2024-06-01');
SELECT * FROM patients;
```

### 5. 🛒 Online Shopping Database
```sql
CREATE DATABASE ShoppingDB;
USE ShoppingDB;
CREATE TABLE products (
    product_id  INT PRIMARY KEY AUTO_INCREMENT,
    name        VARCHAR(200) NOT NULL,
    price       DECIMAL(10,2),
    stock       INT DEFAULT 0,
    category    VARCHAR(50)
);
INSERT INTO products (name, price, stock, category)
VALUES ('AWS T-Shirt', 19.99, 100, 'Merchandise');
SELECT * FROM products;
```

---

## 🛡️ Security Highlights

### EC2 Security Group Rules

| Type | Protocol | Port | Source | Action |
|---|---|---|---|---|
| SSH | TCP | `22` | `Your IP /32` | ✅ ALLOW |
| All other | All | All | `0.0.0.0/0` | ❌ DENY (implicit) |

### RDS Security Group Rules

| Type | Protocol | Port | Source | Action | Rationale |
|---|---|---|---|---|---|
| MySQL/Aurora | TCP | `3306` | `EC2-SG (Group ID)` | ✅ ALLOW | Only EC2 instances in EC2-SG can connect |
| All other | All | All | `0.0.0.0/0` | ❌ DENY (implicit) | No public internet access to MySQL |

> 🔐 **Critical**: The RDS instance has `Publicly Accessible` set to **No**. Even if the security group allowed public IPs, there is no public route to the RDS endpoint.

---

## 🧪 Testing & Validation

### Test 1: Verify EC2-to-RDS Connectivity

```bash
# From EC2 instance
mysql -h <RDS-ENDPOINT> -u admin -p
```
**Expected result**: Successful login and `mysql>` prompt.

### Test 2: Verify Database Isolation

```sql
-- List all databases
SHOW DATABASES;
-- Expected: StudentDB, EmployeeDB, LibraryDB, HospitalDB, ShoppingDB, information_schema, mysql, performance_schema
```

### Test 3: Verify RDS is NOT Publicly Accessible

```bash
# From your local machine (should FAIL)
mysql -h <RDS-ENDPOINT> -u admin -p
# Expected result: Connection timeout — proving the RDS is private-only
```

### Test 4: Verify Read Replica

```bash
# Connect to Read Replica endpoint
mysql -h <READ-REPLICA-ENDPOINT> -u admin -p

-- Verify it is read-only
INSERT INTO StudentDB.students (name) VALUES ('Test');
-- Expected: ERROR 1290 (HY000): The MySQL server is running with the --read-only option
```

---

## 📸 Screenshots

### 1️⃣ RDS Dashboard
> Amazon RDS console showing the primary MySQL instance in `Available` state with endpoint, engine version, instance class, and multi-AZ configuration.

![RDS Dashboard](01%20RDS_Dashboard.png)

---

### 2️⃣ EC2 Dashboard
> AWS EC2 console showing the client EC2 instance in `Running` state with its public IPv4 address used for SSH access.

![EC2 Dashboard](02%20EC2_dashboard.png)

---

### 3️⃣ Successful RDS Connection from EC2
> Terminal output from the EC2 instance showing a successful `mysql -h <RDS-ENDPOINT>` connection, confirming the Security Group and network configuration is correct.

![RDS Connection from EC2](03%20connection.png)

---

### 4️⃣ Student Management System Output
> MySQL query output showing `CREATE DATABASE`, `CREATE TABLE`, `INSERT`, and `SELECT` results for the Student Management System.

![Student Management System](04%20Student%20Management%20System%20output.png)

---

### 5️⃣ Employee Management System Output
> MySQL query output for the Employee Management System demonstrating HR schema creation and data insertion.

![Employee Management System](05%20Employee%20Management%20System%20output.png)

---

### 6️⃣ Library Management System Output
> MySQL query output for the Library Management System showing book records management.

![Library Management System](06%20Library%20Management%20System%20output.png)

---

### 7️⃣ Hospital Management System Output
> MySQL query output for the Hospital Management System showing patient record management.

![Hospital Management System](07%20Hospital%20Management%20System%20output.png)

---

### 8️⃣ Online Shopping Database Output
> MySQL query output for the Online Shopping Database showing product catalog management.

![Online Shopping Database](08%20Online%20Shopping%20Database%20output.png)

---

## 📸 RDS Backup & Snapshots

Amazon RDS provides **automated daily backups** and allows **manual on-demand snapshots** that can be used for point-in-time recovery or cross-region restore.

### Creating a Manual Snapshot (Console)
1. Navigate to **RDS → Databases → Select your instance**.
2. Click **Actions → Take snapshot**.
3. Enter a snapshot name (e.g., `prem-rds-manual-snapshot-01`) and click **Take Snapshot**.

### Creating a Manual Snapshot (CLI)
```bash
aws rds create-db-snapshot \
  --db-instance-identifier "prem-rds-mysql" \
  --db-snapshot-identifier "prem-rds-manual-snapshot-01" \
  --region us-east-1
```

### 9️⃣ RDS Backup Snapshot Creation
> AWS Console showing the manual snapshot being created with status `Creating`.

![RDS Backup Snapshot](09%20RDS%20Backup%20Snapshot.png)

---

### 🔟 RDS Backup Snapshot List
> AWS Console showing the completed manual snapshot in `Available` status, ready for restore.

![RDS Backup Snapshot List](10%20RDS%20Backup%20Snapshot%20list.png)

---

## 📖 Read Replica

A **Read Replica** is an asynchronous copy of the primary RDS instance used to offload read-heavy workloads (reports, analytics, dashboards) without impacting write performance on the primary database.

### Key Facts
- Read Replicas use **asynchronous replication** — there may be a small replication lag.
- They have a **separate DNS endpoint** — applications must explicitly connect to the replica for reads.
- Read Replicas can be **promoted** to standalone databases if the primary fails (manual failover).
- Multi-AZ is for **high availability (failover)**; Read Replicas are for **read scalability** — these are different features.

### Creating a Read Replica (Console)
1. Navigate to **RDS → Databases → Select your instance**.
2. Click **Actions → Create read replica**.
3. Choose the destination region and instance class, then click **Create read replica**.

### Creating a Read Replica (CLI)
```bash
aws rds create-db-instance-read-replica \
  --db-instance-identifier "prem-rds-read-replica" \
  --source-db-instance-identifier "prem-rds-mysql" \
  --db-instance-class db.t3.micro \
  --region us-east-1
```

### 1️⃣1️⃣ Read Replica Demo
> AWS Console showing the Read Replica instance in `Available` state alongside the primary instance, with its own unique endpoint.

![Read Replica Demo](11%20Read%20Replica%20Demo.png)

---

## 🐛 Common Issues & Troubleshooting

| Issue | Cause | Fix |
|---|---|---|
| `ERROR 2003: Can't connect to MySQL server` | EC2 Security Group not referenced in RDS-SG, or wrong RDS endpoint | Verify the RDS Security Group inbound rule references the EC2-SG ID. Confirm you are using the RDS endpoint, not an IP address. |
| Connection hangs / times out | RDS is in a private subnet with no route from EC2 subnet | Ensure both EC2 and RDS are in the same VPC. Check route tables. |
| `Access denied for user 'admin'@'...'` | Incorrect password or username | Double-check the master username and password set during RDS creation. |
| Read Replica shows replication lag | High write throughput on primary | This is expected under heavy write load. Monitor `ReplicaLag` in CloudWatch. |
| `ERROR 1290: The MySQL server is running with --read-only` | You are connected to the Read Replica endpoint | Read Replicas do not accept writes. Connect to the primary endpoint for write operations. |
| RDS snapshot takes too long | Large database size | Snapshot time scales with storage size. Monitor snapshot status in the console. |

---

## 🧹 Cleanup / Destroy

> ⚠️ **Billing Warning**: RDS instances and Read Replicas incur ongoing hourly costs even when idle. Follow these steps to avoid unexpected charges.

### Step 1: Delete the Read Replica
```bash
aws rds delete-db-instance \
  --db-instance-identifier "prem-rds-read-replica" \
  --skip-final-snapshot \
  --region us-east-1
```

### Step 2: Delete the Primary RDS Instance
```bash
aws rds delete-db-instance \
  --db-instance-identifier "prem-rds-mysql" \
  --skip-final-snapshot \
  --region us-east-1
```
> *Wait for the instance to reach `deleted` status before proceeding.*

### Step 3: Terminate the EC2 Instance
```bash
aws ec2 terminate-instances \
  --instance-ids "<EC2-INSTANCE-ID>" \
  --region us-east-1
```

### Step 4: Delete Security Groups
```bash
# Delete RDS Security Group
aws ec2 delete-security-group \
  --group-id "<RDS-SG-ID>" \
  --region us-east-1

# Delete EC2 Security Group
aws ec2 delete-security-group \
  --group-id "<EC2-SG-ID>" \
  --region us-east-1
```

### Step 5: Delete DB Subnet Group
```bash
aws rds delete-db-subnet-group \
  --db-subnet-group-name "rds-private-subnet-group"
```

---

## 🔮 Future Improvements

1. **Multi-AZ Deployment**: Enable Multi-AZ on the primary RDS instance for automatic standby failover within 60–120 seconds.
2. **RDS Proxy**: Add an RDS Proxy in front of the database to pool and manage connections from Lambda functions or containerized applications.
3. **Secrets Manager Integration**: Store the RDS master password in AWS Secrets Manager and rotate it automatically on a schedule.
4. **IAM Database Authentication**: Replace static password authentication with IAM token-based authentication for improved security.
5. **CloudWatch Alarms**: Set up CloudWatch alarms for `CPUUtilization`, `FreeStorageSpace`, and `DatabaseConnections` to proactively monitor the RDS instance.
6. **Parameter Groups**: Create a custom RDS Parameter Group to tune MySQL settings (`innodb_buffer_pool_size`, `max_connections`, etc.) for production workloads.

---

## 🤝 Contributing

Contributions to improve this architecture documentation or automate deployments are welcome.

1. **Fork the Repository**: Create a personal copy of the repository on GitHub.
2. **Create a Feature Branch**:
   ```bash
   git checkout -b feat/rds-iam-auth
   ```
3. **Commit Your Changes**: Follow the conventional commit format:
   ```bash
   git commit -m "feat(rds): add IAM database authentication guide"
   ```
4. **Push & Open a Pull Request**: Push to your branch and open a PR targeting the main branch.

---

## 📄 License

MIT License

Copyright (c) 2026 Prem Kumar S

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

---

## 👤 Author & Contact

<br/>

| Profile Info | Details |
|---|---|
| **Name** | Prem Kumar S |
| **Role** | DevOps Engineer |
| **Location** | Krishnagiri, Tamil Nadu, India 🇮🇳 |
| **GitHub** | [github.com/ThePremkumar](https://github.com/ThePremkumar) |
| **Portfolio** | [thepremkumar.netlify.app](https://thepremkumar.netlify.app) |

<br/>

---

<div align="center">

### ⭐ Star this repo if it helped you! ⭐

*If this project helped you understand Amazon RDS, EC2-to-RDS connectivity, database backups, or read replicas — a star supports open-source cloud documentation.*

<br/>

![AWS](https://img.shields.io/badge/Built%20on-AWS-%23FF9900?style=flat-square&logo=amazon-aws&logoColor=white)
![RDS](https://img.shields.io/badge/Feature-Amazon%20RDS-527FFF?style=flat-square&logo=amazonrds&logoColor=white)
![MySQL](https://img.shields.io/badge/Engine-MySQL-4479A1?style=flat-square&logo=mysql&logoColor=white)
![Made in India](https://img.shields.io/badge/Made%20with%20%E2%9D%A4%EF%B8%8F%20in-India-blue?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-22c55e?style=flat-square)

*© 2026 Prem Kumar S*

</div>
