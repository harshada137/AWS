# Create an Amazon RDS Instance (AWS Console)

This guide explains **how to create an Amazon RDS database instance** using the **AWS Management Console**.

---

## Prerequisites

* An active AWS account
* IAM user with permissions: `AmazonRDSFullAccess`
* Basic understanding of databases (MySQL / PostgreSQL)

---

## Step-by-Step: Create RDS Instance

### 1. Sign in to AWS Console

* Go to **[https://aws.amazon.com/console](https://aws.amazon.com/console)**
* Sign in and select your preferred **AWS Region**

---

### 2. Open RDS Service

* Search for **RDS** in the AWS services search bar
* Click **Amazon RDS**

---

### 3. Create Database

* Click **Create database**

#### Choose creation method

* Select **Standard create**

---

### 4. Engine Options

Choose a database engine:

* MySQL
* PostgreSQL
* MariaDB
* Oracle
* SQL Server

👉 *Recommended for beginners:* **MySQL** or **PostgreSQL**

---

### 5. Templates

Select a template:

* **Production** – High availability, backups enabled
* **Dev/Test** – Lower cost, basic setup
* **Free tier** – Eligible for AWS Free Tier

👉 Choose **Free tier** (if applicable)

---

### 6. DB Instance Settings

* **DB instance identifier**: `my-rds-db`
* **Master username**: `admin`
* **Master password**: (set a strong password)

---

### 7. Instance Configuration

* **DB instance class**: `db.t3.micro` (Free tier)
* **Storage type**: General Purpose (gp2/gp3)
* **Allocated storage**: 20 GB

---

### 8. Connectivity

* **VPC**: Default VPC
* **Public access**: Yes (for testing)
* **VPC security group**: Default or create new
* **Availability Zone**: No preference

⚠️ Ensure **port 3306 (MySQL)** or **5432 (PostgreSQL)** is allowed in the security group.

---

### 9. Database Authentication

* Authentication method: **Password authentication**

---

### 10. Additional Configuration

(Optional but recommended)

* Enable **Automated backups**
* Backup retention: 7 days
* Enable **Deletion protection** (for production)

---

### 11. Create Database

* Review all settings
* Click **Create database**

⏳ RDS creation may take **5–10 minutes**

---

## Verify RDS Instance

* Go to **RDS → Databases**
* Status should be **Available**
* Note the **Endpoint** (used to connect)

Example:

```
my-rds-db.cle123xyz.ap-south-1.rds.amazonaws.com
```

---

## Connect to RDS (MySQL Example)

```bash
mysql -h <endpoint> -u admin -p
```

---

## Important Notes

* RDS **uses Security Groups**, not key pairs
* You cannot SSH into RDS
* Always restrict public access in production

---

## Cleanup (Avoid Charges)

* Go to **RDS → Databases**
* Select the instance → **Actions → Delete**
* Disable final snapshot (only for testing)

---

## Summary

* RDS is a managed relational database service
* Easy to create via AWS Console
* Supports multiple database engines
* Secure, scalable, and production-ready

---

✅ **RDS creation completed successfully**
