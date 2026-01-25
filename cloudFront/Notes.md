# AWS CloudFront - Comprehensive Notes

## Table of Contents
1. [Core Concepts](#core-concepts)
2. [Security](#security)
3. [Performance Optimization](#performance-optimization)
4. [Monitoring & Troubleshooting](#monitoring--troubleshooting)

---

## Core Concepts

### What is CDN and How CloudFront Works

**Content Delivery Network (CDN)** is a geographically distributed network of servers that delivers content to users based on their geographic location. Instead of fetching content from a single origin server, users receive content from the nearest edge location, reducing latency and improving performance.

**AWS CloudFront** is Amazon's CDN service that:
- Caches content at edge locations worldwide (over 450+ Points of Presence)
- Reduces load on origin servers
- Decreases latency for end users
- Provides DDoS protection through AWS Shield Standard (automatic)
- Integrates seamlessly with other AWS services

**How it works**:
1. User requests content (e.g., image, video, HTML)
2. Request is routed to the nearest edge location
3. If content is cached and not expired, CloudFront serves it immediately (cache hit)
4. If content is not cached or expired, CloudFront fetches it from the origin (cache miss)
5. CloudFront caches the content and serves it to the user
6. Subsequent requests for the same content are served from cache

---

### Origin Servers

An **origin** is the source of the original, definitive version of your content. CloudFront supports multiple origin types:

#### 1. **Amazon S3 Bucket**
- Most common origin for static content (images, CSS, JS, videos)
- Can host entire static websites
- Best practice: Use OAC to restrict direct S3 access
- Supports S3 Transfer Acceleration for faster uploads

**Use cases**: Static websites, media files, software downloads, backup storage

#### 2. **EC2 Instance**
- Custom web server running on EC2
- Must be publicly accessible (public IP/Elastic IP)
- Security groups must allow CloudFront IP ranges
- Can run dynamic applications (Node.js, PHP, Python, etc.)

**Use cases**: Dynamic content, custom applications, legacy applications

#### 3. **Application Load Balancer (ALB)**
- Distributes traffic across multiple EC2 instances
- Provides high availability and fault tolerance
- Must be internet-facing
- Can use custom health checks

**Use cases**: Scalable web applications, microservices, high-traffic websites

#### 4. **Custom Origins**
- Any HTTP/HTTPS server accessible from the internet
- On-premises servers
- Third-party web servers
- Other cloud providers

**Use cases**: Hybrid cloud architectures, multi-cloud setups, existing infrastructure

#### 5. **MediaStore Container & MediaPackage**
- Specialized origins for video streaming
- MediaStore: Low-latency video delivery
- MediaPackage: Video packaging and origination

**Use cases**: Live streaming, video on demand (VOD)

---

### Distributions

A **distribution** is the configuration that tells CloudFront where to get your content and how to deliver it.

#### Web Distribution
- Used for static and dynamic content (websites, APIs, media)
- Supports HTTP and HTTPS
- Supports all HTTP methods (GET, POST, PUT, DELETE, etc.)
- Default and recommended distribution type
- Can have multiple origins and cache behaviors

#### RTMP Distribution (DEPRECATED)
- Used for streaming media files using Adobe Flash Media Server's RTMP protocol
- Adobe Flash is discontinued, so RTMP distributions are deprecated
- AWS recommends using Web distributions with HLS or DASH for video streaming

**Key Distribution Settings**:
- **Domain name**: Automatically assigned (e.g., d111111abcdef8.cloudfront.net)
- **Alternate domain names (CNAMEs)**: Your custom domains (e.g., cdn.example.com)
- **SSL certificate**: Default CloudFront or custom ACM certificate
- **Default root object**: File served for root URL (e.g., index.html)

---

### Edge Locations, Regional Edge Caches, and POP

#### Edge Locations
- Physical data centers where CloudFront caches content
- 450+ edge locations across 90+ cities in 48+ countries
- First point of contact for user requests
- Smaller cache size (compared to Regional Edge Caches)
- Serve the most frequently accessed content

#### Regional Edge Caches (REC)
- Larger cache locations between edge locations and origin
- 13 Regional Edge Caches globally
- Store less frequently accessed content
- Reduce load on origin servers
- Content evicted from edge locations is stored here before being purged

**Request Flow**:
1. User → Edge Location (cache hit = content delivered)
2. Edge Location → Regional Edge Cache (if not at edge)
3. Regional Edge Cache → Origin (if not in REC)

#### Point of Presence (POP)
- Refers to the physical location housing edge servers
- Includes both edge locations and Regional Edge Caches
- Each POP can have multiple edge servers

**Benefits**:
- Lower latency (content served from nearest location)
- Higher cache hit ratio (multi-tier caching)
- Reduced origin load (fewer requests reach origin)

---

### Cache Behaviors and TTL

#### Cache Behaviors
A **cache behavior** defines how CloudFront handles requests for specific URL path patterns.

**Key components**:
- **Path pattern**: URLs that match this behavior (e.g., /images/*, *.jpg)
- **Origin**: Which origin to use for this path
- **Viewer protocol policy**: HTTP/HTTPS enforcement
- **Allowed HTTP methods**: GET, POST, PUT, DELETE, etc.
- **Cache policy**: Rules for caching (TTL, headers, cookies, query strings)
- **Origin request policy**: What to forward to origin

**Example scenario**:
```
Default behavior (*):          Origin = S3, Cache for 24 hours
/api/* behavior:               Origin = ALB, No caching
/images/* behavior:            Origin = S3, Cache for 7 days
```

**Behavior precedence**: Most specific path pattern wins. Behaviors are evaluated in order, with the default (*) behavior being the catch-all.

#### Time to Live (TTL)
**TTL** determines how long CloudFront caches an object before checking the origin for updates.

**Three TTL settings**:
1. **Minimum TTL**: Shortest time to cache (default: 0 seconds)
2. **Maximum TTL**: Longest time to cache (default: 31,536,000 seconds = 1 year)
3. **Default TTL**: Used when origin doesn't specify (default: 86,400 seconds = 24 hours)

**TTL Priority Order** (highest to lowest):
1. Origin headers (Cache-Control, Expires)
2. CloudFront cache policy settings
3. Default TTL

**Cache-Control Headers** (from origin):
- `Cache-Control: max-age=3600` → Cache for 1 hour
- `Cache-Control: no-cache` → Always validate with origin
- `Cache-Control: no-store` → Don't cache at all
- `Cache-Control: public` → Can be cached by any cache
- `Cache-Control: private` → Only cache in user's browser

**Best Practices**:
- Static content (images, CSS, JS): Long TTL (7-30 days)
- Dynamic content (APIs, personalized pages): Short TTL or no caching
- Use versioned filenames (style.v2.css) to avoid cache invalidation costs

#### Cache Keys
A **cache key** uniquely identifies a cached object. By default, it's the URL, but you can include:
- Query strings (e.g., ?version=2)
- HTTP headers (e.g., Accept-Language)
- Cookies (e.g., session-id)

**Important**: Including more in the cache key reduces cache hit ratio but increases personalization.

---

### Cache Invalidations and Versioning

#### Cache Invalidation
**Invalidation** removes objects from CloudFront caches before they expire.

**When to use**:
- You've updated content and need immediate changes
- You've discovered incorrect/sensitive content in cache
- Emergency updates required

**How to invalidate**:
1. **Console**: CloudFront → Distribution → Invalidations → Create
2. **CLI**: `aws cloudfront create-invalidation --distribution-id E123456 --paths "/*"`
3. **API**: CreateInvalidation API call

**Invalidation patterns**:
- Single file: `/images/logo.png`
- Directory: `/images/*`
- All files: `/*`
- Multiple files: `/css/style.css /js/app.js`

**Limitations**:
- First 1,000 invalidation paths per month are free
- Additional invalidations cost $0.005 per path
- Invalidations can take 5-15 minutes to complete
- Maximum 3,000 invalidations in progress at once

**Wildcard invalidations**:
- `/*` counts as 1 path
- `/images/*` counts as 1 path
- Wildcards only work at the end of the path

#### Versioning (Best Practice Alternative)
Instead of invalidating, use **versioned filenames**:

```
Before: /css/style.css
After:  /css/style.v2.css  or  /css/style.css?v=2
```

**Benefits**:
- No invalidation costs
- Instant updates
- Old versions remain cached (useful for rollbacks)
- No waiting for invalidation to complete

**Implementation strategies**:
1. **Query string versioning**: `style.css?v=20250124`
2. **Filename versioning**: `style-20250124.css`
3. **Hash-based versioning**: `style-a3f5b2c.css` (build tools generate hash)

**Best practice**: Use build tools (Webpack, Gulp, etc.) to automatically version assets during deployment.

---

## Security

### Signed URLs and Signed Cookies

**Purpose**: Restrict access to content by requiring authentication/authorization.

#### Signed URLs
A **signed URL** contains authentication information in the URL itself.

**When to use**:
- Individual file access (single video, document, download)
- Users cannot modify cookies (mobile apps, email links)
- One-time access tokens

**URL structure**:
```
https://d111111abcdef8.cloudfront.net/video.mp4?
  Expires=1234567890&
  Signature=abc123...&
  Key-Pair-Id=APKA...
```

**Use cases**:
- Premium content downloads
- Time-limited file access
- Secure software distribution
- Private media streaming

**How it works**:
1. User requests access to content
2. Your application validates the user
3. Your application generates a signed URL (using CloudFront private key)
4. User accesses content using the signed URL
5. CloudFront validates the signature and expiration
6. Content is served if valid

#### Signed Cookies
**Signed cookies** store authentication information in browser cookies.

**When to use**:
- Multiple file access (entire video course, photo gallery)
- Want to avoid changing URLs
- Progressive downloads (HLS video streaming)

**Cookie structure**:
```
CloudFront-Expires: 1234567890
CloudFront-Signature: abc123...
CloudFront-Key-Pair-Id: APKA...
```

**Use cases**:
- Subscription-based content (entire course access)
- Multi-file access (all images in a gallery)
- HLS/DASH video streaming (multiple segments)

**Comparison**:

| Feature | Signed URLs | Signed Cookies |
|---------|-------------|----------------|
| Best for | Individual files | Multiple files |
| URL changes | Yes | No |
| Mobile app friendly | Yes | Limited |
| Implementation complexity | Simpler | More complex |
| Cookie support needed | No | Yes |

#### Creating Signed URLs/Cookies

**Requirements**:
1. Create a CloudFront key pair (RSA 2048-bit)
2. Add trusted signers to distribution
3. Use private key to sign URLs/cookies (keep secure!)

**Process**:
1. AWS Console → CloudFront → Key Management → Create key pair
2. Download private key (only chance to download!)
3. Store private key securely (AWS Secrets Manager recommended)
4. Add public key to distribution's trusted signers
5. Use SDK/library to generate signed URLs/cookies

**Expiration policies**:
- **Canned policy**: Simple expiration time only
- **Custom policy**: Expiration + IP restrictions + start time

---

### OAI vs OAC (Origin Access Identity vs Origin Access Control)

Both OAI and OAC restrict direct access to S3 buckets, forcing users to access content only through CloudFront.

#### Origin Access Identity (OAI) - Legacy
**OAI** is a special CloudFront user that accesses S3 on behalf of CloudFront.

**How it works**:
1. Create an OAI in CloudFront
2. Grant the OAI permission in S3 bucket policy
3. Configure CloudFront distribution to use OAI
4. S3 bucket remains private (no public access)

**S3 Bucket Policy Example**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontOAI",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::cloudfront:user/CloudFront Origin Access Identity E123ABC"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

**Limitations**:
- Only supports SSE-S3 encryption (not SSE-KMS)
- Cannot use bucket policies with conditions effectively
- Legacy feature (AWS recommends OAC)

#### Origin Access Control (OAC) - Recommended
**OAC** is the modern replacement for OAI with enhanced security and features.

**Advantages over OAI**:
- Supports **SSE-KMS** encryption
- Supports **all AWS Regions** (including opt-in regions)
- Enhanced security (uses AWS Signature Version 4)
- Better S3 bucket policy support
- Supports **HTTP PUT/POST/DELETE** methods (not just GET)

**How it works**:
1. Create OAC in CloudFront
2. Update S3 bucket policy to allow OAC
3. Configure distribution to use OAC
4. S3 bucket remains private

**S3 Bucket Policy Example**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontOAC",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::123456789012:distribution/E123ABC"
        }
      }
    }
  ]
}
```

#### Migration from OAI to OAC

**Steps**:
1. Create OAC in CloudFront console
2. Update distribution to use OAC (keep OAI temporarily)
3. Update S3 bucket policy to allow both OAI and OAC
4. Test thoroughly
5. Remove OAI from distribution
6. Remove OAI permissions from S3 bucket policy

**Best practice**: Use OAC for all new distributions. Migrate existing OAI distributions gradually.

---

### SSL/TLS Certificates with ACM

**SSL/TLS certificates** encrypt data in transit between CloudFront and users (HTTPS).

#### Certificate Options

**1. Default CloudFront Certificate**
- Domain: `*.cloudfront.net` (e.g., d111111abcdef8.cloudfront.net)
- Cost: Free
- Use case: Testing, non-production
- Limitation: Cannot use custom domain names

**2. Custom SSL Certificate (ACM)**
- Domain: Your domain (e.g., cdn.example.com, www.example.com)
- Cost: Free (ACM certificates are free)
- Use case: Production websites
- Requirement: Certificate must be in **us-east-1** region

#### Setting Up Custom SSL Certificate

**Prerequisites**:
- Own a domain name
- Access to domain's DNS settings (for validation)

**Steps**:
1. **Request certificate in ACM** (must be us-east-1):
   - Console → Certificate Manager → Request certificate
   - Enter domain names (cdn.example.com, *.example.com)
   - Choose DNS validation (recommended) or email validation
   
2. **Validate domain ownership**:
   - Add CNAME records to DNS (ACM provides values)
   - Wait for validation (5 minutes to 24 hours)
   
3. **Attach certificate to CloudFront**:
   - CloudFront → Distribution → Edit → Custom SSL certificate
   - Select your ACM certificate
   
4. **Update DNS**:
   - Create CNAME record: `cdn.example.com → d111111abcdef8.cloudfront.net`

**Viewer Protocol Policy Options**:
- **HTTP and HTTPS**: Allow both (not recommended)
- **Redirect HTTP to HTTPS**: Automatically upgrade (recommended)
- **HTTPS only**: Block HTTP requests

**Origin Protocol Policy** (CloudFront to Origin):
- **HTTP only**: Faster, use if origin is S3 or internal
- **HTTPS only**: Secure, use for public origins
- **Match viewer**: Follow user's protocol

**Security levels**:
- **TLS 1.0** (deprecated, insecure)
- **TLS 1.1** (legacy support)
- **TLS 1.2** (recommended minimum)
- **TLS 1.3** (latest, most secure)

**Best practice**: Use "TLS 1.2 as minimum" to balance security and compatibility.

---

### WAF Integration

**AWS WAF** (Web Application Firewall) protects web applications from common exploits.

**What WAF does**:
- Filters malicious requests before reaching origin
- Protects against SQL injection, XSS attacks
- Rate limiting (prevent DDoS)
- Geo-blocking at application layer
- Custom rules based on IP, headers, body, etc.

**Integration with CloudFront**:
1. Create WAF Web ACL (Access Control List)
2. Define rules (block, allow, count)
3. Attach Web ACL to CloudFront distribution
4. WAF inspects requests at edge locations

**Common WAF Rules**:
- **Rate limiting**: Block IPs making >2000 requests/5 minutes
- **Geo-match**: Block requests from specific countries
- **IP sets**: Block known malicious IPs
- **String matching**: Block requests containing SQL keywords
- **Size constraints**: Block unusually large requests

**Use cases**:
- DDoS protection (Layer 7)
- Bot management
- API rate limiting
- Content scraping prevention
- Compliance requirements (PCI-DSS, HIPAA)

**Cost**: WAF pricing is separate from CloudFront:
- $5.00 per Web ACL per month
- $1.00 per rule per month
- $0.60 per million requests

---

### Geo-Restriction

**Geo-restriction** (geo-blocking) controls which countries can access your content.

#### CloudFront Geo-Restriction
Built-in feature that blocks/allows countries at the distribution level.

**Two modes**:
1. **Whitelist**: Allow only specific countries
2. **Blacklist**: Block specific countries

**How it works**:
- CloudFront determines user's country by IP address
- If country is blocked, returns HTTP 403 Forbidden
- Applies to entire distribution (all content)

**Configuration**:
```
CloudFront → Distribution → Edit → Restrictions → 
  Enable geo-restriction → Whitelist/Blacklist → 
  Select countries
```

**Limitations**:
- All-or-nothing (cannot block specific URLs for specific countries)
- Only country-level (not state/city)
- IP-based detection (can be bypassed with VPN)

**Use cases**:
- Content licensing restrictions (movies, sports)
- Compliance requirements (GDPR, data sovereignty)
- Prevent access from high-risk countries
- Software export restrictions

#### WAF Geo-Blocking (Advanced)
More flexible than CloudFront geo-restriction:
- Block specific URL paths for specific countries
- Combine with other rules (IP, rate limiting)
- More granular control

**Example**: Block /admin/* for all countries except US.

**Best practice**: Use CloudFront geo-restriction for simple blocking, WAF for complex scenarios.

---

## Performance Optimization

### Gzip and Brotli Compression

**Compression** reduces file sizes, improving load times and reducing bandwidth costs.

#### Gzip Compression
Industry standard compression format supported by all browsers.

**How CloudFront handles it**:
1. User's browser sends header: `Accept-Encoding: gzip`
2. CloudFront checks if cached compressed version exists
3. If not, CloudFront requests from origin or compresses on-the-fly
4. Compressed file is cached and served

**Configuration**:
- CloudFront → Response Headers Policy → Enable compression
- Or: Enable in Cache Policy settings

**Compression ratios** (typical):
- HTML: 70-80% reduction
- CSS: 70-85% reduction
- JavaScript: 60-70% reduction
- JSON: 70-80% reduction
- Images (JPEG, PNG): Already compressed (skip)

#### Brotli Compression
Modern compression algorithm with better compression than gzip (10-20% smaller).

**Browser support**: All modern browsers (Chrome, Firefox, Edge, Safari)

**CloudFront support**:
- Enabled via Response Headers Policy
- Requires origin support or CloudFront automatic compression

**Configuration**:
```
Response Headers Policy:
- Brotli compression: Enabled
- Gzip compression: Enabled (fallback)
```

**Best practices**:
- Enable both Brotli and gzip (browser negotiates best)
- Don't compress images, videos, already-compressed files
- Compress text-based content: HTML, CSS, JS, JSON, XML, SVG
- Origin should send `Content-Type` header correctly

**Origin considerations**:
- **S3**: Doesn't compress automatically, upload pre-compressed files or use CloudFront compression
- **EC2/ALB**: Configure web server (nginx, Apache) to compress
- **Custom origin**: Implement compression or let CloudFront handle it

---

### HTTP/2 and HTTP/3 Support

#### HTTP/2
Modern protocol that improves performance over HTTP/1.1.

**Key features**:
- **Multiplexing**: Multiple requests over single connection (no head-of-line blocking)
- **Header compression**: Reduces overhead (HPACK)
- **Server push**: Server sends resources before requested (limited CloudFront support)
- **Binary protocol**: More efficient parsing

**CloudFront support**:
- Enabled by default for HTTPS connections
- Automatic (no configuration needed)
- Falls back to HTTP/1.1 for non-supporting browsers

**Performance benefits**:
- Faster page loads (30-50% improvement typical)
- Reduced latency
- Better mobile performance

**Requirements**:
- HTTPS only (HTTP/2 requires TLS)
- Modern browser (all current browsers support it)

#### HTTP/3 (QUIC)
Latest protocol built on QUIC (Quick UDP Internet Connections).

**Key improvements over HTTP/2**:
- Uses UDP instead of TCP (faster connection establishment)
- Better performance on lossy networks (mobile, WiFi)
- Built-in encryption (TLS 1.3)
- True multiplexing (no head-of-line blocking at transport layer)

**CloudFront support**:
- Available (opt-in)
- Enable in distribution settings
- Automatic fallback to HTTP/2 or HTTP/1.1

**Configuration**:
```
CloudFront → Distribution → Edit → 
  Supported HTTP versions → HTTP/3, HTTP/2, HTTP/1.1
```

**Best practice**: Enable HTTP/3 and HTTP/2 for best performance across all clients.

---

### Lambda@Edge vs CloudFront Functions

Both allow you to run code at CloudFront edge locations, but they have different capabilities and use cases.

#### CloudFront Functions
Lightweight JavaScript functions for simple transformations.

**Characteristics**:
- Ultra-low latency (sub-millisecond execution)
- Written in JavaScript (ECMAScript 5.1 compliant)
- Maximum 10KB code size
- Maximum 2MB memory
- Runs on every request (millions/second scale)

**Execution points**:
- **Viewer request**: After CloudFront receives request from user
- **Viewer response**: Before CloudFront returns response to user

**Use cases**:
- URL redirects and rewrites
- Header manipulation (add, modify, remove)
- A/B testing (route to different origins)
- Simple authorization (token validation)
- Normalize cache keys (lowercase URLs)

**Example**: Redirect HTTP to HTTPS
```javascript
function handler(event) {
    var request = event.request;
    if (request.headers['cloudfront-viewer-protocol'].value === 'http') {
        return {
            statusCode: 301,
            statusDescription: 'Moved Permanently',
            headers: {
                location: { value: 'https://' + request.headers.host.value + request.uri }
            }
        };
    }
    return request;
}
```

#### Lambda@Edge
Full Node.js/Python functions with AWS SDK access.

**Characteristics**:
- Higher latency (5-100ms execution)
- Node.js 20.x or Python 3.12 runtime
- Maximum 50MB code size (1MB for viewer triggers)
- Maximum 10GB memory (viewer triggers: 128MB)
- AWS SDK access (DynamoDB, S3, etc.)
- Runs in regional edge caches

**Execution points**:
- **Viewer request**: After CloudFront receives request
- **Origin request**: Before CloudFront forwards to origin
- **Origin response**: After CloudFront receives origin response
- **Viewer response**: Before CloudFront returns to user

**Use cases**:
- Dynamic content generation
- Image resizing/transformation
- Authentication/authorization (OAuth, JWT)
- A/B testing with database lookups
- Bot detection with ML models
- SEO optimization (user-agent based rendering)

**Example**: Resize images on-the-fly
```javascript
const AWS = require('aws-sdk');
const sharp = require('sharp');
const S3 = new AWS.S3();

exports.handler = async (event) => {
    const request = event.Records[0].cf.request;
    const width = request.querystring.match(/w=(\d+)/)?.[1] || 800;
    
    // Fetch original image from S3
    const original = await S3.getObject({
        Bucket: 'my-images',
        Key: request.uri
    }).promise();
    
    // Resize using sharp
    const resized = await sharp(original.Body)
        .resize(parseInt(width))
        .toBuffer();
    
    return {
        status: '200',
        body: resized.toString('base64'),
        bodyEncoding: 'base64',
        headers: {
            'content-type': [{ value: 'image/jpeg' }]
        }
    };
};
```

#### Comparison Table

| Feature | CloudFront Functions | Lambda@Edge |
|---------|---------------------|-------------|
| Runtime | JavaScript only | Node.js, Python |
| Max execution time | <1ms | 5s (viewer), 30s (origin) |
| Max code size | 10KB | 1MB (viewer), 50MB (origin) |
| Max memory | 2MB | 128MB (viewer), 10GB (origin) |
| AWS SDK access | No | Yes |
| Execution points | Viewer request/response | All 4 points |
| Pricing (per million) | $0.10 | $0.60 (request), $0.00005001 (duration) |
| Network access | No | Yes (HTTP, database) |
| Best for | Simple transformations | Complex logic, integrations |

**Choosing between them**:
- Need AWS services (DynamoDB, S3)? → Lambda@Edge
- Simple header/URL manipulation? → CloudFront Functions
- Need to call external APIs? → Lambda@Edge
- Ultra-low latency critical? → CloudFront Functions
- Complex business logic? → Lambda@Edge
- High request volume? → CloudFront Functions (cheaper)

---

### Query String and Cookie Forwarding

Determines what CloudFront includes in the cache key and forwards to the origin.

#### Query String Forwarding

**Options**:
1. **None**: Ignore query strings (better caching)
2. **All**: Forward all query strings (worse caching)
3. **Whitelist**: Forward only specific parameters

**Example scenarios**:

**Scenario 1**: Product page with tracking parameters
```
/product?id=123&utm_source=google&utm_campaign=summer
```
- Cache key should include: `id`
- Cache key should exclude: `utm_source`, `utm_campaign`
- Configuration: Whitelist `id` parameter

**Scenario 2**: API with pagination
```
/api/products?page=2&limit=20
```
- Cache key should include: `page`, `limit`
- Configuration: Whitelist `page` and `limit`

**Impact on cache hit ratio**:
- Forwarding all parameters: Low cache hit ratio (every unique URL is cached separately)
- No forwarding: High cache hit ratio (all variations cached as one)
- Whitelist: Balanced (cache variations that matter)

#### Cookie Forwarding

**Options**:
1. **None**: Don't forward cookies (better caching)
2. **All**: Forward all cookies (personalized content)
3. **Whitelist**: Forward specific cookies

**Example scenarios**:

**Scenario 1**: User authentication
```
Cookies: session-id=abc123, theme=dark, language=en
```
- Forward: `session-id` (authentication)
- Don't forward: `theme`, `language` (can be in query string or header)

**Scenario 2**: A/B testing
```
Cookies: ab-test=variant-b, user-id=456
```
- Forward: `ab-test` (different cache for each variant)
- Maybe forward: `user-id` (if content is personalized)

**Impact**:
- No cookies: Static content (images, CSS, JS)
- Whitelist cookies: Semi-dynamic content (language, currency)
- All cookies: Fully personalized content (user dashboard)

**Best practices**:
- Static assets (images, CSS, JS): No query strings, no cookies
- Dynamic APIs: Whitelist necessary parameters only
- Personalized content: Forward required cookies, consider Lambda@Edge for complex logic
- Use separate cache behaviors for different content types

---

## Monitoring & Troubleshooting

### CloudWatch Metrics Explained

CloudFront automatically publishes metrics to CloudWatch for monitoring.

#### Key Metrics

**1. Requests**
- Total number of HTTP/HTTPS requests
- Unit: Count
- Use: Monitor traffic volume, detect spikes

**2. BytesDownloaded**
- Total bytes served to users
- Unit: Bytes
- Use: Track bandwidth usage, estimate costs

**3. BytesUploaded**
- Total bytes uploaded to CloudFront (POST, PUT)
- Unit: Bytes
- Use: Monitor upload activity (file uploads, API calls)

**4. 4xxErrorRate**
- Percentage of requests with 4xx errors (client errors)
- Common codes: 403 (Forbidden), 404 (Not Found)
- Unit: Percentage
- Use: Detect broken links, unauthorized access attempts

**5. 5xxErrorRate**
- Percentage of requests with 5xx errors (server errors)
- Common codes: 502 (Bad Gateway), 503 (Service Unavailable), 504 (Gateway Timeout)
- Unit: Percentage
- Use: Detect origin issues, timeouts

**6. TotalErrorRate**
- Percentage of requests with any error (4xx + 5xx)
- Unit: Percentage
- Use: Overall health monitoring

**7. CacheHitRate** (Additional metrics, opt-in)
- Percentage of requests served from cache
- Unit: Percentage
- Target: >85% for static content
- Use: Optimize caching strategy

**8. OriginLatency** (Additional metrics, opt-in)
- Time from request to CloudFront receiving first byte from origin
- Unit: Milliseconds
- Use: Identify slow origin performance

#### Setting Up CloudWatch Alarms

**Example**: Alert on high 5xx error rate
```
Metric: 5xxErrorRate
Threshold: > 5% for 2 consecutive periods (10 minutes)
Action: Send SNS notification
```
