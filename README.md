# AWS VPC Setup — NAT Gateway with Three Private Subnets

This guide walks you through creating a **VPC** with **one public subnet** (hosting a NAT Gateway) and **three private subnets**, allowing private instances to access the internet securely through the NAT Gateway.

---

## 1. Create the VPC

1. Go to **VPC Dashboard → Your VPCs → Create VPC**
2. Configure:
   - **Name tag:** `MyVPC`
   - **IPv4 CIDR block:** `10.0.0.0/16`
   - **IPv6 CIDR block:** None (optional)
   - **Tenancy:** Default  
3. Click **Create VPC**

---

## 2. Create Subnets

You’ll need **1 Public Subnet** and **3 Private Subnets** inside the same VPC.

### ➤ Public Subnet
1. Go to **Subnets → Create subnet**
2. Choose:
   - **VPC:** `MyVPC`
   - **Subnet name:** `Public-Subnet`
   - **Availability Zone:** e.g., `ap-south-1a`
   - **CIDR block:** `10.0.1.0/24`
3. Click **Create subnet**

### ➤ Private Subnets
Repeat the process for each:
- `Private-Subnet-1` → `10.0.2.0/24`
- `Private-Subnet-2` → `10.0.3.0/24`
- `Private-Subnet-3` → `10.0.4.0/24`

---

## 3. Create an Internet Gateway (IGW)

1. Go to **Internet Gateways → Create internet gateway**
   - **Name:** `MyIGW`
2. Click **Create internet gateway**
3. Select it → **Actions → Attach to VPC → MyVPC**

---

## 4. Configure Route Tables

You’ll need two route tables:
- **Public Route Table** → for Public Subnet  
- **Private Route Table** → for Private Subnets  

### ➤ Public Route Table
1. Go to **Route Tables → Create route table**
   - **Name:** `Public-RT`
   - **VPC:** `MyVPC`
2. Click **Create route table**
3. Open it → **Routes → Edit routes → Add route**
   - **Destination:** `0.0.0.0/0`
   - **Target:** Internet Gateway (`MyIGW`)
4. **Subnet associations → Edit → Select `Public-Subnet`**

---

## 5. Create NAT Gateway

The NAT Gateway will live in the **Public Subnet**.

1. Go to **NAT Gateways → Create NAT Gateway**
   - **Name:** `MyNATGW`
   - **Subnet:** `Public-Subnet`
   - **Elastic IP:** Allocate new Elastic IP
2. Click **Create NAT Gateway**
3. Wait until the status changes to **Available**

---

## 6. Configure Private Route Table

1. Go to **Route Tables → Create route table**
   - **Name:** `Private-RT`
   - **VPC:** `MyVPC`
2. Select the route table → **Routes → Edit routes → Add route**
   - **Destination:** `0.0.0.0/0`
   - **Target:** NAT Gateway (`MyNATGW`)
3. **Subnet associations → Edit → Select all private subnets:**
   - `Private-Subnet-1`
   - `Private-Subnet-2`
   - `Private-Subnet-3`

---

## 7. Verify Setup

| Component | Subnet | Internet Access | Route Target |
|------------|---------|----------------|---------------|
| Public EC2 | Public-Subnet | Direct | Internet Gateway |
| Private EC2 | Private-Subnet-1 / 2 / 3 | Via NAT | NAT Gateway |

---

## 8. Architecture Overview

- VPC (10.0.0.0/16)

  - Public Subnet (10.0.1.0/24)
    - `Internet Gateway (IGW)`
    - `NAT Gateway (MyNATGW)`



  - Private Subnet 1 (10.0.2.0/24)
  - Private Subnet 2 (10.0.3.0/24)
  - Private Subnet 3 (10.0.4.0/24)

