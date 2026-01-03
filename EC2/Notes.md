

# AWS EC2 (Elastic Compute Cloud) – Detailed Notes

---

## 1. What is AWS EC2?

**Amazon EC2** is a web service that provides **resizable virtual servers (instances)** in the cloud.
It allows you to **run applications** without buying or maintaining physical hardware.

### Key Purpose

* Host applications
* Run backend services
* Perform compute-heavy workloads
* Create scalable cloud infrastructure

---

## 2. Core Components of EC2

### 2.1 EC2 Instance

A **virtual machine** running on AWS infrastructure.

Each instance includes:

* CPU (vCPU)
* Memory (RAM)
* Storage (EBS / Instance Store)
* Network interface
* Operating system

---

### 2.2 Amazon Machine Image (AMI)

An **AMI** is a **template** used to launch an EC2 instance.

#### AMI Contains:

* Operating System (Amazon Linux, Ubuntu, RHEL, Windows)
* Application software
* Configuration settings

#### Types of AMIs:

* AWS provided AMIs
* Marketplace AMIs
* Custom AMIs (created by users)

---

### 2.3 Instance Types

Defines **hardware configuration** of an EC2 instance.

Format:

```
<family><generation>.<size>
Example: t3.micro
```

#### Common Instance Families:

| Family | Purpose                   |
| ------ | ------------------------- |
| t      | General purpose, low-cost |
| m      | Balanced compute & memory |
| c      | Compute optimized         |
| r      | Memory optimized          |
| i      | Storage optimized         |
| g / p  | GPU workloads             |

---

## 3. EC2 Instance Lifecycle

### Instance States:

1. **Pending** – instance is starting
2. **Running** – instance is active
3. **Stopping** – instance is shutting down
4. **Stopped** – instance is off (EBS remains)
5. **Terminated** – instance deleted permanently

⚠️ **Terminated instances cannot be recovered**

---

## 4. EC2 Storage Options

### 4.1 Elastic Block Store (EBS)

Persistent block storage attached to EC2.

#### Key Points:

* Data persists even after instance stop
* Can be detached and attached to another instance
* Supports snapshots

#### EBS Volume Types:

| Type | Use Case                   |
| ---- | -------------------------- |
| gp3  | General purpose SSD        |
| io2  | High-performance databases |
| st1  | Throughput optimized HDD   |
| sc1  | Cold HDD                   |

---

### 4.2 Instance Store

* Temporary storage
* Data lost on stop/terminate
* Very fast I/O

---

## 5. EC2 Networking

### 5.1 Elastic Network Interface (ENI)

A virtual network card attached to EC2.

Includes:

* Private IP
* Public IP (optional)
* MAC address
* Security groups

---

### 5.2 IP Addresses

| Type       | Description                          |
| ---------- | ------------------------------------ |
| Private IP | Internal communication               |
| Public IP  | Internet access (changes on restart) |
| Elastic IP | Static public IP                     |

---

## 6. Security in EC2

### 6.1 Key Pairs

Used for **SSH authentication**.

* Public key → stored in EC2
* Private key → kept by user

```bash
ssh -i key.pem ec2-user@<public-ip>
```

---

### 6.2 Security Groups

Acts as a **virtual firewall**.

#### Characteristics:

* Stateful
* Allow rules only
* Applied at instance level

Example:

* Allow SSH (22)
* Allow HTTP (80)
* Allow HTTPS (443)

---

### 6.3 IAM Role for EC2

Allows EC2 to access AWS services **without credentials**.

Example:

* EC2 accessing S3
* EC2 writing logs to CloudWatch

---

## 7. EC2 Pricing Models

| Model              | Use Case                      |
| ------------------ | ----------------------------- |
| On-Demand          | Short-term workloads          |
| Reserved Instances | Long-term (1–3 years)         |
| Savings Plans      | Flexible commitment           |
| Spot Instances     | Cost-effective, interruptible |
| Dedicated Hosts    | Compliance requirements       |

---

## 8. Auto Scaling & High Availability

### Auto Scaling Group (ASG)

* Automatically increases/decreases instances
* Based on metrics (CPU, memory, traffic)

### Load Balancer

Distributes traffic across multiple EC2 instances.

Types:

* Application Load Balancer (ALB)
* Network Load Balancer (NLB)

---

## 9. EC2 Monitoring

### Amazon CloudWatch

Monitors:

* CPU utilization
* Network traffic
* Disk I/O
* Memory (custom metric)

Supports:

* Alarms
* Logs
* Dashboards

---

## 10. EC2 User Data

Used to run scripts **at instance launch**.

Example:

```bash
#!/bin/bash
yum update -y
yum install nginx -y
systemctl start nginx
```

---

## 11. EC2 Backup & Recovery

### Snapshots

* Point-in-time backup of EBS volumes
* Stored in S3 (managed by AWS)
* Incremental backups

---

## 12. EC2 Placement Options

### Placement Groups

Controls instance placement.

Types:

* Cluster (low latency)
* Spread (high availability)
* Partition (large-scale apps)

---

## 13. Common EC2 Use Cases

* Web hosting
* Application servers
* CI/CD runners
* Dev/Test environments
* Backend APIs
* Container hosts (Docker)

---

## 14. Best Practices

✔ Use IAM roles instead of access keys
✔ Use security groups with least privilege
✔ Enable CloudWatch monitoring
✔ Use Auto Scaling for production
✔ Regular EBS snapshots
✔ Use Elastic IP carefully

---

## 15. EC2 vs Traditional Server

| Feature      | EC2           | Physical Server |
| ------------ | ------------- | --------------- |
| Scalability  | Instant       | Slow            |
| Cost         | Pay-as-you-go | Fixed           |
| Maintenance  | AWS managed   | Manual          |
| Availability | High          | Limited         |

---

