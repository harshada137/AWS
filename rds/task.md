# AWS RDS & EC2 Practical Setup

This README summarizes the practical task of creating an **RDS MySQL instance** and connecting it to a **Dockerized Node.js app** running on an EC2 instance.

---

## 1️⃣ RDS MySQL Instance Setup

**Steps Performed:**

1. Create a **MySQL RDS instance** using the **Free Tier**.
2. Set **username**: `admin` and a custom **password** (no special characters).
3. Set **Public Access** to **True** to allow remote connections.
4. Create a **security group** allowing **port 3306** from **0.0.0.0/0**.
5. After creation, note the **Endpoint (hostname)** for connecting from applications.

**Key Notes:**

* Free Tier keeps costs minimal.
* Public access allows local or remote connections but should be restricted in production.
* Endpoint is used as `DB_HOST` in applications.

---

## 2️⃣ EC2 Instance Setup

**Steps Performed:**

1. SSH into the EC2 instance.
2. Install Docker:

```bash
sudo yum install -y docker
sudo service docker start
sudo usermod -aG docker ec2-user
```

3. Pull Docker image of Node.js MySQL app:

```bash
docker pull philippaul/node-mysql-app:02
```

---

## 3️⃣ Running Dockerized App with RDS

**Run the Node.js app and connect to RDS:**

```bash
docker run --rm -p 80:3000 \
-e DB_HOST="<your-db-hostname>" \
-e DB_USER="admin" \
-e DB_PASSWORD="<your-db-password>" \
-d philippaul/node-mysql-app:02
```

**Connecting to MySQL directly:**

```bash
docker run -it --rm mysql:8.0 mysql -h <your-db-hostname> -u admin -p
```

**Replace placeholders:**

* `<your-db-hostname>` → RDS endpoint
* `<your-db-password>` → RDS admin password

---

## 4️⃣ Summary

* RDS MySQL instance created with **Free Tier**, public access, and open port 3306.
* EC2 instance configured with Docker and connected to RDS.
* Node.js Docker app successfully communicates with RDS.

**Best Practices:**

* For production, **restrict security group** to known IPs.
* Consider **encryption at rest and in transit**.
* Avoid using public access unless necessary.

---


