# Basic Tutorial: Deploy a Web Application on AWS ECS

## 🎯 Objective
Deploy a simple Nginx web server on AWS ECS using Fargate launch type, with access via Application Load Balancer.

## 📋 Prerequisites
- AWS Account with administrative access
- AWS CLI installed and configured (`aws configure`)
- Basic understanding of Docker
- A VPC with public subnets (or use default VPC)

## ⏱️ Estimated Time
30-45 minutes

---

## Step 1: Create an ECR Repository

Amazon Elastic Container Registry (ECR) is where we'll store our Docker image.

```bash
# Create the repository
aws ecr create-repository \
  --repository-name my-nginx-app \
  --region us-east-1

# Expected output will show repository details including the repositoryUri
# Save this URI - you'll need it later
# Example: 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-nginx-app
```

**What this does:** Creates a private Docker registry in AWS to store your container images.

---

## Step 2: Create a Simple Web Application

Create a directory for your project:

```bash
mkdir my-nginx-app
cd my-nginx-app
```

### Create index.html

```bash
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>My ECS Application</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
        }
        .container {
            text-align: center;
            padding: 40px;
            background: rgba(255, 255, 255, 0.1);
            border-radius: 20px;
            backdrop-filter: blur(10px);
        }
        h1 { font-size: 3em; margin: 0; }
        p { font-size: 1.2em; margin-top: 20px; }
    </style>
</head>
<body>
    <div class="container">
        <h1>🚀 Hello from AWS ECS!</h1>
        <p>Your containerized application is running successfully on Fargate</p>
        <p><strong>Server: Nginx on Amazon ECS</strong></p>
    </div>
</body>
</html>
EOF
```

### Create Dockerfile

```bash
cat > Dockerfile << 'EOF'
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
EOF
```

**What this does:** 
- Uses lightweight Nginx Alpine image
- Copies your HTML file into the web server
- Exposes port 80 for HTTP traffic

---

## Step 3: Build and Push Docker Image

### Authenticate Docker to ECR

```bash
# Replace <account-id> with your AWS account ID
# Replace <region> with your AWS region
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  <account-id>.dkr.ecr.us-east-1.amazonaws.com
```

**Find your account ID:**
```bash
aws sts get-caller-identity --query Account --output text
```

### Build the Docker image

```bash
docker build -t my-nginx-app .
```

### Tag the image

```bash
# Replace <account-id> with your actual account ID
docker tag my-nginx-app:latest \
  <account-id>.dkr.ecr.us-east-1.amazonaws.com/my-nginx-app:latest
```

### Push to ECR

```bash
docker push <account-id>.dkr.ecr.us-east-1.amazonaws.com/my-nginx-app:latest
```

**Verify the push:**
```bash
aws ecr describe-images --repository-name my-nginx-app --region us-east-1
```

---

## Step 4: Create an ECS Cluster

```bash
aws ecs create-cluster \
  --cluster-name my-web-cluster \
  --region us-east-1
```

**What this does:** Creates a logical grouping for your ECS tasks and services.

**Verify:**
```bash
aws ecs describe-clusters --clusters my-web-cluster --region us-east-1
```

---

## Step 5: Create IAM Roles

### Create Task Execution Role

This role allows ECS to pull images from ECR and write logs to CloudWatch.

#### Create trust policy file

```bash
cat > trust-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ecs-tasks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF
```

#### Create the role

```bash
aws iam create-role \
  --role-name ecsTaskExecutionRole \
  --assume-role-policy-document file://trust-policy.json
```

#### Attach the managed policy

```bash
aws iam attach-role-policy \
  --role-name ecsTaskExecutionRole \
  --policy-arn arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy
```

**What this does:** Grants ECS permissions to pull container images and send logs to CloudWatch.

---

## Step 6: Create CloudWatch Log Group

```bash
aws logs create-log-group \
  --log-group-name /ecs/my-nginx-app \
  --region us-east-1
```

**What this does:** Creates a log group where your container logs will be stored.

---

## Step 7: Register Task Definition

Create a task definition file:

```bash
# First, get your account ID
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

# Create task definition file
cat > task-definition.json << EOF
{
  "family": "my-nginx-task",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "executionRoleArn": "arn:aws:iam::${ACCOUNT_ID}:role/ecsTaskExecutionRole",
  "containerDefinitions": [
    {
      "name": "nginx",
      "image": "${ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/my-nginx-app:latest",
      "portMappings": [
        {
          "containerPort": 80,
          "protocol": "tcp"
        }
      ],
      "essential": true,
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/my-nginx-app",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
EOF
```

**Register the task definition:**

```bash
aws ecs register-task-definition \
  --cli-input-json file://task-definition.json
```

