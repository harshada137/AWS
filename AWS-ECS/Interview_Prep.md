
## 🟢 1. What is AWS ECS and why is it used?

AWS **Elastic Container Service (ECS)** is a fully managed container orchestration service used to **run, manage, and scale Docker containers** on AWS.

It is used because:

* It removes the complexity of managing container infrastructure
* Integrates deeply with AWS services (IAM, ALB, CloudWatch, ECR)
* Supports both **serverless (Fargate)** and **server-based (EC2)** models

---

## 🟢 2. What are the main components of ECS?

The core ECS components are:

* **Cluster** – Logical grouping where containers run
* **Task Definition** – Blueprint describing how containers should run
* **Task** – A running instance of a task definition
* **Service** – Manages long-running tasks and ensures desired count
* **Container** – Docker container running inside a task

---

## 🟢 3. What is an ECS cluster?

An ECS cluster is a **logical boundary** where ECS schedules and runs tasks or services.

* With **EC2 launch type**, it contains EC2 instances
* With **Fargate**, it contains no servers (AWS manages infra)

It does **not** run containers by itself.

---

## 🟢 4. What is a task definition?

A task definition is a **JSON blueprint** that defines:

* Docker image
* CPU and memory
* Ports
* Environment variables
* IAM roles
* Logging configuration

ECS uses this definition to launch containers.

---

## 🟢 5. What is the difference between a task and a service?

| Task                    | Service                |
| ----------------------- | ---------------------- |
| One-time or short-lived | Long-running           |
| Can stop anytime        | Automatically restarts |
| Manual scaling          | Auto scaling           |
| Used for batch jobs     | Used for web apps      |

---

## 🟢 6. What launch types are supported by ECS?

ECS supports:

1. **EC2 Launch Type** – You manage EC2 instances
2. **Fargate Launch Type** – Serverless containers (AWS manages infra)

---

## 🟢 7. What is AWS Fargate?

AWS Fargate is a **serverless compute engine** for containers.

With Fargate:

* No EC2 instance management
* You only specify CPU, memory, and container image
* AWS handles provisioning, scaling, and patching

---

## 🟢 8. When would you choose Fargate over EC2 in ECS?

Choose **Fargate** when:

* You want zero server management
* Workloads are unpredictable
* Faster deployment is required
* You want better security isolation

Choose **EC2** when:

* You need custom AMIs
* Want lower cost for steady workloads
* Need GPU or special hardware

---

## 🟢 9. What is an ECS service role?

An ECS service role allows ECS to:

* Register/deregister tasks with ALB
* Manage load balancer listeners
* Perform health checks

It gives ECS permissions to act **on your behalf**.

---

## 🟢 10. What is the purpose of `ecsTaskExecutionRole`?

The **ecsTaskExecutionRole** allows ECS to:

* Pull images from **Amazon ECR**
* Write logs to **CloudWatch**
* Retrieve secrets from **SSM / Secrets Manager**

Without this role, tasks may fail to start.

---

## 🟡 11. How does networking work in ECS with Fargate?

Fargate uses **awsvpc network mode**:

* Each task gets its own **Elastic Network Interface (ENI)**
* Each task gets a **private IP**
* Optional public IP for internet access
* Security groups are attached directly to tasks

---

## 🟡 12. What is the difference between bridge and awsvpc network modes?

| Bridge                  | awsvpc                 |
| ----------------------- | ---------------------- |
| Shared EC2 networking   | Dedicated ENI per task |
| Port conflicts possible | No port conflicts      |
| Used with EC2           | Required for Fargate   |
| Less secure             | More secure            |

---

## 🟡 13. How does ECS integrate with Application Load Balancer?

ECS integrates with ALB by:

* Registering tasks as targets
* Routing traffic using target groups
* Using health checks to restart unhealthy tasks
* Supporting path-based and host-based routing

Used mainly with **ECS services**.

---

## 🟡 14. How do you perform rolling updates in ECS?

Rolling updates are done by:

* Updating the task definition
* ECS gradually replaces old tasks with new ones
* Controlled using:

  * Minimum healthy percent
  * Maximum percent

This ensures **zero downtime deployments**.

---

## 🟡 15. What happens if an ECS task fails?

* **Standalone task:** Stops and does not restart
* **Service task:** ECS automatically restarts it
* Failure details appear in:

  * ECS task events
  * CloudWatch logs

---

## 🟡 16. How do you scale applications in ECS?

Scaling can be done by:

* Increasing desired task count manually
* Auto Scaling based on:

  * CPU utilization
  * Memory utilization
  * CloudWatch alarms

Supported for ECS services.

---

## 🟡 17. How are environment variables managed in ECS?

Environment variables can be set:

* Directly in task definition
* From **SSM Parameter Store**
* From **AWS Secrets Manager**

Sensitive data should always use **secrets**, not plain text.

---

## 🟡 18. How does ECS pull images from Amazon ECR?

Process:

1. Task execution role authenticates to ECR
2. ECS pulls image from repository
3. Image is cached (EC2) or fetched (Fargate)

If permissions are missing, task fails with image pull error.

---

## 🟡 19. How do security groups work in ECS?

* Security groups control inbound and outbound traffic
* With Fargate:

  * Attached directly to the task ENI
* With EC2:

  * Attached to EC2 instance and optionally tasks

They define which ports are accessible.

---

## 🔴 20. What are common reasons ECS tasks fail to start?

Common reasons:

* Incorrect IAM roles
* Image pull failure
* Insufficient CPU or memory
* Wrong port configuration
* Security group blocking traffic
* Container crashes on startup


