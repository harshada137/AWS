# Load Balancing & Auto Scaling – Notes 

---

## 1. Load Balancing

### What is Load Balancing?
Load Balancing is the process of distributing incoming network traffic across multiple servers or instances to ensure **high availability, reliability, and performance**.

### Why Load Balancing is Required
- Prevents server overload
- Improves application availability
- Provides fault tolerance
- Reduces response time
- Supports horizontal scaling

### Types of Load Balancing
- **Static Load Balancing**  
  Traffic distribution is predefined, no real-time health awareness.
- **Dynamic Load Balancing**  
  Traffic is routed based on real-time health and load.

### Load Balancing Algorithms
- Round Robin
- Least Connections
- Weighted Round Robin
- IP Hash

### AWS Load Balancer Types
- **Application Load Balancer (ALB)**  
  - Layer 7 (HTTP/HTTPS)  
  - Path-based and host-based routing
- **Network Load Balancer (NLB)**  
  - Layer 4 (TCP/UDP)  
  - Ultra-low latency, high throughput
- **Gateway Load Balancer (GWLB)**  
  - Used for security appliances
- **Classic Load Balancer (CLB)** *(Legacy)*

### Health Checks
- Periodically checks instance health
- Routes traffic only to healthy targets
- Removes unhealthy instances automatically

---

## 2. Auto Scaling

### What is Auto Scaling?
Auto Scaling automatically **adds or removes EC2 instances** based on traffic or load to maintain performance and reduce cost.

### Core Components
- **Launch Template / Configuration**
- **Auto Scaling Group (ASG)**
- **Scaling Policies**

### Types of Scaling
- Manual Scaling
- Dynamic Scaling
  - Target Tracking
  - Step Scaling
  - Simple Scaling
- Scheduled Scaling
- Predictive Scaling

### Scaling Metrics
- CPU Utilization
- Memory Usage
- Network In / Out
- Custom CloudWatch metrics

### Auto Scaling Group Parameters
- Minimum Capacity
- Desired Capacity
- Maximum Capacity

### Benefits of Auto Scaling
- Cost optimization
- High availability
- Automatic recovery
- Elastic infrastructure

---

## 3. Load Balancer + Auto Scaling (Together)

- Load Balancer distributes traffic
- Auto Scaling adjusts instance count
- Load Balancer auto-registers new instances
- Creates a **self-healing and scalable architecture**
