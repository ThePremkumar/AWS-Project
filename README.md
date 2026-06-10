# EC2 Security Group & NACL Student Projects (Public + Private Subnet)

These projects are perfect for AWS Solution Architect students because they learn Security Groups, NACLs, Route Tables, Internet Gateway, NAT Gateway, Public & Private Subnets together.

## [Project 1: Public Web Server Access Control](https://github.com/ThePremkumar/AWS-Project/tree/main/Project%201%20-%20Public%20Web%20Server%20Access)

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

## [Project 2: NACL Blocking Website](https://github.com/ThePremkumar/AWS-Project/tree/main/Project%202%20-%20NACL%20Blocking%20Website)

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

## [Project 3: Bastion Host to Private Server](https://github.com/ThePremkumar/AWS-Project/tree/main/Project%203%20-%20Bastion%20Host%20to%20Private%20Server)

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

## [Project 4: Public Web Server + Private Database](https://github.com/ThePremkumar/AWS-Project/tree/main/Project%204%20-%20Public%20Web%20Server%20+%20Private%20Database)

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

## [Project 5: Security Group Stateful Demo](https://github.com/ThePremkumar/AWS-Project/tree/main/Project%205%20-%20Security%20Group%20Stateful%20Demo)

**Objective**
Allow: `SSH TCP 22` only.
Connect using SSH.
Observe: No outbound rule needed. Traffic still returns.

**Learning**
Security Group = Stateful
Return traffic automatically allowed.

---

## [Project 6: NACL Stateless Demo](https://github.com/ThePremkumar/AWS-Project/tree/main/Project%206%20-%20NACL%20Stateless%20Demo)

**Objective**
Allow inbound: `80`
Remove outbound ephemeral ports: `1024-65535`

**Result:**
Website fails.

**Learning**
NACL = Stateless
Return traffic requires explicit rule.

---

## [Project 7: Secure Admin Network](https://github.com/ThePremkumar/AWS-Project/tree/main/Project%207%20-%20Secure%20Admin%20Network)

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

## [Project 8: Public vs Private EC2 Challenge](https://github.com/ThePremkumar/AWS-Project/tree/main/Project%208%20-%20Public%20vs%20Private%20EC2%20Challenge)

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

## [Project 9: NAT Gateway Validation](https://github.com/ThePremkumar/AWS-Project/tree/main/Project%209%20-%20NAT%20Gateway%20Validation)

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

## [Project 10: Mini Production Architecture](https://github.com/ThePremkumar/AWS-Project/tree/main/Project%2010%20-%20Mini%20Production%20Architecture)

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

## [Final Interview Project (Best One)](https://github.com/ThePremkumar/AWS-Project/tree/main/Final%20Interview%20Project)

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
