# Application Load Balancer (ALB) Creation Guide (AWS)

This document explains **how to create an Application Load Balancer (ALB) in AWS** using the **AWS Management Console**.

---

## 📌 Prerequisites

Before creating an ALB, ensure the following:

* AWS Account
* At least **2 EC2 instances**
* EC2 instances are in the **same VPC**
* A web server (Apache/Nginx) is running on **port 80 or 443**
* EC2 security group allows inbound traffic on application port

---

## 🧭 Step 1: Navigate to Load Balancers

1. Log in to **AWS Management Console**
2. Go to **EC2 Dashboard**
3. From the left sidebar, click **Load Balancers**
4. Click **Create Load Balancer**

---

## ⚙️ Step 2: Choose Load Balancer Type

Select:

* **Application Load Balancer**

Click **Create**

---

## 🧱 Step 3: Configure Load Balancer

### Basic Configuration

* **Name**: `my-alb`
* **Scheme**:

  * Internet-facing (public access)
  * Internal (private access)
* **IP address type**: IPv4

---

## 🌐 Step 4: Network Mapping

* Select **VPC**
* Choose **at least 2 Availability Zones**
* Select one subnet per AZ

---

## 🔐 Step 5: Configure Security Group

* Select an existing security group or create a new one
* Allow inbound rules:

```
HTTP  | Port 80  | Source: 0.0.0.0/0
HTTPS | Port 443 | Source: 0.0.0.0/0 (optional)
```

---

## 🎯 Step 6: Create Target Group

Click **Create target group**

### Target Group Settings

* **Target type**: Instance
* **Name**: `alb-target-group`
* **Protocol**: HTTP
* **Port**: 80
* **VPC**: Same as EC2

### Health Check Configuration

* Protocol: HTTP
* Path: `/`

Click **Next**

---

## 🖥️ Step 7: Register Targets

* Select EC2 instances
* Click **Include as pending**
* Click **Create target group**

---

## 🔁 Step 8: Configure Listener & Routing

* **Listener**:

  * Protocol: HTTP
  * Port: 80

* **Default Action**:

  * Forward to created target group

Click **Create Load Balancer**

---

## ✅ Step 9: Verify ALB

1. Wait for **Load Balancer state = Active**
2. Copy **ALB DNS name**
3. Open DNS in browser

Example:

```
my-alb-123456.ap-south-1.elb.amazonaws.com
```

If configured correctly, the application will load successfully.

---

## 🔍 Troubleshooting Checklist

* EC2 instances are **healthy** in target group
* EC2 security group allows traffic from **ALB security group**
* Web server is running on EC2
* Correct port and protocol configured

---

## 🏗️ Architecture Flow

```
User → Application Load Balancer → Target Group → EC2 Instances
```

---

## 📘 Notes

* ALB works at **Layer 7 (HTTP/HTTPS)**
* Supports **path-based routing**, **host-based routing**, and **SSL termination**

---

✅ **ALB setup completed successfully**

---

### Author

Harshada
