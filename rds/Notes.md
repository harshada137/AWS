# Amazon RDS (Relational Database Service) — Notes

## Overview

Amazon RDS is a fully managed relational database service that simplifies database setup, operation, and scaling. It supports multiple database engines such as **MySQL, PostgreSQL, MariaDB, Oracle, and SQL Server**.

---

## 1. RDS Features

* **Managed Service**: Automatic backups, patching, and updates
* **Scalability**: Vertical scaling (instance size) and read replicas
* **High Availability**: Multi-AZ deployments
* **Security**: Encryption at rest (KMS), encryption in transit (SSL/TLS), IAM integration
* **Monitoring**: CloudWatch metrics and enhanced monitoring

---

## 2. RDS Security

* Uses **VPC security groups** for network access control
* Supports **IAM database authentication**
* Supports **encryption at rest** using AWS KMS
* **SSL connections** enforce encryption in transit

---

## 3. RDS Versioning & Backups

* **Automated Backups**: Retain backups for a specified period (up to 35 days)
* **Manual Snapshots**: User-initiated backups
* **Point-in-Time Recovery**: Restore to any time within retention window

---

## 4. RDS Replication

* **Read Replicas**: Offload read traffic
* **Cross-Region Read Replicas**: Disaster recovery & low-latency access
* **Multi-AZ**: Synchronous replication for high availability

---

## 5. RDS Storage Types

* **General Purpose (SSD)**: Balance between price and performance
* **Provisioned IOPS (SSD)**: High-performance workloads
* **Magnetic Storage**: Legacy workloads (deprecated)

---

## 6. RDS Maintenance & Monitoring

* **Automatic Patching**: Keeps DB engine updated
* **Enhanced Monitoring**: OS-level metrics
* **Event Subscriptions**: Notifications for DB events

---

## Summary

* RDS provides **managed, scalable, and secure relational databases**
* Supports **high availability** and **replication**
* Backup and restore mechanisms ensure **data safety**
* Security and monitoring are built-in for enterprise workloads

---

**End of Notes**
