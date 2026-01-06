# Amazon S3 Interview Questions and Answers

---

## 🔹 Beginner Level

### 1. What is Amazon S3?

Amazon S3 is an object storage service that stores data as objects in buckets and provides high durability, scalability, and availability.

---

### 2. What is a bucket in S3?

A bucket is a container for storing objects in S3. Each bucket has a globally unique name.

---

### 3. What are S3 objects?

Objects are the actual data stored in S3. Each object consists of data, metadata, and a unique key.

---

## 🔹 Intermediate Level

### 4. What is an S3 bucket policy?

A bucket policy is a resource-based policy that defines permissions for users and services at the bucket level using JSON format.

---

### 5. Explain S3 Versioning.

S3 Versioning keeps multiple versions of an object in a bucket to protect against accidental deletion or overwrite.

---

### 6. What happens when you delete a versioned object?

A delete marker is created instead of permanently deleting the object unless a specific version ID is deleted.

---

## 🔹 Advanced Level

### 7. What is S3 Replication?

S3 Replication automatically copies objects from one bucket to another bucket either in the same region (SRR) or different region (CRR).

---

### 8. What are the prerequisites for S3 replication?

* Versioning enabled on source and destination buckets
* IAM role with replication permissions

---

### 9. What is AWS Snow Family?

AWS Snow Family is a set of physical devices used to migrate large volumes of data to AWS when network transfer is not feasible.

---

### 10. Explain S3 Storage Gateway.

S3 Storage Gateway is a hybrid storage service that connects on-premises environments to AWS cloud storage using file, volume, or tape gateways.

---

## 🔹 Scenario-Based

### 11. How do you protect data from accidental deletion in S3?

By enabling versioning and optionally MFA Delete.

---

### 12. How do you replicate data to another AWS account?

Enable versioning, configure cross-account IAM roles, and set up S3 replication rules.

---

## ⭐ Interview Tip

Always mention **security, durability (11 9s), and use cases** when answering S3 questions.

---

**End of Questions**