**Understanding Task Definition Parameters:**
- `cpu: "256"` = 0.25 vCPU
- `memory: "512"` = 512 MB RAM
- `networkMode: "awsvpc"` = Each task gets its own network interface
- `requiresCompatibilities: ["FARGATE"]` = Serverless launch type

---

## Step 8: Create Application Load Balancer

### Get your VPC ID and Subnets

```bash
# Get default VPC ID
VPC_ID=$(aws ec2 describe-vpcs \
  --filters Name=isDefault,Values=true \
  --query 'Vpcs[0].VpcId' \
  --output text)

echo "VPC ID: $VPC_ID"

# Get two public subnets in different AZs
SUBNETS=$(aws ec2 describe-subnets \
  --filters "Name=vpc-id,Values=$VPC_ID" \
  --query 'Subnets[0:2].SubnetId' \
  --output text)

SUBNET_1=$(echo $SUBNETS | awk '{print $1}')
SUBNET_2=$(echo $SUBNETS | awk '{print $2}')

echo "Subnet 1: $SUBNET_1"
echo "Subnet 2: $SUBNET_2"
```

### Create Security Group for ALB

```bash
ALB_SG_ID=$(aws ec2 create-security-group \
  --group-name alb-nginx-sg \
  --description "Security group for ALB" \
  --vpc-id $VPC_ID \
  --query 'GroupId' \
  --output text)

echo "ALB Security Group: $ALB_SG_ID"

# Allow HTTP traffic from anywhere
aws ec2 authorize-security-group-ingress \
  --group-id $ALB_SG_ID \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0
```

### Create the Application Load Balancer

```bash
ALB_ARN=$(aws elbv2 create-load-balancer \
  --name my-nginx-alb \
  --subnets $SUBNET_1 $SUBNET_2 \
  --security-groups $ALB_SG_ID \
  --scheme internet-facing \
  --type application \
  --ip-address-type ipv4 \
  --query 'LoadBalancers[0].LoadBalancerArn' \
  --output text)

echo "ALB ARN: $ALB_ARN"

# Get ALB DNS name
ALB_DNS=$(aws elbv2 describe-load-balancers \
  --load-balancer-arns $ALB_ARN \
  --query 'LoadBalancers[0].DNSName' \
  --output text)

echo "ALB DNS: $ALB_DNS"
echo "Access your app at: http://$ALB_DNS"
```

### Create Target Group

```bash
TARGET_GROUP_ARN=$(aws elbv2 create-target-group \
  --name my-nginx-targets \
  --protocol HTTP \
  --port 80 \
  --vpc-id $VPC_ID \
  --target-type ip \
  --health-check-path / \
  --health-check-interval-seconds 30 \
  --health-check-timeout-seconds 5 \
  --healthy-threshold-count 2 \
  --unhealthy-threshold-count 3 \
  --query 'TargetGroups[0].TargetGroupArn' \
  --output text)

echo "Target Group ARN: $TARGET_GROUP_ARN"
```

### Create Listener

```bash
aws elbv2 create-listener \
  --load-balancer-arn $ALB_ARN \
  --protocol HTTP \
  --port 80 \
  --default-actions Type=forward,TargetGroupArn=$TARGET_GROUP_ARN
```

---

## Step 9: Create Security Group for ECS Tasks

```bash
TASK_SG_ID=$(aws ec2 create-security-group \
  --group-name ecs-nginx-tasks-sg \
  --description "Security group for ECS tasks" \
  --vpc-id $VPC_ID \
  --query 'GroupId' \
  --output text)

echo "Task Security Group: $TASK_SG_ID"

# Allow traffic from ALB only
aws ec2 authorize-security-group-ingress \
  --group-id $TASK_SG_ID \
  --protocol tcp \
  --port 80 \
  --source-group $ALB_SG_ID
```

**Security Best Practice:** Tasks only accept traffic from the ALB, not directly from the internet.

---

## Step 10: Create ECS Service

```bash
aws ecs create-service \
  --cluster my-web-cluster \
  --service-name my-nginx-service \
  --task-definition my-nginx-task \
  --desired-count 2 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[$SUBNET_1,$SUBNET_2],securityGroups=[$TASK_SG_ID],assignPublicIp=ENABLED}" \
  --load-balancers targetGroupArn=$TARGET_GROUP_ARN,containerName=nginx,containerPort=80
```

**What this does:**
- Creates a service that maintains 2 running tasks
- Tasks are placed in public subnets (for demo purposes)
- Tasks register with the ALB target group
- ECS automatically replaces failed tasks

---

## Step 11: Verify Deployment

### Check service status

```bash
aws ecs describe-services \
  --cluster my-web-cluster \
  --services my-nginx-service \
  --query 'services[0].{Status:status,Running:runningCount,Desired:desiredCount}' \
  --output table
```

### List running tasks

