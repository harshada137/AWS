
## AWS CloudFront 

### 1. What is CloudFront?

A **CDN** that delivers content with **low latency** by caching it at global **edge locations**.

**Uses:** Static websites, videos, APIs, software downloads
**Benefits:** Faster delivery, reduced origin load, DDoS protection, WAF, HTTPS

---

### 2. Edge Location vs Availability Zone

* **Edge Location:** Caches & serves content (CloudFront)
* **AZ:** Runs AWS resources like EC2, RDS (inside Regions)

---

### 3. CloudFront Origins

* **S3 bucket** (static content)
* **EC2 / ALB**
* **Custom HTTP origin**
* **MediaStore / MediaPackage**

---

### 4. Cache Behaviors

Define how requests are handled based on **path patterns** (`/api/*`, `/images/*`).

* Control origin, TTL, methods, HTTPS, compression
* Most specific rule wins

---

### 5. Price Classes

* **All:** Best performance, highest cost
* **200:** Excludes expensive regions
* **100:** Cheapest (NA + Europe)

---

### 6. TTL (Time to Live)

Controls cache duration.

* Static files → long TTL
* Dynamic content → short/no TTL
  Headers like `Cache-Control` override defaults.

---

### 7. Cache Hit vs Miss

* **Hit:** Served from edge (fast, cheap)
* **Miss:** Fetched from origin (slower, costlier)

---

### 8. HTTPS in CloudFront

* Viewer → CloudFront: HTTP, HTTPS, or Redirect to HTTPS
* CloudFront → Origin: HTTP or HTTPS
* Use **ACM cert (us-east-1)** for custom domains

---

### 9. Cache Invalidation

Removes cached objects before TTL expiry.

* First **1000 paths/month free**
* Best practice: **versioned filenames** (`style.v2.css`)

---

### 10. CloudFront vs S3 Transfer Acceleration

* **CloudFront:** Fast downloads (cached)
* **S3 Transfer Acceleration:** Fast uploads to S3

---

## Intermediate Concepts (Very Short)

### 11. OAI vs OAC

* **OAI:** Legacy, limited
* **OAC:** Recommended, supports KMS, all HTTP methods, SigV4

---

### 12. Cache Update Strategies

* Versioned files (best)
* Invalidation (emergency)
* Short TTL
* Cache-Control headers

---

### 13. Signed URLs

Secure, time-limited access to content.

* Used for **paid/private content**
* Alternative: **Signed Cookies** (multiple files)

---

### 14. CloudFront Functions vs Lambda@Edge

* **Functions:** Ultra-fast, cheap, simple logic
* **Lambda@Edge:** Complex logic, AWS SDK, higher cost

---

### 15. Geo-Blocking

* **CloudFront Geo-Restriction:** Simple, free, whole site
* **AWS WAF Geo-Match:** Advanced, path-based, paid


