![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Amazon RDS](https://img.shields.io/badge/Amazon%20RDS-527FFF?style=for-the-badge&logo=amazonrds&logoColor=white)
![Amazon EC2](https://img.shields.io/badge/Amazon%20EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![CloudWatch](https://img.shields.io/badge/CloudWatch-FF4F8B?style=for-the-badge&logo=amazon-aws&logoColor=white)
![License: MIT](https://img.shields.io/badge/License-MIT-22c55e?style=for-the-badge)
![Status: Active](https://img.shields.io/badge/Status-Active-brightgreen?style=for-the-badge)
![Made with ❤️ in India](https://img.shields.io/badge/Made%20with%20%E2%9D%A4%EF%B8%8F%20in-India-blue?style=for-the-badge)

---

# 🗄️ RDS via EC2 — Full Project Suite

> **A comprehensive AWS hands-on project demonstrating Amazon RDS MySQL management through EC2 — covering multi-database schemas, automated backups & snapshots, Multi-AZ high availability, read replicas, and CloudWatch monitoring.**

---

## 📑 Table of Contents

- [Overview](#-overview)
- [Architecture Diagram](#-architecture-diagram)
- [AWS Services Used](#-aws-services-used)
- [Key Features](#-key-features)
- [Prerequisites](#-prerequisites)
- [Project Structure](#-project-structure)
- [Setup & Deployment](#-setup--deployment)
- [Project 1 — Student Management System](#-project-1--student-management-system)
- [Project 2 — Employee Management System](#-project-2--employee-management-system)
- [Project 3 — Library Management System](#-project-3--library-management-system)
- [Project 4 — Hospital Management System](#-project-4--hospital-management-system)
- [Project 5 — Online Shopping Database](#-project-5--online-shopping-database)
- [Project 6 — RDS Backup & Snapshot](#-project-6--rds-backup--snapshot)
- [Project 7 — Multi-AZ High Availability Demo](#-project-7--multi-az-high-availability-demo)
- [Project 8 — Read Replica Demo](#-project-8--read-replica-demo)
- [Project 9 — Connect EC2 to RDS](#-project-9--connect-ec2-to-rds)
- [Project 10 — RDS Monitoring with CloudWatch](#-project-10--rds-monitoring-with-cloudwatch)
- [Security Highlights](#-security-highlights)
- [Screenshots](#-screenshots)
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

Once connected, **five real-world database schemas** are created and queried to validate full database functionality. The project additionally covers four critical AWS production patterns:

| # | Database / Feature | Domain |
|---|---|---|
| 1 | Student Management System | Education |
| 2 | Employee Management System | HR / Enterprise |
| 3 | Library Management System | Operations |
| 4 | Hospital Management System | Healthcare |
| 5 | Online Shopping Database | E-Commerce |
| 6 | RDS Backup & Snapshots | Disaster Recovery |
| 7 | Multi-AZ High Availability | Fault Tolerance |
| 8 | Read Replica | Horizontal Read Scaling |
| 9 | EC2 → RDS Connectivity | Secure Networking |
| 10 | CloudWatch Monitoring & Alarms | Observability |

### Why It Was Built / Real-World Use Case

In production AWS environments, databases are **never exposed to the public internet**. Amazon RDS is deployed inside private subnets, and application servers (EC2, ECS, Lambda) connect to it via private DNS endpoints. This project simulates and validates this exact pattern:

- **Application–Database Separation**: The EC2 instance acts as the application server; the RDS instance acts as the managed database tier.
- **Managed Database Service**: Instead of self-managing MySQL on EC2, Amazon RDS handles automated patching, backups, and failover.
- **Security Group Chaining**: The RDS security group permits port `3306` exclusively from the EC2 security group — not from any public IP.

### Key Problem It Solves

> *"EC2 → RDS (via SG reference)"* is the standard, secure AWS database connectivity pattern — replacing the insecure anti-pattern of *"Internet → MySQL:3306"*.

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
 │  │  │  EC2 Instance (Amazon Linux — MySQL Client)                     │ │  │
 │  │  │  Security Group: EC2-SG                                         │ │  │
 │  │  │  Inbound: Port 22 from Your IP                                  │ │  │
 │  │  └──────────────────────────┬──────────────────────────────────────┘ │  │
 │  └─────────────────────────────┼────────────────────────────────────────┘  │
 │                                │                                           │
 │                                │ MySQL Port 3306 (Source: EC2-SG)          │
 │                                ▼                                           │
 │  ┌──────────────────────────────────────────────────────────────────────┐  │
 │  │  Private Subnet                                                      │  │
 │  │                                                                      │  │
 │  │  ┌─────────────────────────────────────────────────────────────────┐ │  │
 │  │  │  Amazon RDS MySQL (Primary — Read/Write)                        │ │  │
 │  │  │  Security Group: RDS-SG                                         │ │  │
 │  │  │  Inbound: Port 3306 from EC2-SG only                            │ │  │
 │  │  │                                                                 │ │  │
 │  │  │  ┌────────────────────┐    ┌──────────────────────────────────┐  │ │  │
 │  │  │  │  Primary RDS (R/W) │───▶│  Read Replica RDS (Read-Only)    │  │ │  │
 │  │  │  └────────────────────┘    └──────────────────────────────────┘  │ │  │
 │  │  └─────────────────────────────────────────────────────────────────┘ │  │
 │  └──────────────────────────────────────────────────────────────────────┘  │
 │                                                                            │
 │  ┌──────────────────────────────────────────────────────────────────────┐  │
 │  │  CloudWatch — Metrics, Alarms & SNS Notifications                   │  │
 │  └──────────────────────────────────────────────────────────────────────┘  │
 └────────────────────────────────────────────────────────────────────────────┘
```

### Traffic Flow

1. **Developer → EC2**: SSH into the EC2 instance using your `.pem` key pair over port `22`.
2. **EC2 → RDS**: The EC2 instance connects to the RDS MySQL endpoint using the `mysql` CLI on port `3306`. The RDS Security Group allows this because the source is the EC2 Security Group.
3. **RDS → Read Replica**: The primary RDS instance asynchronously replicates writes to the Read Replica, which handles read-only workloads.
4. **RDS → CloudWatch**: Metrics (CPU, storage, connections, IOPS) are streamed to CloudWatch for monitoring and alerting.

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
| **RDS Multi-AZ** | Synchronous standby replica for automatic failover |
| **Amazon CloudWatch** | Metrics monitoring, alarms, and SNS email alerts |
| **Amazon SNS** | Email notification delivery for CloudWatch alarms |
| **Internet Gateway** | Provides internet access to the public subnet (for EC2 SSH) |

---

## ✨ Key Features

- 🔗 **EC2-to-RDS Connectivity**: Validated end-to-end MySQL connection via EC2 using the RDS endpoint DNS name.
- 🗃️ **Multi-Database Management**: Five real-world database schemas created and queried on the same RDS instance.
- 🔒 **Security Group Chaining**: RDS Security Group references the EC2 Security Group as source — no public IP allowed.
- 📸 **Manual Snapshot & Restore**: RDS backup snapshot created, verified, and restored.
- 🔄 **Multi-AZ Deployment**: Synchronous standby enabled with reboot-with-failover validation.
- 📖 **Read Replica**: Horizontal read scaling provisioned; read-only enforcement verified.
- 📊 **CloudWatch Monitoring**: CPU, storage, connections, and IOPS monitored with alarm-triggered SNS email alerts.
- 🛡️ **Private Subnet Isolation**: The RDS instance has no public IP and cannot be reached from the internet.

---

## 🛠️ Prerequisites

| Requirement | Details |
|---|---|
| **AWS Account** | Standard / Free Tier |
| **EC2 Key Pair (.pem)** | Required for SSH access to the EC2 instance |
| **MySQL Client (mariadb105)** | Installed on EC2 via `sudo yum install mariadb105 -y` |
| **AWS CLI** | v2.x (optional — for CLI-based provisioning) |

---

## 📂 Project Structure

```
AWS-Project/
└── RDS via EC2/
    ├── 01 RDS_Dashboard.png                      # RDS instance overview in AWS Console
    ├── 02 EC2_dashboard.png                      # EC2 instance running as client
    ├── 03 connection.png                         # Successful mysql CLI connection to RDS endpoint
    ├── 04 Student Management System output.png   # studentdb schema demo
    ├── 05 Employee Management System output.png  # employeedb schema demo
    ├── 06 Library Management System output.png   # librarydb schema demo
    ├── 07 Hospital Management System output.png  # hospitaldb schema demo
    ├── 08 Online Shopping Database output.png    # shoppingdb schema demo
    ├── 09 RDS Backup Snapshot.png                # Manual snapshot creation
    ├── 10 RDS Backup Snapshot list.png           # Snapshot list / restore view
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

### Step 4: Launch the EC2 Instance (Amazon Linux)

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
sudo yum install mariadb105 -y
```

### Step 6: Connect to RDS from EC2

```bash
mysql -h <RDS-ENDPOINT> -u admin -p
# Enter your password when prompted
```

*Verify connection:*
```sql
SHOW DATABASES;
```

---

## 🎓 Project 1 — Student Management System

### Create Database & Table

```sql
CREATE DATABASE studentdb;
USE studentdb;

CREATE TABLE students (
    id         INT AUTO_INCREMENT PRIMARY KEY,
    name       VARCHAR(100),
    department VARCHAR(50),
    age        INT,
    email      VARCHAR(100)
);
```

### Insert Data

```sql
INSERT INTO students (name, department, age, email)
VALUES
    ('Rahul', 'CSE', 20, 'rahul@gmail.com'),
    ('Priya', 'ECE', 21, 'priya@gmail.com');
```

### Queries

```sql
-- Retrieve all students
SELECT * FROM students;

-- Filter by department
SELECT * FROM students WHERE department = 'CSE';

-- Update a record
UPDATE students SET age = 22 WHERE id = 1;

-- Delete a record
DELETE FROM students WHERE id = 2;

-- Count total students
SELECT COUNT(*) FROM students;
```

---

## 👔 Project 2 — Employee Management System

### Create Database & Table

```sql
CREATE DATABASE employeedb;
USE employeedb;

CREATE TABLE employees (
    emp_id     INT AUTO_INCREMENT PRIMARY KEY,
    emp_name   VARCHAR(100),
    department VARCHAR(50),
    salary     DECIMAL(10,2)
);
```

### Queries

```sql
-- Insert an employee
INSERT INTO employees (emp_name, department, salary)
VALUES ('John', 'IT', 50000);

-- Retrieve all employees
SELECT * FROM employees;

-- Filter by salary
SELECT * FROM employees
WHERE salary > 40000;

-- Update salary
UPDATE employees
SET salary = 60000
WHERE emp_id = 1;

-- Delete an employee
DELETE FROM employees
WHERE emp_id = 1;
```

---

## 📚 Project 3 — Library Management System

### Create Database & Table

```sql
CREATE DATABASE librarydb;
USE librarydb;

CREATE TABLE books (
    book_id  INT AUTO_INCREMENT PRIMARY KEY,
    title    VARCHAR(100),
    author   VARCHAR(100),
    quantity INT
);
```

### Queries

```sql
-- Insert a book
INSERT INTO books (title, author, quantity)
VALUES ('Python Basics', 'James', 5);

-- Retrieve all books
SELECT * FROM books;

-- Update quantity
UPDATE books
SET quantity = 4
WHERE book_id = 1;

-- Delete a book
DELETE FROM books
WHERE book_id = 1;
```

---

## 🏥 Project 4 — Hospital Management System

### Create Database & Table

```sql
CREATE DATABASE hospitaldb;
USE hospitaldb;

CREATE TABLE patients (
    patient_id   INT AUTO_INCREMENT PRIMARY KEY,
    patient_name VARCHAR(100),
    disease      VARCHAR(100),
    doctor       VARCHAR(100)
);
```

### Queries

```sql
-- Insert a patient
INSERT INTO patients (patient_name, disease, doctor)
VALUES ('Ravi', 'Fever', 'Dr Kumar');

-- Retrieve all patients
SELECT * FROM patients;

-- Filter by disease
SELECT * FROM patients
WHERE disease = 'Fever';

-- Update assigned doctor
UPDATE patients
SET doctor = 'Dr Raj'
WHERE patient_id = 1;
```

---

## 🛒 Project 5 — Online Shopping Database

### Create Database & Table

```sql
CREATE DATABASE shoppingdb;
USE shoppingdb;

CREATE TABLE products (
    product_id   INT AUTO_INCREMENT PRIMARY KEY,
    product_name VARCHAR(100),
    price        DECIMAL(10,2),
    stock        INT
);
```

### Queries

```sql
-- Insert a product
INSERT INTO products (product_name, price, stock)
VALUES ('Laptop', 50000, 10);

-- Retrieve all products
SELECT * FROM products;

-- Filter by price
SELECT * FROM products
WHERE price > 10000;

-- Update stock
UPDATE products
SET stock = 8
WHERE product_id = 1;

-- Delete a product
DELETE FROM products
WHERE product_id = 1;
```

---

## 📸 Project 6 — RDS Backup & Snapshot

Amazon RDS provides **automated daily backups** and allows **manual on-demand snapshots** that can be used for point-in-time recovery or cross-region restore.

### Step 1: Create a Manual Snapshot (Console)

1. Navigate to **RDS → Databases → Select your instance**.
2. Click **Actions → Take Snapshot**.
3. Enter a snapshot name and click **Take Snapshot**.

### Verify Data Before Snapshot

```sql
USE studentdb;
SELECT * FROM students;
```

### Step 2: Restore from Snapshot (Console)

1. Navigate to **RDS → Snapshots**.
2. Select your snapshot → click **Actions → Restore Snapshot**.
3. Configure the new instance name and settings → click **Restore DB Instance**.

### Verify Restored Data

```sql
USE studentdb;
SELECT * FROM students;
```

> ✅ If the data matches the pre-snapshot state, the restore is successful.

### Creating a Snapshot via CLI

```bash
aws rds create-db-snapshot \
  --db-instance-identifier "prem-rds-mysql" \
  --db-snapshot-identifier "prem-rds-manual-snapshot-01" \
  --region us-east-1
```

---

## 🔄 Project 7 — Multi-AZ High Availability Demo

**Multi-AZ** maintains a **synchronous standby replica** in a different Availability Zone. In the event of a primary failure, RDS automatically fails over to the standby (typically within 60–120 seconds) with no data loss.

### Step 1: Enable Multi-AZ (Console)

1. Navigate to **RDS → Databases → Select your instance**.
2. Click **Modify**.
3. Under **Availability & Durability**, set **Multi-AZ Deployment = Yes**.
4. Click **Continue → Apply Immediately**.

> ⏳ Wait several minutes for the modification to complete.

### Verification

```sql
SHOW DATABASES;
```

> Confirms the primary is functioning normally after Multi-AZ is enabled.

### Step 2: Failover Test

1. Navigate to **RDS → Databases → Select your instance**.
2. Click **Actions → Reboot**.
3. Select **Reboot with Failover** → click **Confirm**.

> ⏳ The endpoint DNS name remains the same — RDS automatically re-points it to the new primary.

### Reconnect After Failover

```bash
mysql -h <rds-endpoint> -u admin -p
```

> ✅ Successful reconnection confirms automatic failover completed.

---

## 📖 Project 8 — Read Replica Demo

A **Read Replica** is an asynchronous copy of the primary RDS instance used to offload read-heavy workloads (reports, analytics, dashboards) without impacting write performance on the primary.

### Key Facts

- Read Replicas use **asynchronous replication** — there may be a small replication lag.
- They have a **separate DNS endpoint** — applications must explicitly connect to the replica for reads.
- Read Replicas can be **promoted** to standalone databases if the primary fails (manual failover).
- Multi-AZ is for **high availability (failover)**; Read Replicas are for **read scalability** — these are different features.

### Step 1: Create Primary Database

Ensure `studentdb` and the `students` table exist on the primary instance (see Project 1).

### Step 2: Create Read Replica (Console)

1. Navigate to **RDS → Databases → Select your primary instance**.
2. Click **Actions → Create Read Replica**.
3. Choose destination region and instance class → click **Create Read Replica**.

### Write on Primary

```sql
INSERT INTO students (name, department, age, email)
VALUES ('Kiran', 'CSE', 21, 'kiran@gmail.com');
```

### Read on Replica

```bash
mysql -h <READ-REPLICA-ENDPOINT> -u admin -p
```

```sql
USE studentdb;
SELECT * FROM students;
```

> ✅ The record for `Kiran` appears — confirming replication from primary to replica.

### Test Read-Only Enforcement

```sql
INSERT INTO students VALUES (...);
```

**Expected Error:**

```
ERROR 1290 (HY000): The MySQL server is running with the --read-only option so it cannot execute this statement
```

### Creating a Read Replica via CLI

```bash
aws rds create-db-instance-read-replica \
  --db-instance-identifier "prem-rds-read-replica" \
  --source-db-instance-identifier "prem-rds-mysql" \
  --db-instance-class db.t3.micro \
  --region us-east-1
```

---

## 🔌 Project 9 — Connect EC2 to RDS

This project validates the complete **EC2-to-RDS connectivity** workflow using Amazon Linux and the `mariadb105` MySQL client package.

### Step 1: Create EC2 (Amazon Linux)

Launch an EC2 instance with:
- **AMI**: Amazon Linux 2023
- **Instance Type**: t2.micro / t3.micro
- **Security Group**: Allow port 22 from your IP

### Step 2: Install MySQL Client on EC2

```bash
sudo yum install mariadb105 -y
```

### Step 3: Connect to RDS

```bash
mysql -h <rds-endpoint> -u admin -p
```

### Step 4: Verify Connection

```sql
SHOW DATABASES;

USE studentdb;

SELECT * FROM students;
```

> ✅ Seeing all five databases (`studentdb`, `employeedb`, `librarydb`, `hospitaldb`, `shoppingdb`) confirms a successful EC2-to-RDS connection.

---

## 📊 Project 10 — RDS Monitoring with CloudWatch

### Step 1: Enable Enhanced Monitoring

1. Navigate to **RDS → Databases → Select your instance**.
2. Go to the **Monitoring** tab.
3. Scroll down to **CloudWatch Metrics** to see real-time graphs.

### Step 2: Key Metrics to Monitor

| Metric | What It Measures |
|---|---|
| **CPU Utilization** | Database engine CPU usage (%) |
| **Free Storage Space** | Available disk space on the RDS volume |
| **Database Connections** | Number of active client connections |
| **Read IOPS** | Read operations per second |
| **Write IOPS** | Write operations per second |

### Step 3: Generate Load to See Metric Changes

```sql
-- Insert test data to generate write activity
INSERT INTO students (name, department, age, email)
VALUES ('Test', 'CSE', 20, 'test@gmail.com');
```

Run the following repeatedly to generate read load:

```sql
SELECT * FROM students;
```

> 📈 Observe the **Read IOPS** and **Database Connections** metrics spike in CloudWatch.

### Step 4: Create a CloudWatch Alarm

1. Navigate to **CloudWatch → Alarms → Create Alarm**.
2. Select metric: **RDS → Per-Database Metrics → CPUUtilization**.
3. Set condition: **Greater than 80%** for **1 consecutive period (5 min)**.
4. Set action: **Send notification to SNS Topic** → configure an **Email Endpoint**.
5. Enter alarm name: `RDS-High-CPU-Alarm`.
6. Click **Create Alarm**.

> ✅ When CPU exceeds 80%, CloudWatch sends an email alert via SNS.

### Step 5: Verify Alarm Configuration

```bash
aws cloudwatch describe-alarms \
  --alarm-names "RDS-High-CPU-Alarm" \
  --region us-east-1
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
> MySQL query output showing `CREATE DATABASE studentdb`, `CREATE TABLE students`, `INSERT`, `SELECT`, `UPDATE`, `DELETE`, and `COUNT(*)` results.

![Student Management System](04%20Student%20Management%20System%20output.png)

---

### 5️⃣ Employee Management System Output
> MySQL query output for `employeedb` demonstrating HR schema creation, data insertion, salary filter, update, and delete.

![Employee Management System](05%20Employee%20Management%20System%20output.png)

---

### 6️⃣ Library Management System Output
> MySQL query output for `librarydb` showing book record management including insert, select, update, and delete.

![Library Management System](06%20Library%20Management%20System%20output.png)

---

### 7️⃣ Hospital Management System Output
> MySQL query output for `hospitaldb` showing patient record management with disease filter and doctor update.

![Hospital Management System](07%20Hospital%20Management%20System%20output.png)

---

### 8️⃣ Online Shopping Database Output
> MySQL query output for `shoppingdb` showing product catalog management with price filter and stock update.

![Online Shopping Database](08%20Online%20Shopping%20Database%20output.png)

---

### 9️⃣ RDS Backup Snapshot Creation
> AWS Console showing the manual snapshot being created with status `Creating`.

![RDS Backup Snapshot](09%20RDS%20Backup%20Snapshot.png)

---

### 🔟 RDS Backup Snapshot List
> AWS Console showing the completed manual snapshot in `Available` status, ready for restore.

![RDS Backup Snapshot List](10%20RDS%20Backup%20Snapshot%20list.png)

---

### 1️⃣1️⃣ Read Replica Demo
> AWS Console showing the Read Replica instance in `Available` state alongside the primary instance, with its own unique endpoint.

![Read Replica Demo](11%20Read%20Replica%20Demo.png)

---

## 🐛 Common Issues & Troubleshooting

| Issue | Cause | Fix |
|---|---|---|
| `ERROR 2003: Can't connect to MySQL server` | EC2-SG not referenced in RDS-SG, or wrong endpoint | Verify the RDS Security Group inbound rule references the EC2-SG ID. Confirm you are using the RDS endpoint DNS name. |
| Connection hangs / times out | RDS in private subnet with no route from EC2 subnet | Ensure both EC2 and RDS are in the same VPC. Check route tables and subnet associations. |
| `Access denied for user 'admin'@'...'` | Incorrect password or username | Double-check the master username and password set during RDS creation. |
| `sudo yum install mariadb105 -y` fails | Wrong package name for Amazon Linux version | Use `sudo dnf install mariadb105 -y` on Amazon Linux 2023. |
| Read Replica shows replication lag | High write throughput on primary | Expected under heavy write load. Monitor `ReplicaLag` in CloudWatch. |
| `ERROR 1290: The MySQL server is running with --read-only` | Connected to Read Replica endpoint | Read Replicas do not accept writes. Connect to the primary endpoint for writes. |
| RDS snapshot takes too long | Large database size | Snapshot time scales with storage. Monitor snapshot status in the console. |
| Multi-AZ failover does not complete | Insufficient wait time after reboot | Wait 60–120 seconds; the endpoint DNS automatically re-points to the new primary. |
| CloudWatch alarm not triggering | Threshold too high, or insufficient load | Lower the CPU threshold for testing, or run a heavier SELECT loop to generate load. |

---

## 🧹 Cleanup / Destroy

> ⚠️ **Billing Warning**: RDS instances, Read Replicas, and Multi-AZ standbys incur ongoing hourly costs even when idle. Clean up immediately after validation.

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

### Step 6: Delete CloudWatch Alarm
```bash
aws cloudwatch delete-alarms \
  --alarm-names "RDS-High-CPU-Alarm" \
  --region us-east-1
```

### Step 7: Delete Manual Snapshots
```bash
aws rds delete-db-snapshot \
  --db-snapshot-identifier "prem-rds-manual-snapshot-01" \
  --region us-east-1
```

---

## 🔮 Future Improvements

1. **RDS Proxy**: Add an RDS Proxy in front of the database to pool and manage connections from Lambda functions or containerized applications.
2. **Secrets Manager Integration**: Store the RDS master password in AWS Secrets Manager and rotate it automatically on a schedule.
3. **IAM Database Authentication**: Replace static password authentication with IAM token-based authentication for improved security.
4. **VPC Endpoints for CloudWatch**: Route CloudWatch metrics through a VPC Interface Endpoint to keep monitoring traffic within the AWS network.
5. **Parameter Groups**: Create a custom RDS Parameter Group to tune MySQL settings (`innodb_buffer_pool_size`, `max_connections`, etc.) for production workloads.
6. **Automated Failover Testing with AWS FIS**: Use AWS Fault Injection Simulator to automate regular Multi-AZ failover drills.

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

### Conventional Commit Types

```
feat      → New architectural components, configurations, or features
fix       → Corrections to SQL queries, CLI commands, or configurations
docs      → Updates to README or diagrams
chore     → Routine maintenance, formatting, or cleanup tasks
```

---

## 📄 License

```
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
```

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

*If this project helped you understand Amazon RDS, EC2-to-RDS connectivity, database backups, Multi-AZ failover, read replicas, or CloudWatch monitoring — a star supports open-source cloud documentation.*

<br/>

![AWS](https://img.shields.io/badge/Built%20on-AWS-%23FF9900?style=flat-square&logo=amazon-aws&logoColor=white)
![RDS](https://img.shields.io/badge/Feature-Amazon%20RDS-527FFF?style=flat-square&logo=amazonrds&logoColor=white)
![MySQL](https://img.shields.io/badge/Engine-MySQL-4479A1?style=flat-square&logo=mysql&logoColor=white)
![CloudWatch](https://img.shields.io/badge/Monitoring-CloudWatch-FF4F8B?style=flat-square&logo=amazon-aws&logoColor=white)
![Made in India](https://img.shields.io/badge/Made%20with%20%E2%9D%A4%EF%B8%8F%20in-India-blue?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-22c55e?style=flat-square)

*© 2026 Prem Kumar S*

</div>
