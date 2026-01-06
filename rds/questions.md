# Amazon RDS Interview Questions and Answers

---

## 🔹 Beginner Level

### 1. What is Amazon RDS?

Amazon RDS (Relational Database Service) is a fully managed service that simplifies the setup, operation, and scaling of relational databases in the cloud.

---

### 2. Which database engines are supported by RDS?

* MySQL
* PostgreSQL
* MariaDB
* Oracle
* SQL Server

---

### 3. What is a DB instance?

A DB instance is a database environment in RDS, consisting of the database engine, compute resources, and storage.

---

## 🔹 Intermediate Level

### 4. What is Multi-AZ deployment?

Multi-AZ deploys a synchronous standby replica in another Availability Zone for **high availability and automatic failover**.

---

### 5. Explain RDS automated backups.

Automated backups allow RDS to **automatically backup your DB instance** and store it in S3. Retention period can be configured up to **35 days**.

---

### 6. What are read replicas?

Read replicas provide **read-only copies** of the database to offload read traffic and improve performance. They can also be **cross-region**.

---

## 🔹 Advanced Level

### 7. How does RDS ensure security?

* VPC security groups control network access
* IAM database authentication
* Encryption at rest (KMS) and in transit (SSL/TLS)
* Automatic backups with encryption

---

### 8. How can you scale an RDS instance?

* **Vertical scaling**: Increase instance size (vCPU/RAM)
* **Horizontal scaling**: Use read replicas to distribute read load

---

### 9. How does point-in-time recovery work?

RDS uses automated backups and transaction logs to restore the database to **any time within the backup retention period**.

---

### 10. Explain RDS storage types.

* **General Purpose (SSD)**: Balanced price and performance
* **Provisioned IOPS (SSD)**: High-performance workloads
* **Magnetic**: Legacy workloads (not recommended)

---

## 🔹 Scenario-Based

### 11. How do you design a highly available RDS architecture?

* Enable Multi-AZ deployment
* Use read replicas for load distribution
* Enable automated backups and monitoring

---

### 12. How do you migrate on-premise database to RDS?

* Use AWS Database Migration Service (DMS) for minimal downtime migration
* Backup & restore for smaller datasets

---

### 13. How do you secure RDS from unauthorized access?

* Configure VPC security groups
* Enable encryption (at rest & in transit)
* Use IAM policies and roles

---

## ⭐ Interview Tip

Focus on **backup strategies, failover mechanisms, replication, security, and performance tuning**.

---

**End of Questions**
