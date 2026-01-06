# How to Create an S3 Bucket (Step-by-Step)


## 🔹 Method 1: Using AWS Management Console

### Step 1: Login to AWS Console

* Go to AWS Management Console
* Search for **S3**

---

### Step 2: Create Bucket

* Click **Create bucket**
* Enter a **globally unique bucket name**
* Select the **AWS region**

---

### Step 3: Configure Object Ownership

* Choose **ACLs disabled (recommended)**
* Bucket owner enforced

---

### Step 4: Block Public Access

* Keep **Block all public access** enabled (recommended)
* Modify only if hosting a public website

---

### Step 5: Bucket Versioning (Optional but Recommended)

* Enable versioning to protect objects

---

### Step 6: Encryption

* Enable **Server-side encryption (SSE-S3)**
* AWS manages encryption keys

---

### Step 7: Create Bucket

* Review settings
* Click **Create bucket**

---

## 🔹 Method 2: Using AWS CLI

```bash
aws s3 mb s3://my-unique-bucket-name --region ap-south-1
```

---

## 🔹 Verify Bucket Creation

* Go to S3 Dashboard
* Bucket status should be **Active**

---

## 🔐 Best Practices

* Enable versioning
* Enable encryption
* Use bucket policies instead of ACLs
* Apply least privilege access

---

## 🎯 Common Mistakes

* Non-unique bucket name
* Public access enabled unintentionally
* Versioning not enabled

---

**End of Guide**
