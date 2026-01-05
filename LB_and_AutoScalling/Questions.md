# Load Balancing & Auto Scaling – Detailed Answers

---

## Load Balancing – Answers

### 1. What is load balancing?
Load balancing is the technique of distributing incoming application traffic across multiple servers or instances to ensure high availability, reliability, and performance.

---

### 2. Why do we use a load balancer?
To prevent overloading a single server, improve response time, provide fault tolerance, and ensure continuous availability.

---

### 3. What is a health check?
A health check is a periodic test performed by a load balancer to verify whether an instance is healthy and capable of handling traffic.

---

### 4. Difference between ALB and NLB?
| ALB | NLB |
|----|----|
| Layer 7 | Layer 4 |
| HTTP/HTTPS | TCP/UDP |
| Path & host routing | Ultra-low latency |

---

### 5. What happens when an instance becomes unhealthy?
The load balancer stops sending traffic to it and routes traffic only to healthy instances.

---

### 6. Difference between Layer 4 and Layer 7 load balancing?
- **Layer 4** works at transport level (IP + port)
- **Layer 7** understands HTTP headers, URLs, and paths

---

### 7. What is sticky session?
Sticky session ensures a user is always routed to the same backend instance during a session.

---

### 8. What is path-based routing?
Routing traffic based on URL paths, e.g. `/login`, `/api`.

---

### 9. What is cross-zone load balancing?
Traffic is evenly distributed across instances in all availability zones.

---

### 10. Can a load balancer work without Auto Scaling?
Yes, but scaling must be handled manually.

---

### 11. How does ALB handle high traffic?
By distributing requests across multiple targets and scaling with Auto Scaling Groups.

---

### 12. How does NLB achieve low latency?
By operating at Layer 4 and using static IPs.

---

### 13. How do you secure a load balancer?
- HTTPS with SSL certificates
- Security groups
- AWS WAF
- IAM policies

---

### 14. What is a target group?
A logical group of instances registered with a load balancer.

---

### 15. How do you handle sudden traffic spikes?
Using ALB + Auto Scaling with target tracking policies.

---

## Auto Scaling – Answers

### 16. What is Auto Scaling?
Auto Scaling automatically increases or decreases EC2 instances based on demand.

---

### 17. What is an Auto Scaling Group?
A collection of EC2 instances managed together for scaling and health.

---

### 18. What is desired capacity?
The number of instances Auto Scaling tries to maintain.

---

### 19. Can Auto Scaling replace failed instances?
Yes, it automatically launches new instances.

---

### 20. What are scaling policies?
Rules that define when and how scaling occurs.

---

### 21. Difference between manual and dynamic scaling?
Manual requires user action; dynamic responds automatically to metrics.

---

### 22. What metrics trigger Auto Scaling?
CPU utilization, memory usage, network traffic.

---

### 23. What is cooldown period?
Time Auto Scaling waits before executing another scaling action.

---

### 24. What is scheduled scaling?
Scaling based on a predefined time schedule.

---

### 25. How does Auto Scaling reduce cost?
By terminating unused instances during low traffic.

---

### 26. What is target tracking scaling?
Maintains a target metric value like CPU = 60%.

---

### 27. What happens when ASG reaches max capacity?
No new instances are launched even if demand increases.

---

### 28. What is predictive scaling?
Uses historical data to forecast future demand.

---

### 29. How does Auto Scaling integrate with ALB?
ALB automatically registers new instances added by ASG.

---

### 30. How do you design a highly available ASG?
- Multiple AZs
- Health checks
- ALB integration
- Proper scaling policies

---

## Scenario-Based Answers

### 31. Traffic increases 5x suddenly. What happens?
Auto Scaling launches new instances and ALB distributes traffic.

---

### 32. One EC2 instance crashes. Explain the flow.
Health check fails → traffic stopped → ASG launches new instance.

---

### 33. How do you achieve zero downtime during scaling?
Use ALB, health checks, and rolling scaling.

---

### 34. How do you control cost?
Set proper min/max capacity and use target tracking.

---

### 35. Real-world use case?
E-commerce website handling flash sales using ALB + ASG.
