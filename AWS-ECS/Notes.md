# AWS ECS Complete Notes

## Table of Contents
1. [Introduction](#introduction)
2. [Core Concepts](#core-concepts)
3. [Launch Types](#launch-types)
4. [Networking](#networking)
5. [Storage Options](#storage-options)
6. [Service Discovery](#service-discovery)
7. [Load Balancing](#load-balancing)
8. [Auto Scaling](#auto-scaling)
9. [Security](#security)
10. [Monitoring & Logging](#monitoring--logging)

---

## Introduction

### What is Amazon ECS?

Amazon Elastic Container Service (ECS) is a fully managed container orchestration service that allows you to easily deploy, manage, and scale containerized applications using Docker containers.

**Key Benefits:**
- Fully managed service (no control plane to manage)
- Deep AWS integration (IAM, VPC, CloudWatch, etc.)
- Cost-effective (no additional charge for ECS, only pay for resources)
- Supports both serverless (Fargate) and EC2 launch types
- Built-in service discovery and load balancing

---

## Core Concepts

### 1. Cluster
A **cluster** is a logical grouping of tasks or services. It's like a container for your containerized applications.

- Region-specific
- Can contain multiple services and tasks
- Can use Fargate, EC2, or both simultaneously
- Free to create and use

**Example:**
```
Cluster: production-cluster
├── Service: web-app (3 tasks)
├── Service: api-service (5 tasks)
└── Service: worker-service (2 tasks)
```

### 2. Task Definition
A **task definition** is a blueprint (JSON format) that describes how to run your Docker containers.

**Key Components:**
- **Family name**: Name of the task definition
- **Container definitions**: One or more containers (max 10)
- **Task role**: IAM role for the task to access AWS services
- **Execution role**: IAM role for ECS agent to pull images, write logs
- **Network mode**: awsvpc, bridge, host, or none
- **CPU and Memory**: Resources allocated to the task
- **Volumes**: Storage configurations
- **Launch type**: Fargate or EC2

**Example Structure:**
```json
{
  "family": "my-app",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "containerDefinitions": [{
    "name": "web",
    "image": "nginx:latest",
    "portMappings": [{"containerPort": 80}]
  }]
}
```

### 3. Task
A **task** is a running instance of a task definition. Think of it as a single running copy of your application.

- Smallest deployable unit in ECS
- Can contain one or multiple containers
- Has its own lifecycle (pending, running, stopped)
- Can be run as standalone or part of a service

### 4. Service
A **service** maintains a desired number of tasks running simultaneously.

**Key Features:**
- Ensures specified number of tasks are always running
- Can register tasks with load balancers
- Supports auto-scaling
- Handles task failures and replacements
- Enables rolling deployments

**Service Types:**
- **REPLICA**: Maintains desired number of tasks across cluster
- **DAEMON**: Runs exactly one task per container instance

### 5. Container Instance (EC2 Launch Type Only)
An EC2 instance running the ECS agent, registered to a cluster.

**Components:**
- ECS agent (communicates with ECS service)
- Docker daemon
- Operating system (Amazon Linux 2, Ubuntu, etc.)

---

## Launch Types

### 1. AWS Fargate (Serverless)

**What is it?**
Serverless compute engine for containers. AWS manages all infrastructure.

**Characteristics:**
- No servers to manage
- Pay only for resources your tasks use
- Each task runs in isolation with its own kernel
- Automatic scaling of infrastructure
- Built-in security isolation

**Use Cases:**
- Microservices
- Batch processing
- Web applications
- CI/CD pipelines

**Pros:**
✅ No infrastructure management
✅ Improved security (task-level isolation)
✅ Pay-per-use pricing
✅ Faster scaling
✅ Reduced operational overhead

**Cons:**
❌ Higher cost per vCPU/GB compared to EC2
❌ Less customization
❌ Cannot use GPUs
❌ Limited to specific CPU/memory combinations

**Pricing:**
Based on vCPU and memory requested:
- vCPU: ~$0.04048 per hour
- Memory: ~$0.004445 per GB per hour

### 2. EC2 Launch Type

**What is it?**
You manage EC2 instances that host your containers.

**Characteristics:**
- Full control over EC2 instances
- You provision and manage instances
- Can use Spot instances for cost savings
- Support for all instance types (including GPU)
- Can use Reserved Instances

**Use Cases:**
- Long-running applications
- Cost-sensitive workloads
- GPU-intensive applications
- Applications requiring specific instance types
- Large-scale deployments

**Pros:**
✅ Lower cost for steady workloads
✅ Full instance control
✅ GPU support
✅ Can use Spot/Reserved instances
✅ More configuration options

**Cons:**
❌ You manage infrastructure
❌ Need to handle patching and scaling
❌ More operational overhead
❌ Capacity planning required

**Cost Optimization:**
- Use Reserved Instances for predictable workloads (up to 72% savings)
- Use Spot instances for fault-tolerant workloads (up to 90% savings)
- Right-size your instances

### 3. ECS Anywhere

**What is it?**
Run ECS tasks on your own on-premises infrastructure or VMs.

**Use Cases:**
- Hybrid cloud deployments
- Data sovereignty requirements
- Edge computing
- Gradual cloud migration

---

## Networking

### Network Modes

#### 1. awsvpc (Recommended & Required for Fargate)

**How it works:**
- Each task gets its own Elastic Network Interface (ENI)
- Task has its own private IP address
- Task uses VPC security groups
- Full VPC networking features

**Pros:**
✅ Enhanced security (task-level security groups)
✅ Better isolation
✅ Simplified networking
✅ Native VPC integration

**Cons:**
❌ Limited by ENI limits per instance
❌ Slightly higher latency

**Use Cases:**
- Microservices requiring isolation
- Security-sensitive applications
- All Fargate tasks

**Configuration:**
```json
{
  "networkMode": "awsvpc",
  "networkConfiguration": {
    "awsvpcConfiguration": {
      "subnets": ["subnet-12345"],
      "securityGroups": ["sg-67890"],
      "assignPublicIp": "ENABLED"
    }
  }
}
```

#### 2. bridge (Default for EC2)

**How it works:**
- Uses Docker's built-in virtual network
- Containers share host's network namespace
- Port mapping required

**Pros:**
✅ Simple setup
✅ Good for development

**Cons:**
❌ Port conflicts possible
❌ Less secure

#### 3. host

**How it works:**
- Container uses host's network directly
- No network isolation

**Pros:**
✅ Best performance
✅ No port mapping needed

**Cons:**
❌ No isolation
❌ Port conflicts
❌ Security concerns

---

## Storage Options

### 1. Ephemeral Storage

**What is it?**
Temporary storage that exists only during task lifetime.

**Fargate:**
- Default: 20 GB
- Maximum: 200 GB
- Configured in task definition

**EC2:**
- Depends on instance storage
- Lost when task stops

**Use Cases:**
- Temporary files
- Caches
- Scratch space

### 2. Docker Volumes

**What is it?**
Managed by Docker, persists on host.

**Types:**
- Named volumes
- Bind mounts

**Use Cases:**
- Data sharing between containers in same task
- Persistent data on EC2 instances

### 3. EFS (Elastic File System)

**What is it?**
Fully managed, scalable file system for AWS cloud.

**Features:**
- Multi-AZ support
- Automatic scaling
- Shared across multiple tasks
- Works with both Fargate and EC2

**Use Cases:**
- Shared application data
- Content management systems
- Machine learning datasets
- Logs and metrics

**Configuration:**
```json
{
  "volumes": [{
    "name": "efs-storage",
    "efsVolumeConfiguration": {
      "fileSystemId": "fs-12345",
      "transitEncryption": "ENABLED"
    }
  }],
  "mountPoints": [{
    "sourceVolume": "efs-storage",
    "containerPath": "/mnt/efs"
  }]
}
```

**Pros:**
✅ Fully managed
✅ Scales automatically
✅ Multi-AZ durability
✅ Concurrent access

**Cons:**
❌ Higher cost than EBS
❌ Slightly higher latency
❌ Performance depends on size

### 4. EBS Volumes (EC2 Only)

**What is it?**
Block storage for EC2 instances.

**Use Cases:**
- Databases
- High-performance applications

**Limitation:**
- Cannot be shared across tasks
- Single-AZ only

---

## Service Discovery

### What is Service Discovery?

Allows your services to discover and connect to each other without hardcoding IP addresses.

### How it Works

ECS integrates with **AWS Cloud Map** to provide service discovery.

**Components:**
1. **Namespace**: DNS namespace (e.g., internal.local)
2. **Service**: Represents your ECS service
3. **Service Registry**: Stores service information

### Types

#### 1. DNS-based Discovery

**How it works:**
- Tasks register with Cloud Map
- DNS queries return task IP addresses
- Automatic health checking
- TTL-based caching

**DNS Record Types:**
- **A Record**: Returns IPv4 addresses
- **SRV Record**: Returns IP + port
- **AAAA Record**: Returns IPv6 addresses

**Example:**
```
Service: api-service.internal.local
Tasks: 10.0.1.5, 10.0.1.8, 10.0.1.12
```

#### 2. API-based Discovery

**How it works:**
- Query Cloud Map API directly
- More control over discovery
- Programmatic access

### Configuration

```json
{
  "serviceRegistries": [{
    "registryArn": "arn:aws:servicediscovery:...",
    "containerName": "web",
    "containerPort": 80
  }]
}
```

### Use Cases

- Microservices architecture
- Service-to-service communication
- Dynamic environments
- Multi-tier applications

### Benefits

✅ No hardcoded IPs
✅ Automatic updates
✅ Health checking
✅ Works across AZs
✅ Simplified architecture

---

## Load Balancing

### Integration with Elastic Load Balancing

ECS integrates with three types of load balancers:

#### 1. Application Load Balancer (ALB)

**Best for:** HTTP/HTTPS traffic

**Features:**
- Path-based routing (`/api/*`, `/images/*`)
- Host-based routing (`api.example.com`, `web.example.com`)
- WebSocket support
- HTTP/2 support
- Native integration with ECS
- Dynamic port mapping

**Use Cases:**
- Web applications
- Microservices
- RESTful APIs

**Example Routing:**
```
ALB
├── /api/* → Backend Service (port 8080)
├── /admin/* → Admin Service (port 3000)
└── /* → Frontend Service (port 80)
```

#### 2. Network Load Balancer (NLB)

**Best for:** TCP/UDP traffic, high performance

**Features:**
- Extreme performance (millions of requests/sec)
- Ultra-low latency
- Static IP addresses
- Preserves client IP
- TLS termination

**Use Cases:**
- Gaming applications
- IoT applications
- Real-time communications
- TCP/UDP protocols

#### 3. Classic Load Balancer (CLB)

**Status:** Legacy (not recommended)

**Note:** Use ALB or NLB for new applications.

### Target Groups

**What are they?**
Logical grouping of targets (ECS tasks) for load balancers.

**Health Checks:**
- Path: `/health`
- Interval: 30 seconds
- Timeout: 5 seconds
- Healthy threshold: 2
- Unhealthy threshold: 2

**Configuration:**
```json
{
  "loadBalancers": [{
    "targetGroupArn": "arn:aws:elasticloadbalancing:...",
    "containerName": "web",
    "containerPort": 80
  }]
}
```

### Dynamic Port Mapping

**What is it?**
ALB automatically routes traffic to dynamically assigned ports on ECS tasks.

**How it works:**
1. Task starts with ephemeral port
2. Task registers with ALB target group
3. ALB routes traffic to correct port

**Benefits:**
✅ Multiple tasks per instance
✅ Efficient resource utilization
✅ Simplified deployment

---

## Auto Scaling

### 1. Service Auto Scaling

Automatically adjusts the number of tasks in your service.

#### Target Tracking Scaling

**Maintains a specific metric value.**

**Common Metrics:**
- **ECSServiceAverageCPUUtilization**: CPU usage (recommended: 70%)
- **ECSServiceAverageMemoryUtilization**: Memory usage (recommended: 80%)
- **ALBRequestCountPerTarget**: Requests per task

**Configuration:**
```bash
aws application-autoscaling register-scalable-target \
  --service-namespace ecs \
  --scalable-dimension ecs:service:DesiredCount \
  --resource-id service/my-cluster/my-service \
  --min-capacity 2 \
  --max-capacity 10

aws application-autoscaling put-scaling-policy \
  --policy-name cpu-scaling \
  --service-namespace ecs \
  --scalable-dimension ecs:service:DesiredCount \
  --resource-id service/my-cluster/my-service \
  --policy-type TargetTrackingScaling \
  --target-tracking-scaling-policy-configuration \
    MetricType=ECSServiceAverageCPUUtilization,TargetValue=70.0
```

#### Step Scaling

**Scales based on CloudWatch alarms.**

**Example:**
- CPU > 80% for 2 minutes → Add 2 tasks
- CPU < 30% for 5 minutes → Remove 1 task

#### Scheduled Scaling

**Scale at specific times.**

**Use Cases:**
- Peak hours (9 AM - 5 PM)
- Batch processing windows
- Maintenance windows

**Example:**
```bash
aws application-autoscaling put-scheduled-action \
  --service-namespace ecs \
  --scalable-dimension ecs:service:DesiredCount \
  --resource-id service/my-cluster/my-service \
  --scheduled-action-name scale-up-morning \
  --schedule "cron(0 8 * * ? *)" \
  --scalable-target-action MinCapacity=5,MaxCapacity=20
```

### 2. Cluster Auto Scaling (EC2 Only)

Automatically scales EC2 instances in your cluster.

**Capacity Providers:**
- Define how to scale infrastructure
- Can mix Fargate and EC2
- Support for Spot instances

**Managed Scaling:**
- Target capacity utilization (recommended: 100%)
- Automatic scale-out and scale-in

**Configuration:**
```bash
aws ecs put-cluster-capacity-providers \
  --cluster my-cluster \
  --capacity-providers FARGATE FARGATE_SPOT my-capacity-provider \
  --default-capacity-provider-strategy \
    capacityProvider=FARGATE,weight=1,base=2
```

### Best Practices

1. **Set appropriate thresholds**: Not too aggressive (flapping), not too slow
2. **Use multiple metrics**: CPU + Memory + Request Count
3. **Configure cooldown periods**: Prevent rapid scaling
4. **Set min/max limits**: Cost control
5. **Monitor scaling activities**: Ensure scaling works as expected
6. **Test scaling policies**: Simulate load

---

## Security

### 1. IAM Roles

#### Task Execution Role

**Purpose:** Allows ECS agent to perform actions on your behalf.

**Permissions:**
- Pull images from ECR
- Write logs to CloudWatch
- Retrieve secrets from Secrets Manager/Parameter Store

**Required Policy:** `AmazonECSTaskExecutionRolePolicy`

#### Task Role

**Purpose:** Allows containers in your task to access AWS services.

**Examples:**
- S3 read/write
- DynamoDB access
- SQS queue operations
- SNS publishing

**Best Practice:** One role per task, least privilege

### 2. Secrets Management

#### AWS Secrets Manager

**Use for:**
- Database credentials
- API keys
- Certificates

**In Task Definition:**
```json
{
  "secrets": [{
    "name": "DB_PASSWORD",
    "valueFrom": "arn:aws:secretsmanager:region:account:secret:db-secret"
  }]
}
```

**Benefits:**
✅ Automatic rotation
✅ Encryption at rest
✅ Audit logging
✅ Fine-grained access control

#### Systems Manager Parameter Store

**Use for:**
- Configuration values
- Feature flags
- Non-critical secrets

**In Task Definition:**
```json
{
  "secrets": [{
    "name": "API_URL",
    "valueFrom": "arn:aws:ssm:region:account:parameter/app/api-url"
  }]
}
```

**Benefits:**
✅ Free (standard parameters)
✅ Simple to use
✅ Integration with CloudFormation

### 3. Network Security

#### Security Groups

**Rules:**
- Inbound: Allow specific ports (80, 443, 8080)
- Outbound: Allow necessary connections
- Source: Restrict to known IPs/security groups

**Best Practice:**
- One security group per service
- Least privilege principle
- Use security group references instead of IPs

#### Network ACLs

**Subnet-level firewall**
- Stateless (must configure inbound + outbound)
- Additional layer of security

### 4. Image Security

#### ECR Image Scanning

**Types:**
- **Basic scanning**: CVE detection
- **Enhanced scanning**: Continuous monitoring

**Configuration:**
```bash
aws ecr put-image-scanning-configuration \
  --repository-name my-app \
  --image-scanning-configuration scanOnPush=true
```

#### Best Practices

1. **Use official base images**
2. **Scan images regularly**
3. **Update dependencies**
4. **Use specific image tags** (not `latest`)
5. **Remove unnecessary tools** from production images
6. **Run containers as non-root user**

### 5. Encryption

#### At Rest
- EBS volumes (EC2 launch type)
- EFS file systems
- S3 buckets
- ECR repositories

#### In Transit
- TLS for load balancers
- VPC encryption (optional)
- Service-to-service encryption

---

## Monitoring & Logging

### 1. CloudWatch Logs

**Log Drivers:**
- `awslogs`: CloudWatch Logs (recommended)
- `splunk`: Splunk logging
- `awsfirelens`: FluentBit/Fluentd

**Configuration:**
```json
{
  "logConfiguration": {
    "logDriver": "awslogs",
    "options": {
      "awslogs-group": "/ecs/my-app",
      "awslogs-region": "us-east-1",
      "awslogs-stream-prefix": "ecs"
    }
  }
}
```

**Best Practices:**
- Set retention policies (7, 30, 90 days)
- Use log groups per service
- Enable log insights for querying

### 2. Container Insights

**What is it?**
Enhanced monitoring for ECS clusters and services.

**Metrics Provided:**
- CPU utilization
- Memory utilization
- Network throughput
- Task count
- Service metrics

**Enable:**
```bash
aws ecs update-cluster-settings \
  --cluster my-cluster \
  --settings name=containerInsights,value=enabled
```

**Cost:** Additional CloudWatch charges apply

### 3. CloudWatch Metrics

**Task-Level Metrics:**
- CPUUtilization
- MemoryUtilization
- TaskCount

**Service-Level Metrics:**
- DesiredTaskCount
- RunningTaskCount
- PendingTaskCount

**Cluster-Level Metrics:**
- CPUReservation
- MemoryReservation
- RegisteredContainerInstancesCount

### 4. CloudWatch Alarms

**Examples:**

```bash
# High CPU alarm
aws cloudwatch put-metric-alarm \
  --alarm-name high-cpu \
  --alarm-description "CPU above 80%" \
  --metric-name CPUUtilization \
  --namespace AWS/ECS \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2

# Unhealthy tasks
aws cloudwatch put-metric-alarm \
  --alarm-name unhealthy-tasks \
  --alarm-description "Unhealthy task count > 0" \
  --metric-name HealthyHostCount \
  --namespace AWS/ApplicationELB \
  --statistic Average \
  --period 60 \
  --threshold 1 \
  --comparison-operator LessThanThreshold
```

### 5. AWS X-Ray

**Distributed Tracing:**
- Trace requests across services
- Identify bottlenecks
- Visualize service map

**Setup:**
1. Add X-Ray daemon as sidecar container
2. Instrument application code
3. Configure IAM permissions

### 6. Third-Party Monitoring

**Popular Tools:**
- Datadog
- New Relic
- Dynatrace
- Prometheus + Grafana

---

## Best Practices Summary

### 1. Design
- Use Fargate for simplicity, EC2 for cost optimization
- Implement health checks
- Design for failure
- Use multiple availability zones

### 2. Security
- Use task IAM roles
- Store secrets in Secrets Manager
- Enable encryption at rest and in transit
- Scan container images regularly
- Follow least privilege principle

### 3. Performance
- Right-size tasks (CPU/memory)
- Use auto-scaling
- Implement caching
- Optimize container images

### 4. Cost Optimization
- Use Fargate Spot for fault-tolerant workloads
- Use EC2 Spot instances
- Right-size resources
- Delete unused resources
- Use Reserved Instances for predictable workloads

### 5. Operations
- Enable Container Insights
- Set up comprehensive monitoring
- Implement CI/CD pipelines
- Use Infrastructure as Code (CloudFormation, Terraform)
- Tag all resources

### 6. Networking
- Use awsvpc network mode
- Implement service discovery
- Use private subnets for tasks
- Configure security groups properly

---

## Common Use Cases

1. **Microservices Architecture**
   - Multiple services communicating via service discovery
   - Each service scales independently
   - Use ALB for routing

2. **Batch Processing**
   - Run tasks on schedule or trigger
   - Use Fargate Spot for cost savings
   - Auto-scale based on queue depth

3. **Web Applications**
   - Frontend + Backend services
   - ALB for load balancing
   - Auto-scaling for high availability

4. **CI/CD Pipelines**
   - Build and test containers
   - Deploy to staging/production
   - Blue-green deployments

5. **Machine Learning**
   - Training jobs on GPU instances
   - Inference services
   - Batch predictions

---

## Next Steps

1. Complete the basic tutorial
2. Work on the advanced task
3. Prepare for interview questions
4. Build a real-world project

---

**Happy Learning! 🚀**
