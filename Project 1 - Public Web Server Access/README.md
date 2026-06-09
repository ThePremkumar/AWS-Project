![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Amazon EC2](https://img.shields.io/badge/Amazon%20EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white)
![Amazon VPC](https://img.shields.io/badge/Amazon%20VPC-8C4FFF?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Apache](https://img.shields.io/badge/Apache%20httpd-D22128?style=for-the-badge&logo=apache&logoColor=white)
![License: MIT](https://img.shields.io/badge/License-MIT-22c55e?style=for-the-badge)
![Status: Active](https://img.shields.io/badge/Status-Active-brightgreen?style=for-the-badge)
![Made with ❤️ in India](https://img.shields.io/badge/Made%20with%20%E2%9D%A4%EF%B8%8F%20in-India-blue?style=for-the-badge)

---

# 🌐 Project 1: Public Web Server Access Control

> **Deploy a publicly accessible Apache HTTP web server on AWS** — built inside a custom VPC with a hardened security group, restricted SSH management plane, and end-to-end validated browser connectivity over HTTP.

This project models a real-world baseline cloud infrastructure pattern: isolating compute in a purpose-built network, exposing only what is necessary to the public internet, and locking down administrative access to a known IP.

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
- [Security Highlights](#-security-highlights)
- [Testing & Validation](#-testing--validation)
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

This project provisions a **production-baseline public web server** on AWS using the management console. An Apache HTTP Server (`httpd 2.4.67`) runs on an Amazon Linux 2023 EC2 instance (`t3.micro`) inside a custom Virtual Private Cloud. The server is reachable globally on **port 80** while SSH administrative access on **port 22** is locked to a single trusted IP address.

### Real-World Use Case

Every SaaS product, internal tool, or microservice in production lives behind infrastructure nearly identical to this pattern:

- A **VPC** provides network isolation from other AWS customers
- **Public subnets** with an Internet Gateway enable internet-facing services
- **Security groups** act as the first line of defence at the instance level
- **EC2 + Apache** form the compute and serving layer

### Problem Solved

> "How do I expose a web service to the public internet on AWS while keeping management access completely locked down?"

This project answers that question end-to-end — from VPC creation through to a browser-verified live webpage — making it the ideal reference for anyone learning cloud networking, security hardening, or EC2 provisioning.

---

## 🏗️ Architecture Diagram

```
┌──────────────────────────────────────────────────────────────────────────┐
│                           PUBLIC INTERNET                                │
│                     (Browser → http://44.195.40.245)                     │
└────────────────────────────────┬─────────────────────────────────────────┘
                                 │  HTTP :80 / SSH :22
                                 ▼
┌──────────────────────────────────────────────────────────────────────────┐
│                    INTERNET GATEWAY (IGW)                                │
│              Attached to vpc-05f63b6d0d18fa5ff (Prem-VPC)               │
└────────────────────────────────┬─────────────────────────────────────────┘
                                 │
                                 ▼
┌──────────────────────────────────────────────────────────────────────────┐
│                      Prem-VPC  (us-east-1)                               │
│               vpc-05f63b6d0d18fa5ff                                      │
│                                                                          │
│  ┌─────────────────────────────────┐  ┌─────────────────────────────┐   │
│  │  Prem-Public-A  (us-east-1a)   │  │  Prem-Public-B (us-east-1b) │   │
│  │  subnet-07476134253af8471       │  │  subnet-0f7e7f6acef2acf3a   │   │
│  │  CIDR: 20.0.1.0/24             │  │  CIDR: 20.0.2.0/24          │   │
│  │                                 │  │  (HA-ready / spare)         │   │
│  │  ┌───────────────────────────┐  │  │                             │   │
│  │  │   EC2: Web Server         │  │  │                             │   │
│  │  │   i-051e04658c7577de3     │  │  │                             │   │
│  │  │   t3.micro                │  │  │                             │   │
│  │  │   Amazon Linux 2023       │  │  │                             │   │
│  │  │   Public IP: 44.195.40.245│  │  │                             │   │
│  │  │   Private: ip-20-0-1-45   │  │  │                             │   │
│  │  │                           │  │  │                             │   │
│  │  │  ┌─────────────────────┐  │  │  │                             │   │
│  │  │  │  Apache httpd 2.4.67│  │  │  │                             │   │
│  │  │  │  Listening :80      │  │  │  │                             │   │
│  │  │  │  /var/www/html/     │  │  │  │                             │   │
│  │  │  └─────────────────────┘  │  │  │                             │   │
│  │  └───────────┬───────────────┘  │  │                             │   │
│  │              │                   │  │                             │   │
│  │  ┌───────────▼───────────────┐  │  │                             │   │
│  │  │  Security Group           │  │  │                             │   │
│  │  │  sg-0950304b5cf423e70     │  │  │                             │   │
│  │  │  "webserver Apache"       │  │  │                             │   │
│  │  │  ✅ :80 ← 0.0.0.0/0      │  │  │                             │   │
│  │  │  🔒 :22 ← 27.7.187.252/32│  │  │                             │   │
│  │  └───────────────────────────┘  │  │                             │   │
│  └─────────────────────────────────┘  └─────────────────────────────┘   │
│                                                                          │
│   Route Table: 0.0.0.0/0 → IGW  (all public subnets)                    │
└──────────────────────────────────────────────────────────────────────────┘
```

**Traffic Flow:**

```
User Browser
    → DNS/IP (44.195.40.245)
    → Internet Gateway (Prem-VPC)
    → Route Table (0.0.0.0/0 → IGW)
    → Prem-Public-A (20.0.1.0/24)
    → Security Group (:80 ALLOW from 0.0.0.0/0)
    → EC2 Instance (i-051e04658c7577de3)
    → Apache httpd (listening :80)
    → Serves /var/www/html/index.html
    → HTTP 200 Response → Browser
```

---

## ☁️ AWS Services Used

| Service | Purpose | Configuration Observed |
|---|---|---|
| **Amazon VPC** | Isolated private network for all resources | `vpc-05f63b6d0d18fa5ff` — named `Prem-VPC`, region `us-east-1` |
| **Public Subnets** | Host internet-accessible resources | `Prem-Public-A` (`20.0.1.0/24`, AZ-1a) · `Prem-Public-B` (`20.0.2.0/24`, AZ-1b) |
| **Internet Gateway** | Bridges VPC to the public internet | Attached to `Prem-VPC`; route `0.0.0.0/0 → IGW` in public route table |
| **Route Tables** | Direct traffic from subnets to IGW | Public route: `0.0.0.0/0 → IGW` applied to both public subnets |
| **Amazon EC2** | Virtual machine running the web server | `i-051e04658c7577de3` · `t3.micro` · Amazon Linux 2023 · Public IP `44.195.40.245` |
| **Security Groups** | Stateful instance-level firewall | `sg-0950304b5cf423e70` — `webserver Apache`: SSH restricted, HTTP open |
| **Apache httpd** | HTTP web server software (on EC2) | `httpd-2.4.67-1.amzn2023.0.1.x86_64` · Active (running) · Port 80 |

---

## ✨ Key Features

- 🏠 **Custom VPC** — dedicated network with full control over IP addressing (`20.0.0.0/16`)
- 🌍 **Multi-AZ Subnet Design** — two public subnets across `us-east-1a` and `us-east-1b` for resilience
- 🔒 **Least-Privilege SSH** — port 22 restricted to a single IP (`27.7.187.252/32`); no open-world SSH
- 🌐 **Public HTTP Access** — port 80 open to `0.0.0.0/0` for full public web serving
- ⚙️ **Apache 2.4.67 on Amazon Linux 2023** — production-grade, up-to-date web server stack
- ✅ **End-to-End Verified** — browser-confirmed live webpage at the EC2 public IP
- 📸 **Fully Documented** — every step captured with screenshots for reproducibility
- 💰 **Free Tier Friendly** — `t3.micro` instance type, single region, minimal resource footprint

---

## ✅ Prerequisites

| Requirement | Detail | Link |
|---|---|---|
| **AWS Account** | Free Tier eligible | [aws.amazon.com/free](https://aws.amazon.com/free/) |
| **IAM Permissions** | `ec2:*`, `vpc:*` on target region | [IAM Docs](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started.html) |
| **SSH Key Pair** | `.pem` file generated at EC2 launch | AWS Console → EC2 → Key Pairs |
| **SSH Client** | Terminal, Git Bash, PuTTY, or WSL | — |
| **Web Browser** | Any modern browser for validation | — |
| **AWS Region** | `us-east-1` (N. Virginia) | All resources deployed here |

### IAM Minimum Permissions

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:*",
        "vpc:*"
      ],
      "Resource": "*"
    }
  ]
}
```

---

## 📁 Project Structure

```
AWS Project/
└── Project 1 - Public Web Server Access/
    │
    ├── README.md                               ← This file
    │
    └── output/                                 ← Evidence & validation screenshots
        ├── 01_Public_Subnet.png                ← VPC subnets (Prem-Public-A/B) in Available state
        ├── 02_EC2_Running.png                  ← EC2 instance summary, public IP, Running state
        ├── 03_Security_Group_Rules.png         ← Inbound rules: SSH :22 + HTTP :80
        ├── 04_Apache_Service_Running.png       ← systemctl status httpd — active (running)
        └── 05_Website_Accessed_From_Browser.png ← Browser at 44.195.40.245 — live page
```

> **Note:** This project was built using the **AWS Management Console** (no Terraform/CLI).
> All configuration is manual and fully reproducible following this README.

---

## 🚀 Setup & Deployment

### Step 1 — Create the VPC

1. Open **AWS Console** → **VPC** → **Your VPCs** → **Create VPC**
2. Set the following:

```
Name tag:   Prem-VPC
IPv4 CIDR:  20.0.0.0/16
Tenancy:    Default
```

3. Click **Create VPC**

---

### Step 2 — Create Public Subnets

Navigate to **VPC → Subnets → Create Subnet** and create both:

| Subnet Name | VPC | Availability Zone | IPv4 CIDR |
|---|---|---|---|
| `Prem-Public-A` | `Prem-VPC` | `us-east-1a` | `20.0.1.0/24` |
| `Prem-Public-B` | `Prem-VPC` | `us-east-1b` | `20.0.2.0/24` |

---

### Step 3 — Attach Internet Gateway

```
VPC → Internet Gateways → Create Internet Gateway
  Name: Prem-IGW

Actions → Attach to VPC → Select: Prem-VPC → Attach
```

---

### Step 4 — Update Route Table

```
VPC → Route Tables → Select the route table for Prem-VPC
Routes tab → Edit Routes → Add Route:
  Destination:  0.0.0.0/0
  Target:       Prem-IGW (Internet Gateway)

Subnet Associations → Edit → Associate both Prem-Public-A and Prem-Public-B
```

---

### Step 5 — Create Security Group

Navigate to **EC2 → Security Groups → Create Security Group**:

```
Name:        webserver Apache
Description: Allow SSH from admin IP and HTTP from anywhere
VPC:         Prem-VPC
```

**Add Inbound Rules:**

| Type | Protocol | Port Range | Source | Description |
|---|---|---|---|---|
| SSH | TCP | 22 | `27.7.187.252/32` | Admin-only SSH |
| HTTP | TCP | 80 | `0.0.0.0/0` | Public web traffic |

> ⚠️ Replace `27.7.187.252/32` with your own public IP. Find it at [whatismyip.com](https://www.whatismyip.com/)

---

### Step 6 — Launch EC2 Instance

Navigate to **EC2 → Instances → Launch Instances**:

| Setting | Value |
|---|---|
| **Name** | `Web Server` |
| **AMI** | Amazon Linux 2023 AMI (x86_64) |
| **Instance Type** | `t3.micro` |
| **Key Pair** | Create or select existing `.pem` key |
| **VPC** | `Prem-VPC` |
| **Subnet** | `Prem-Public-A` |
| **Auto-assign Public IP** | **Enable** |
| **Security Group** | `webserver Apache` |

Click **Launch Instance** and wait for the state to show **Running**.

---

### Step 7 — Connect via SSH

```bash
# Set permissions on your key file
chmod 400 your-key.pem

# Connect to the instance
ssh -i "your-key.pem" ec2-user@44.195.40.245
```

---

### Step 8 — Install and Start Apache

```bash
# Elevate to root
sudo su

# Update all installed packages
yum update -y

# Install Apache HTTP Server
yum install httpd -y

# Start the Apache service
systemctl start httpd

# Enable Apache to persist across reboots
systemctl enable httpd

# Confirm the service is active
systemctl status httpd
```

Expected status output (as seen in screenshot `04`):

```
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; preset: disabled)
     Active: active (running) since Tue 2026-06-09 06:16:08 UTC; 9s ago
    Main PID: 25920 (httpd)
      Status: "Total requests: 0; Idle/Busy workers 100/0;Requests/sec: 0; Bytes served/sec: 0 B/sec"
      Memory: 13.5M
         CPU: 71ms
...
Jun 09 06:16:08 ip-20-0-1-45.ec2.internal httpd[25920]: Server configured, listening on: port 80
```

---

### Step 9 — Deploy Custom Web Page

```bash
# Navigate to Apache web root
cd /var/www/html

# Create the index page
cat > index.html << 'EOF'
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>AWS EC2 Web Server</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
      background: #f0f2f5;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
    }
    .card {
      background: white;
      border-radius: 16px;
      padding: 48px 64px;
      box-shadow: 0 8px 32px rgba(0,0,0,0.08);
      text-align: center;
      max-width: 600px;
    }
    h1 { color: #1a6bbf; font-size: 1.8rem; margin-bottom: 16px; }
    .status { color: #16a34a; font-weight: 700; font-size: 1.1rem; margin-bottom: 24px; }
    p { color: #4b5563; line-height: 1.8; margin: 4px 0; }
    strong { color: #111827; }
  </style>
</head>
<body>
  <div class="card">
    <h1>Project 1: Public Web Server Access Control</h1>
    <p class="status">✅ Apache Web Server is Running Successfully!</p>
    <p>This webpage is hosted on an AWS EC2 instance.</p>
    <p><strong>Instance Type:</strong> t3.micro</p>
    <p><strong>Web Server:</strong> Apache HTTP Server (httpd)</p>
    <p><strong>Protocol:</strong> HTTP (Port 80)</p>
    <p><strong>Created By:</strong> Prem Kumar S</p>
  </div>
</body>
</html>
EOF
```

---

### Step 10 — Verify Public Access

Open any browser and navigate to:

```
http://44.195.40.245
```

✅ You should see the **"Project 1: Public Web Server Access Control"** page confirming the Apache server is live.

---

## 🔍 How It Works

### 1. Custom VPC (`Prem-VPC`)
The VPC (`vpc-05f63b6d0d18fa5ff`) creates a logically isolated network within AWS. Using the `20.0.0.0/16` range provides 65,536 private IP addresses to partition into subnets. Without a VPC, EC2 instances would share the default AWS network with no logical separation.

### 2. Public Subnets (`Prem-Public-A` / `Prem-Public-B`)
Subnets `20.0.1.0/24` and `20.0.2.0/24` are carved from the VPC range. Placing them in separate Availability Zones (`us-east-1a` and `us-east-1b`) is a HA design principle — if one AZ fails, the other subnet remains available. Both subnets have **Auto-assign Public IP** enabled so instances receive a routable public IP at launch.

### 3. Internet Gateway + Route Table
The IGW acts as the VPC's on-ramp to the internet. Without it, traffic cannot enter or leave the VPC. The route table entry `0.0.0.0/0 → IGW` tells AWS: *"any traffic destined for an IP not inside the VPC should go to the Internet Gateway."* This is what makes a subnet "public."

### 4. Security Group (`webserver Apache` — `sg-0950304b5cf423e70`)
Security groups are stateful — return traffic for allowed connections is automatically permitted. Two inbound rules are enforced:
- **SSH :22** is locked to `27.7.187.252/32` (a single `/32` CIDR — one IP address). This prevents brute-force and unauthorized access attempts from the public internet.
- **HTTP :80** is open to `0.0.0.0/0` because the entire purpose of the server is public web access.

### 5. EC2 Instance (`i-051e04658c7577de3`)
The `t3.micro` instance runs Amazon Linux 2023 (based on the package versions: `amzn2023`). It is placed in `Prem-Public-A` and assigned the public IP `44.195.40.245`. The private DNS `ip-20-0-1-45.ec2.internal` corresponds to the private IP `20.0.1.45` — confirming the instance received an IP from the `20.0.1.0/24` subnet as expected.

### 6. Apache HTTP Server (`httpd 2.4.67`)
Apache is installed via `yum` from the Amazon Linux 2023 package repository. Version `httpd-2.4.67-1.amzn2023.0.1.x86_64` is installed with 13 packages and dependencies. On start, Apache forks worker processes (PIDs 25920–25924) and binds to `0.0.0.0:80`, making the web root `/var/www/html/` publicly accessible.

---

## 🛡️ Security Highlights

### Inbound Rules — Security Group `webserver Apache`

| Rule ID | Type | Protocol | Port | Source | Action | Reasoning |
|---|---|---|---|---|---|---|
| `sgr-05ecca8d58327a465` | SSH | TCP | 22 | `27.7.187.252/32` | ✅ ALLOW | Restricts shell access to a single trusted IP — least privilege admin |
| `sgr-0ab185bd952bfce30` | HTTP | TCP | 80 | `0.0.0.0/0` | ✅ ALLOW | Public web access — intentionally open for this web server use case |

### Outbound Rules
Security groups allow all outbound traffic by default. This enables the instance to:
- Pull packages via `yum update` and `yum install`
- Send HTTP responses back to clients (stateful — permitted automatically)

### Security Design Decisions

| Decision | Implementation | Why |
|---|---|---|
| SSH restricted by IP | `/32` CIDR on port 22 | Prevents brute-force from any unauthorized IP |
| No `0.0.0.0/0` on SSH | Source: `27.7.187.252/32` only | Industry best practice — never expose SSH to the world |
| HTTP open to world | `0.0.0.0/0` on port 80 | Required — this is a public web server |
| No HTTPS (port 443) | Not configured | Scope of this project is HTTP; TLS is a future enhancement |
| No key-pair password | PEM-based authentication | Stronger than password auth; key must be guarded |

---

## 🧪 Testing & Validation

### 1. Verify EC2 Instance is Running

```bash
aws ec2 describe-instances \
  --instance-ids i-051e04658c7577de3 \
  --query 'Reservations[*].Instances[*].[InstanceId,State.Name,PublicIpAddress]' \
  --output table \
  --region us-east-1
```

### 2. Verify Apache Service on Instance

```bash
# SSH into the instance first
ssh -i "your-key.pem" ec2-user@44.195.40.245

# Check service status
systemctl status httpd

# Verify Apache is listening on port 80
ss -tlnp | grep :80

# Test local response
curl -I http://localhost
```

Expected:
```
HTTP/1.1 200 OK
Server: Apache/2.4.67 (Amazon Linux)
```

### 3. Verify Public HTTP Access

```bash
# From your local machine — test HTTP response
curl -I http://44.195.40.245
```

Expected:
```
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Server: Apache/2.4.67 (Amazon Linux)
```

### 4. Verify Security Group Rules via CLI

```bash
aws ec2 describe-security-groups \
  --group-ids sg-0950304b5cf423e70 \
  --query 'SecurityGroups[*].IpPermissions' \
  --output json \
  --region us-east-1
```

### 5. Browser Validation

Navigate to `http://44.195.40.245` — the live page should display:
- Title: *"Project 1: Public Web Server Access Control"*
- Status: *"Apache Web Server is Running Successfully!"*
- Fields: Instance Type, Web Server, Protocol, Created By

---

## 📸 Screenshots

### 1️⃣ Public Subnets — VPC Dashboard

> Both `Prem-Public-A` (`20.0.1.0/24`) and `Prem-Public-B` (`20.0.2.0/24`) created inside `Prem-VPC` with state **Available**.

![01 - Public Subnets Created in VPC](./output/01_Public_Subnet.png)

---

### 2️⃣ EC2 Instance — Running State

> Instance `i-051e04658c7577de3` (Web Server) in **Running** state. Public IP `44.195.40.245` assigned. Instance type `t3.micro`. Subnet `Prem-Public-A`.

![02 - EC2 Instance Running with Public IP](./output/02_EC2_Running.png)

---

### 3️⃣ Security Group — Inbound Rules

> Security group `webserver Apache` (`sg-0950304b5cf423e70`) showing two inbound rules: SSH on port 22 restricted to `27.7.187.252/32`, HTTP on port 80 open to `0.0.0.0/0`.

![03 - Security Group Inbound Rules](./output/03_Security_Group_Rules.png)

---

### 4️⃣ Apache httpd — Active and Running

> Terminal output of `systemctl status httpd` showing `active (running)` since `2026-06-09 06:16:08 UTC`. Apache `2.4.67` listening on port 80. PID `25920`, Memory `13.5M`.

![04 - Apache Service Active via systemctl](./output/04_Apache_Service_Running.png)

---

### 5️⃣ Website — Browser Verified at Public IP

> Custom HTML page at `http://44.195.40.245` loaded successfully — confirms full end-to-end connectivity: Internet → IGW → Subnet → Security Group → EC2 → Apache → Browser.

![05 - Website Live in Browser](./output/05_Website_Accessed_From_Browser.png)

---

## 🐛 Common Issues & Troubleshooting

| Issue | Likely Cause | Fix |
|---|---|---|
| Browser shows "This site can't be reached" | Port 80 blocked in Security Group | Verify inbound rule: TCP 80 `0.0.0.0/0` exists in `webserver Apache` |
| SSH: `Permission denied (publickey)` | Wrong username or key file | Use `ec2-user` for Amazon Linux 2023; ensure `.pem` has `chmod 400` |
| SSH timeout / connection refused | Your IP changed, or SG blocks port 22 | Update the SSH inbound rule source to your current IP (`curl ifconfig.me`) |
| Apache not found after `yum install` | `yum` cache stale or package name wrong | Run `yum clean all && yum install httpd -y` |
| Apache starts but page shows default test page | Custom `index.html` not written to `/var/www/html/` | Confirm file exists: `ls -la /var/www/html/index.html` |
| EC2 has no public IP | Auto-assign Public IP was disabled at launch | Allocate and associate an Elastic IP, or relaunch with auto-assign enabled |
| VPC route table not routing to internet | IGW not attached or route missing | VPC → Route Tables → confirm `0.0.0.0/0 → igw-xxxxx` entry exists |
| `httpd` stops after SSH session ends | Service not enabled for auto-start | Run `systemctl enable httpd` to persist across reboots |

---

## 🧹 Cleanup / Destroy

> ⚠️ **Billing Warning:** AWS charges for running EC2 instances, Elastic IPs, and data transfer. Delete all resources when done to avoid unexpected costs.

### Recommended Deletion Order

**1. Terminate EC2 Instance**
```bash
aws ec2 terminate-instances \
  --instance-ids i-051e04658c7577de3 \
  --region us-east-1
```

Wait for instance state → `terminated` before proceeding.

**2. Delete Security Group**
```bash
aws ec2 delete-security-group \
  --group-id sg-0950304b5cf423e70 \
  --region us-east-1
```

**3. Delete Subnets**
```bash
# Prem-Public-A
aws ec2 delete-subnet \
  --subnet-id subnet-07476134253af8471 \
  --region us-east-1

# Prem-Public-B
aws ec2 delete-subnet \
  --subnet-id subnet-0f7e7f6acef2acf3a \
  --region us-east-1
```

**4. Detach and Delete Internet Gateway**
```bash
# Detach from VPC
aws ec2 detach-internet-gateway \
  --internet-gateway-id <igw-id> \
  --vpc-id vpc-05f63b6d0d18fa5ff \
  --region us-east-1

# Delete the IGW
aws ec2 delete-internet-gateway \
  --internet-gateway-id <igw-id> \
  --region us-east-1
```

**5. Delete VPC**
```bash
aws ec2 delete-vpc \
  --vpc-id vpc-05f63b6d0d18fa5ff \
  --region us-east-1
```

**6. Verify via Console**

> Always cross-check in the **AWS Console → EC2 → Instances** and **VPC → Your VPCs** to confirm all resources are removed.

---

## 🔮 Future Improvements

1. **HTTPS / TLS Termination** — Add an ACM certificate and configure port 443 with a redirect from port 80. Use an Application Load Balancer (ALB) to handle TLS termination cleanly.

2. **Auto Scaling Group (ASG)** — Replace the single EC2 with an ASG across `Prem-Public-A` and `Prem-Public-B` to achieve horizontal scaling and automatic recovery from instance failure.

3. **Infrastructure as Code (Terraform)** — Convert all AWS Console steps into `.tf` files for repeatable, version-controlled deployments. Eliminates manual click-ops entirely.

4. **NACL Layer** — Add a Network Access Control List (NACL) at the subnet boundary as a stateless second layer of security, complementing the stateful Security Group.

5. **Monitoring & Alerting** — Enable CloudWatch metrics and set alarms on `CPUUtilization`, `StatusCheckFailed`, and Apache access logs via CloudWatch Agent.

---

## 🤝 Contributing

All contributions are welcome — from typo fixes to new project additions.

```bash
# 1. Fork this repository on GitHub
# Click the "Fork" button → creates your own copy

# 2. Clone your fork locally
git clone https://github.com/<your-username>/<repo-name>.git
cd <repo-name>

# 3. Create a feature branch (never commit to main directly)
git checkout -b feat/your-feature-name

# 4. Make your changes
# ...

# 5. Commit using Conventional Commits format
git add .
git commit -m "feat: add NACL configuration for Project 1"
# Types: feat | fix | docs | refactor | chore | test

# 6. Push your branch
git push origin feat/your-feature-name

# 7. Open a Pull Request on GitHub
# Compare & pull request → describe what you changed → Submit
```

### Commit Message Format

```
<type>(<scope>): <short description>

Types:
  feat      → New feature or project addition
  fix       → Bug fix or correction
  docs      → README or documentation only
  refactor  → Code/config change without new feature
  chore     → Tooling, CI, formatting
  test      → Adding or updating tests/validation steps
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

| | |
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

*If you found this project useful — for learning, reference, or interview prep — a star goes a long way in supporting open-source documentation.*

<br/>

![AWS](https://img.shields.io/badge/Built%20on-AWS-%23FF9900?style=flat-square&logo=amazon-aws&logoColor=white)
![Made in India](https://img.shields.io/badge/Made%20with%20%E2%9D%A4%EF%B8%8F%20in-India-blue?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-22c55e?style=flat-square)

*© 2026 Prem Kumar S · Krishnagiri, Tamil Nadu, India*

</div>
