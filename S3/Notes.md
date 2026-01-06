# Amazon S3 (Simple Storage Service) — Notes

## Overview

Amazon S3 is an object storage service that offers scalability, data availability, security, and performance. It stores data as **objects** within **buckets** and is commonly used for backups, static websites, data lakes, logs, and application assets.

---

## 1. S3 Policies

### What are S3 Policies?

S3 bucket policies are **resource-based IAM policies** written in JSON that define who can access a bucket and what actions they can perform.

### Key Points

* Applied **at bucket level**
* Written in **JSON format**
* Evaluate permissions using **Allow / Deny**
* Explicit **Deny always wins**

### Common Use Cases

* Allow public read access for static website hosting
* Grant cross-account access
* Restrict access based on IP address or VPC endpoint

### Example Policy (Read-only access)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

### Policy Components

* **Effect**: Allow / Deny
* **Principal**: Who gets access
* **Action**: What actions are allowed/denied
* **Resource**: Bucket or objects
* **Condition** (optional)

---

## 2. S3 Versioning

### What is Versioning?

S3 Versioning keeps **multiple versions of an object** in the same bucket, protecting against accidental deletion or overwrite.

### Key Points

* Enabled **per bucket**
* Once enabled, **cannot be disabled** (only suspended)
* Each object version has a **unique version ID**

### Benefits

* Data protection
* Easy rollback
* Recovery from accidental deletes

### Object Delete Behavior

* Delete without version ID → adds a **delete marker**
* Delete with version ID → permanently deletes that version

### Use Cases

* Backup & recovery
* Compliance and auditing

---

## 3. S3 Replication

### What is S3 Replication?

S3 Replication automatically copies objects from one bucket to another.

### Types of Replication

1. **CRR (Cross-Region Replication)**
2. **SRR (Same-Region Replication)**

### Requirements

* Versioning must be **enabled on both buckets**
* IAM role with replication permissions

### Key Features

* Replicates new objects only
* Can replicate delete markers (optional)
* Supports cross-account replication

### Use Cases

* Disaster recovery
* Compliance
* Low-latency access

---

## 4. S3 Snow Family

### What is AWS Snow Family?

AWS Snow Family provides **physical devices** to transfer large amounts of data into and out of AWS when network transfer is slow or costly.

### Members of Snow Family

#### 1. Snowcone

* Small and portable
* Up to 8 TB storage
* Used for edge computing

#### 2. Snowball Edge

* 50–80 TB storage
* Supports compute + storage
* Used for large migrations

#### 3. Snowmobile

* Exabyte-scale (up to 100 PB)
* Truck-based data transfer
* Used for massive data center migrations

### Use Cases

* Data center migration
* Disaster recovery
* Edge processing

---

## 5. S3 Storage Gateway

### What is Storage Gateway?

AWS Storage Gateway connects **on-premises environments** to AWS cloud storage.

### Types of Storage Gateway

#### 1. File Gateway

* Presents S3 as **NFS/SMB** file share
* Files stored as objects in S3

#### 2. Volume Gateway

* Block storage for applications
* Stored as EBS snapshots in AWS

#### 3. Tape Gateway

* Virtual tape library
* Replaces physical tape backups

### Benefits

* Hybrid cloud storage
* Low latency access
* Seamless integration

### Use Cases

* Hybrid backup
* Disaster recovery
* Legacy application support

---

## Summary

* **S3 Policies** control access
* **Versioning** protects data
* **Replication** improves availability
* **Snow Family** enables offline data transfer
* **Storage Gateway** enables hybrid storage

---

**End of Notes**
