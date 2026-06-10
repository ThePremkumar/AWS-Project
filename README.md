# EC2 Security Group & NACL Student Projects (Public + Private Subnet)

These projects are perfect for AWS Solution Architect students because they learn Security Groups, NACLs, Route Tables, Internet Gateway, NAT Gateway, Public & Private Subnets together.

## Project 1: Public Web Server Access Control

**Architecture**
```text
Internet
   │
   ▼
Security Group
   │
   ▼
Public EC2 (Apache)
```

**Objective**
* Launch EC2 in Public Subnet
* Install Apache
* Allow HTTP (80)
* Block SSH from unwanted IPs

**SG Rules**
| Type | Port | Source |
|---|---|---|
| SSH | 22 | Your IP |
| HTTP | 80 | 0.0.0.0/0 |

**NACL Rules**
| Rule | Port | Action |
|---|---|---|
| 100 | 80 | Allow |
| 110 | 22 | Allow |
| * | All | Deny |

**Learning**
✅ SG Stateful
✅ NACL Stateless
✅ Public Access

---

## Project 2: NACL Blocking Website

**Architecture**
```text
Internet
   │
   ▼
NACL (Deny Port 80)
   │
   ▼
EC2 Apache
```

**Objective**
Website works initially.
Then:
* Add DENY Rule for Port 80 in NACL

**Result:**
❌ Website not accessible

**Learning**
Students understand:
Security Group allows but NACL denies = Traffic blocked

---

## Project 3: Bastion Host to Private Server

**Architecture**
```text
Internet
   │
   ▼
Public EC2 (Bastion)
   │ SSH
   ▼
Private EC2
```

**Objective**
* Public EC2 in Public Subnet
* Private EC2 in Private Subnet
* Login to Private EC2 through Bastion

**SG Configuration**
*Bastion SG*
| Port | Source |
|---|---|
| 22 | Your IP |

*Private EC2 SG*
| Port | Source |
|---|---|
| 22 | Bastion SG |

**Learning**
✅ Private Subnet Concept
✅ SG Referencing
✅ Secure Access

---

## Project 4: Public Web Server + Private Database

**Architecture**
```text
Internet
   │
   ▼
Web EC2 (Public)
   │ 3306
   ▼
DB EC2 (Private)
```

**Objective**
* Apache on Public EC2
* MySQL on Private EC2

**SG Rules**
*Web Server*
| Port |
|---|
| 80 |
| 22 |

*Database*
| Port | Source |
|---|---|
| 3306 | from Web SG |

**Learning**
Students learn real-world 3-tier design.

---

## Project 5: Security Group Stateful Demo

**Objective**
Allow: `SSH TCP 22` only.
Connect using SSH.
Observe: No outbound rule needed. Traffic still returns.

**Learning**
Security Group = Stateful
Return traffic automatically allowed.

---

## Project 6: NACL Stateless Demo

**Objective**
Allow inbound: `80`
Remove outbound ephemeral ports: `1024-65535`

**Result:**
Website fails.

**Learning**
NACL = Stateless
Return traffic requires explicit rule.

---

## Project 7: Secure Admin Network

**Scenario**
Company Admin Team:
`192.168.1.10`
`192.168.1.20`
Only they can SSH.

**SG Rule**
`22` → Specific IP

**Learning**
Production Security Practice

---

## Project 8: Public vs Private EC2 Challenge

**Deploy**
* Public EC2: Public IP Enabled
* Private EC2: No Public IP

**Student Tasks**
1. Ping Public EC2
2. SSH Public EC2
3. SSH Private EC2 directly
4. Access Private via Bastion

**Expected Result**
| Test | Result |
|---|---|
| Public SSH | Success |
| Private SSH Direct | Fail |
| Private via Bastion | Success |

---

## Project 9: NAT Gateway Validation

**Architecture**
```text
Internet
   │
NAT Gateway
   │
Private EC2
```

**Objective**
Run: `sudo dnf update -y`
Before NAT: ❌ Fail
After NAT: ✅ Success

**Learning**
Private instances access internet without public IP.

---

## Project 10: Mini Production Architecture

**Architecture**
```text
Internet
   │
  ALB
   │
Public Web EC2
   │
Private App EC2
   │
Private DB EC2
```

**Student Tasks**
* Create VPC
* Public Subnet
* Private Subnet
* Route Tables
* SG
* NACL
* NAT Gateway

**Learning**
Complete AWS Solution Architect Design.

---

## Final Interview Project (Best One)

**VPC**
```text
VPC
├── Public Subnet
│   └── Bastion Host
│
├── Public Subnet
│   └── Web Server
│
└── Private Subnet
    └── Database Server
```

**Students must configure:**
✅ Security Groups
✅ NACLs
✅ Route Tables
✅ Internet Gateway
✅ NAT Gateway
✅ SSH via Bastion
✅ Web Access via Browser
✅ Database Access only from Web Server