```bash
aws ecs list-tasks \
  --cluster my-web-cluster \
  --service-name my-nginx-service
```

### Check task details

```bash
# Get task ARN
TASK_ARN=$(aws ecs list-tasks \
  --cluster my-web-cluster \
  --service-name my-nginx-service \
  --query 'taskArns[0]' \
  --output text)

# Describe task
aws ecs describe-tasks \
  --cluster my-web-cluster \
  --tasks $TASK_ARN
```

### Wait for healthy targets

```bash
echo "Waiting for targets to become healthy..."
aws elbv2 wait target-in-service \
  --target-group-arn $TARGET_GROUP_ARN
echo "Targets are healthy!"
```

### Access your application

```bash
echo "Your application is available at:"
echo "http://$ALB_DNS"
```

Open this URL in your browser to see your application!

---

## Step 12: View Logs

```bash
# List log streams
aws logs describe-log-streams \
  --log-group-name /ecs/my-nginx-app \
  --order-by LastEventTime \
  --descending \
  --max-items 1 \
  --query 'logStreams[0].logStreamName' \
  --output text

# View logs (replace <log-stream-name> with the output from above)
aws logs tail /ecs/my-nginx-app --follow
```

---

## Step 13: Test Scaling

### Manually scale your service

```bash
aws ecs update-service \
  --cluster my-web-cluster \
  --service my-nginx-service \
  --desired-count 4
```

Watch the service scale up:

```bash
watch -n 5 'aws ecs describe-services \
  --cluster my-web-cluster \
  --services my-nginx-service \
  --query "services[0].{Running:runningCount,Desired:desiredCount}" \
  --output table'
```

Press `Ctrl+C` to stop watching.

---

## Step 14: Clean Up Resources

**Important:** Run these commands to avoid AWS charges.

```bash
# Scale service to 0
aws ecs update-service \
  --cluster my-web-cluster \
  --service my-nginx-service \
  --desired-count 0

# Wait for tasks to stop
echo "Waiting for tasks to stop..."
sleep 30

# Delete service
aws ecs delete-service \
  --cluster my-web-cluster \
  --service my-nginx-service \
  --force

# Delete ALB listener
LISTENER_ARN=$(aws elbv2 describe-listeners \
  --load-balancer-arn $ALB_ARN \
  --query 'Listeners[0].ListenerArn' \
  --output text)

aws elbv2 delete-listener --listener-arn $LISTENER_ARN

# Delete ALB
aws elbv2 delete-load-balancer --load-balancer-arn $ALB_ARN

# Wait for ALB to delete
echo "Waiting for ALB to delete..."
sleep 30

# Delete target group
aws elbv2 delete-target-group --target-group-arn $TARGET_GROUP_ARN

# Delete cluster
aws ecs delete-cluster --cluster my-web-cluster

# Delete security groups
aws ec2 delete-security-group --group-id $TASK_SG_ID
aws ec2 delete-security-group --group-id $ALB_SG_ID

# Delete log group
aws logs delete-log-group --log-group-name /ecs/my-nginx-app

# Delete ECR repository (including all images)
aws ecr delete-repository \
  --repository-name my-nginx-app \
  --force

# Delete IAM role (detach policy first)
aws iam detach-role-policy \
  --role-name ecsTaskExecutionRole \
  --policy-arn arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy

aws iam delete-role --role-name ecsTaskExecutionRole

echo "Cleanup complete!"
```

---

## 🎉 Congratulations!

You've successfully:
- ✅ Created a Docker container
- ✅ Pushed it to Amazon ECR
- ✅ Deployed it to ECS with Fargate
- ✅ Set up an Application Load Balancer
- ✅ Configured logging
- ✅ Tested scaling
- ✅ Cleaned up all resources

---

## 📝 Key Takeaways

1. **ECR** stores your container images
2. **Task Definition** defines how to run your container
3. **Cluster** is a logical grouping of tasks
4. **Service** maintains desired number of tasks
5. **ALB** distributes traffic across tasks
6. **Fargate** handles all infrastructure automatically

---

## 🔍 Troubleshooting

### Tasks not starting?

```bash
# Check service events
aws ecs describe-services \
  --cluster my-web-cluster \
  --services my-nginx-service \
  --query 'services[0].events[0:5]' \
  --output table
```

### Can't access the application?

1. Check ALB security group allows port 80
2. Check task security group allows traffic from ALB
3. Verify targets are healthy in target group
4. Check task logs for errors

### Permission errors?

- Ensure ecsTaskExecutionRole has correct policies
- Verify AWS CLI credentials have necessary permissions

---

## 🚀 Next Steps

1. Add HTTPS with ACM certificate
2. Implement auto-scaling
3. Add a custom domain with Route 53
4. Deploy a multi-container application
5. Try the advanced task!

---

**Happy Deploying! 🎯**
