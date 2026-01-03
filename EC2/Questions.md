

# AWS EC2 Interview Questions with Detailed Answers

---

## 1️⃣ What is Amazon EC2?

**Answer:**
Amazon Elastic Compute Cloud (EC2) is an AWS service that provides **scalable virtual servers** called **instances**. It allows users to run applications in the cloud without managing physical hardware.

EC2 provides:

* Virtual compute capacity
* Full control over OS and software
* Pay-as-you-go pricing

It is commonly used for web servers, application servers, backend services, and DevOps workloads.

---

## 2️⃣ What is an EC2 instance?

**Answer:**
An EC2 instance is a **virtual machine** running in AWS.
It includes:

* vCPU (processing power)
* RAM (memory)
* Storage (EBS or instance store)
* Networking capabilities

Each instance runs an operating system and can host applications just like a physical server.

---

## 3️⃣ What is an AMI?

**Answer:**
An **Amazon Machine Image (AMI)** is a template used to launch EC2 instances.

An AMI contains:

* Operating system
* Pre-installed software (optional)
* Configuration settings

Types of AMIs:

* AWS-provided AMIs
* Marketplace AMIs
* Custom AMIs created by users

AMI ensures consistency when launching multiple instances.

---

## 4️⃣ What are EC2 instance types?

**Answer:**
Instance types define the **hardware configuration** of an EC2 instance.

They specify:

* Number of vCPUs
* Amount of RAM
* Network performance
* Storage capability

Example:

```
t3.micro
```

Instance families:

* **t** – General purpose
* **c** – Compute optimized
* **m** – Balanced workloads
* **r** – Memory optimized

---

## 5️⃣ What is the EC2 instance lifecycle?

**Answer:**
The lifecycle defines different states of an EC2 instance:

1. **Pending** – instance is starting
2. **Running** – instance is active
3. **Stopping** – shutting down
4. **Stopped** – powered off, storage remains
5. **Terminated** – instance permanently deleted

Once terminated, an instance **cannot be recovered**.

---

## 6️⃣ Difference between Stop and Terminate?

**Answer:**

| Stop                  | Terminate                     |
| --------------------- | ----------------------------- |
| Instance is shut down | Instance is deleted           |
| EBS data is preserved | EBS data is deleted (default) |
| Can be restarted      | Cannot be restarted           |
| No compute charges    | No charges                    |

Stopping is temporary; termination is permanent.

---

## 7️⃣ What is an EBS volume?

**Answer:**
Elastic Block Store (EBS) is **persistent block storage** for EC2 instances.

Key features:

* Data persists after instance stop
* Can be detached and attached to another instance
* Supports snapshots (backups)

Used for operating systems, databases, and application data.

---

## 8️⃣ What happens to data when EC2 is terminated?

**Answer:**

* **EBS root volume** → deleted by default
* **Additional EBS volumes** → remain if configured
* **Instance store data** → lost permanently

To preserve data, snapshots should be taken before termination.

---

## 9️⃣ What is a key pair in EC2?

**Answer:**
A key pair is used for **secure authentication** to an EC2 instance.

It consists of:

* **Public key** → stored in EC2
* **Private key** → downloaded by the user

Used mainly for SSH access to Linux instances.

---

## 1️⃣0️⃣ What is a security group?

**Answer:**
A security group acts as a **virtual firewall** for EC2 instances.

Characteristics:

* Stateful
* Allows only inbound/outbound rules
* Applied at instance level

Example:

* Allow SSH on port 22
* Allow HTTP on port 80

---

## 1️⃣1️⃣ Is a security group stateful or stateless?

**Answer:**
Security groups are **stateful**.

This means:

* If inbound traffic is allowed, the response is automatically allowed
* No need to create separate outbound rules for responses

---

## 1️⃣2️⃣ What is the difference between Public IP and Elastic IP?

**Answer:**

| Public IP              | Elastic IP         |
| ---------------------- | ------------------ |
| Changes on stop/start  | Static             |
| Assigned automatically | Manually allocated |
| Short-term use         | Long-term use      |

Elastic IP is preferred for production systems.

---

## 1️⃣3️⃣ What is user data in EC2?

**Answer:**
User data is a **startup script** executed when an EC2 instance launches.

Common uses:

* Install software
* Configure services
* Start applications

Runs only **once at launch** by default.

---

## 1️⃣4️⃣ Can you change instance type after launch?

**Answer:**
Yes, but the instance must be **stopped** first.

Steps:

1. Stop instance
2. Change instance type
3. Start instance

Not possible for instances using instance store as root volume.

---

## 1️⃣5️⃣ What are EC2 pricing models?

**Answer:**
EC2 supports multiple pricing options:

* **On-Demand** – pay per hour/second
* **Reserved Instances** – long-term commitment
* **Spot Instances** – low cost, interruptible
* **Dedicated Hosts** – compliance needs

---

## 1️⃣6️⃣ What is an Elastic Network Interface (ENI)?

**Answer:**
ENI is a virtual network card attached to EC2.

It includes:

* Private IP
* Public IP (optional)
* MAC address
* Security groups

Multiple ENIs can be attached to one instance.

---

## 1️⃣7️⃣ What is monitoring in EC2?

**Answer:**
EC2 monitoring is done using **Amazon CloudWatch**.

Metrics include:

* CPU utilization
* Network traffic
* Disk activity

Used for performance analysis and alerts.

---

## 1️⃣8️⃣ What is an EC2 placement group?

**Answer:**
Placement groups control how EC2 instances are placed on hardware.

Types:

* **Cluster** – low latency
* **Spread** – high availability
* **Partition** – large-scale systems

---

## 1️⃣9️⃣ How do you secure an EC2 instance?

**Answer:**
EC2 security is achieved by:

* Using security groups
* Restricting SSH access
* Using key pairs
* Regular patching
* Monitoring activity

---

## 2️⃣0️⃣ Why is EC2 important in AWS?

**Answer:**
EC2 is the **core compute service** in AWS.

It enables:

* Application hosting
* DevOps pipelines
* Scalable infrastructure
* Cost-efficient computing

Most AWS architectures depend on EC2 directly or indirectly.

---
