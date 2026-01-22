# 📘 AWS Lambda - Detailed Notes

## 📋 Table of Contents
1. [Introduction to AWS Lambda](#introduction-to-aws-lambda)
2. [Core Concepts](#core-concepts)
3. [Lambda Architecture](#lambda-architecture)
4. [Function Configuration](#function-configuration)
5. [Invocation Methods](#invocation-methods)
6. [Execution Environment](#execution-environment)
7. [Permissions & Security](#permissions--security)
8. [Performance Optimization](#performance-optimization)
9. [Integration with AWS Services](#integration-with-aws-services)
10. [Monitoring & Logging](#monitoring--logging)
11. [Best Practices](#best-practices)
12. [Limitations](#limitations)
13. [Pricing](#pricing)

---

## Introduction to AWS Lambda

### What is AWS Lambda?

AWS Lambda is a **serverless compute service** that allows you to run code without provisioning or managing servers. It automatically scales your applications by running code in response to events and only charges you for the compute time you consume.

### Key Characteristics

- **Event-driven:** Executes code in response to triggers
- **Serverless:** No infrastructure management required
- **Auto-scaling:** Automatically scales based on demand
- **Pay-per-use:** Only pay for actual execution time
- **Highly available:** Built-in fault tolerance across multiple availability zones

### Use Cases

- Real-time file processing
- Stream processing
- Web applications and APIs
- IoT backends
- Data transformation
- Scheduled tasks (cron jobs)
- Chatbots and voice assistants
- ETL operations

---

## Core Concepts

### Lambda Function

A Lambda function is the fundamental unit in AWS Lambda. It consists of:
- **Code:** Your application logic
- **Configuration:** Runtime, memory, timeout, environment variables
- **Execution Role:** IAM role with permissions
- **Triggers:** Events that invoke the function

### Handler Function

The entry point that AWS Lambda calls to start execution of your code.

**Example (Python):**
```python
def lambda_handler(event, context):
    # event: Input data passed to the function
    # context: Runtime information
    return {
        'statusCode': 200,
        'body': 'Hello from Lambda!'
    }
```

### Event Object

Contains data about the triggering event. Structure varies based on the event source.

### Context Object

Provides runtime information:
- Request ID
- Remaining execution time
- Function name and version
- Memory limit
- CloudWatch log stream

---

## Lambda Architecture

### Components

1. **Event Source:** Service that triggers Lambda (S3, API Gateway, etc.)
2. **Lambda Function:** Your code
3. **Execution Environment:** Runtime container
4. **Destinations:** Where results are sent (optional)

### Execution Model

```
Event → Event Source Mapping → Lambda Service → Execution Environment → Function Code → Response
```

### Execution Environment Lifecycle

1. **Init Phase:**
   - Download function code and dependencies
   - Initialize runtime
   - Run initialization code (outside handler)
   - Duration: Milliseconds to seconds

2. **Invoke Phase:**
   - Execute handler function
   - Process event
   - Return response

3. **Shutdown Phase:**
   - Environment terminated after inactivity
   - Typically after 15-30 minutes of idle time

### Container Reuse (Warm vs Cold Start)

**Cold Start:**
- First invocation or after idle period
- Complete environment initialization required
- Higher latency (100ms - several seconds)

**Warm Start:**
- Execution environment reused
- Only invoke phase executed
- Lower latency (single-digit milliseconds)

---

## Function Configuration

### Runtime

Supported runtimes:
- **Python:** 3.8, 3.9, 3.10, 3.11, 3.12
- **Node.js:** 16.x, 18.x, 20.x
- **Java:** 8, 11, 17, 21
- **.NET:** 6, 8
- **Go:** 1.x
- **Ruby:** 3.2, 3.3
- **Custom Runtime:** Using Runtime API

### Memory Allocation

- Range: 128 MB to 10,240 MB (10 GB)
- Increment: 1 MB
- CPU power scales proportionally with memory
- More memory = faster execution but higher cost

### Timeout

- Range: 1 second to 900 seconds (15 minutes)
- Default: 3 seconds
- Important: Set based on expected execution time

### Environment Variables

Key-value pairs available to function code:
- Maximum size: 4 KB
- Can be encrypted using AWS KMS
- Useful for configuration without code changes

### Ephemeral Storage (/tmp)

- Range: 512 MB to 10,240 MB (10 GB)
- Default: 512 MB
- Temporary file storage during execution
- Persists during execution context
- Not shared between instances

---

## Invocation Methods

### 1. Synchronous Invocation

**How it works:**
- Caller waits for function completion
- Function returns response directly
- Error handling is caller's responsibility

**Event Sources:**
- Amazon API Gateway
- Application Load Balancer
- AWS CLI/SDK
- Amazon Cognito
- AWS Step Functions

**Example:**
```bash
aws lambda invoke \
  --function-name my-function \
  --payload '{"key": "value"}' \
  response.json
```

### 2. Asynchronous Invocation

**How it works:**
- Lambda queues the event
- Returns success immediately (202 status)
- Lambda handles retries (2 automatic retries)
- Can configure Dead Letter Queue (DLQ)

**Event Sources:**
- Amazon S3
- Amazon SNS
- Amazon EventBridge
- AWS CodeCommit
- AWS IoT

**Retry Behavior:**
- First retry: After 1 minute
- Second retry: After 2 minutes
- If still fails: Send to DLQ (if configured)

### 3. Poll-Based Invocation (Event Source Mapping)

**How it works:**
- Lambda polls the event source
- Retrieves records in batches
- Invokes function with batch
- Manages checkpoints automatically

**Event Sources:**
- Amazon Kinesis Data Streams
- Amazon DynamoDB Streams
- Amazon SQS
- Amazon MSK (Kafka)

**Characteristics:**
- Lambda reads from source
- Batch processing
- Automatic retry with exponential backoff
- Records processed in order (for streams)

---

## Execution Environment

### Execution Context

A temporary runtime environment that includes:
- `/tmp` directory for temporary files
- Background processes or callbacks
- Static initialization code
- Connections to databases/services

**Benefits of reusing context:**
- Reduced latency
- Connection pooling
- Cached data

**Best Practice:**
```python
# Initialize outside handler (reused across invocations)
import boto3
s3_client = boto3.client('s3')

def lambda_handler(event, context):
    # This runs on every invocation
    s3_client.get_object(Bucket='my-bucket', Key='file.txt')
```

### Concurrency

**How it works:**
- Lambda creates one execution environment per concurrent request
- Scales automatically
- Default limit: 1,000 concurrent executions per region (can be increased)

**Types of Concurrency:**

1. **Unreserved Concurrency:**
   - Shared pool across all functions
   - Default behavior

2. **Reserved Concurrency:**
   - Dedicated capacity for a function
   - Prevents function from using more than specified
   - Can throttle other functions

3. **Provisioned Concurrency:**
   - Pre-initialized execution environments
   - Eliminates cold starts
   - Always ready to respond
   - Additional cost

---

## Permissions & Security

### Execution Role (IAM Role)

Grants Lambda permissions to access AWS services.

**Required permissions:**
- CloudWatch Logs (create log groups, streams, put events)
- VPC networking (if VPC-enabled)
- Service-specific permissions (S3, DynamoDB, etc.)

**Example Policy:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

### Resource-Based Policy

Controls which services/accounts can invoke your function.

**Use cases:**
- Allow API Gateway to invoke function
- Cross-account access
- Service-to-service permissions

### VPC Configuration

Lambda can access resources in your VPC:
- Specify subnets and security groups
- Lambda creates ENIs (Elastic Network Interfaces)
- Can access RDS, ElastiCache, internal APIs
- Internet access requires NAT Gateway

### Encryption

- **At rest:** Function code encrypted using KMS
- **In transit:** All data encrypted with TLS
- **Environment variables:** Can be encrypted with KMS
- **Secrets:** Use AWS Secrets Manager or Parameter Store

---

## Performance Optimization

### Reducing Cold Start Time

1. **Minimize package size:**
   - Remove unnecessary dependencies
   - Use Lambda Layers for shared code
   - Exclude development dependencies

2. **Use Provisioned Concurrency:**
   - For latency-sensitive applications
   - Keeps functions warm

3. **Optimize initialization code:**
   - Minimize work outside handler
   - Lazy load large dependencies
   - Use smaller runtime (Python, Node.js faster than Java, .NET)

4. **Keep functions warm:**
   - Scheduled CloudWatch Events (ping every 5 minutes)
   - Not recommended for cost optimization

### Improving Execution Performance

1. **Increase memory allocation:**
   - CPU power scales with memory
   - Test different configurations

2. **Optimize code:**
   - Use efficient algorithms
   - Minimize network calls
   - Implement caching

3. **Connection pooling:**
   - Reuse database connections
   - Use RDS Proxy

4. **Parallel processing:**
   - Use async/await patterns
   - Process independent tasks concurrently

### Memory vs Cost Trade-off

Higher memory = Higher CPU power but higher cost per millisecond
Sometimes increasing memory reduces total cost due to faster execution.

**Formula:**
```
Cost = (Memory/1024) × Execution Time × Price per GB-second
```

---

## Integration with AWS Services

### Common Integrations

**Amazon S3:**
- Trigger: Object creation, deletion
- Use case: Image processing, data transformation

**Amazon API Gateway:**
- Trigger: HTTP requests
- Use case: RESTful APIs, webhooks

**Amazon DynamoDB:**
- Trigger: DynamoDB Streams
- Use case: Real-time data processing, auditing

**Amazon SQS:**
- Trigger: Messages in queue
- Use case: Asynchronous processing, decoupling

**Amazon EventBridge:**
- Trigger: Scheduled events, custom events
- Use case: Cron jobs, event-driven workflows

**Amazon Kinesis:**
- Trigger: Stream records
- Use case: Real-time analytics, log processing

**AWS Step Functions:**
- Orchestrate multiple Lambda functions
- Use case: Complex workflows, state machines

---

## Monitoring & Logging

### CloudWatch Logs

- Automatic logging to CloudWatch
- Log groups created per function
- Log streams per execution environment
- Contains function output and errors

**Log format:**
```
START RequestId: abc123 Version: $LATEST
[INFO] Processing event
END RequestId: abc123
REPORT RequestId: abc123 Duration: 142.50 ms Billed Duration: 143 ms Memory Size: 512 MB Max Memory Used: 85 MB
```

### CloudWatch Metrics

**Default metrics:**
- **Invocations:** Number of times function invoked
- **Duration:** Time function takes to execute
- **Errors:** Number of failed invocations
- **Throttles:** Number of throttled invocation attempts
- **ConcurrentExecutions:** Number of concurrent executions
- **DeadLetterErrors:** Failures sending to DLQ

### AWS X-Ray

Distributed tracing for Lambda functions:
- End-to-end request tracking
- Service map visualization
- Performance bottleneck identification
- Error and exception analysis

**Enable X-Ray:**
- Add IAM permissions
- Enable active tracing in function configuration
- Use X-Ray SDK in code

### CloudWatch Insights

Query and analyze logs:
```
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 20
```

---

## Best Practices

### Code Organization

1. **Separate handler from business logic**
2. **Use environment variables for configuration**
3. **Implement proper error handling**
4. **Keep functions small and focused (single responsibility)**
5. **Use Lambda Layers for shared code**

### Performance

1. **Initialize SDK clients outside handler**
2. **Reuse database connections**
3. **Minimize deployment package size**
4. **Use appropriate memory settings**
5. **Implement caching strategies**

### Security

1. **Follow principle of least privilege**
2. **Encrypt sensitive data**
3. **Use Secrets Manager for credentials**
4. **Enable VPC for private resource access**
5. **Regularly rotate IAM credentials**

### Reliability

1. **Implement idempotency**
2. **Use Dead Letter Queues**
3. **Set appropriate timeouts**
4. **Handle partial batch failures**
5. **Implement retry logic with exponential backoff**

### Monitoring

1. **Enable CloudWatch Logs**
2. **Create CloudWatch Alarms for errors and throttles**
3. **Use structured logging**
4. **Implement distributed tracing with X-Ray**
5. **Monitor costs regularly**

### Development

1. **Use Infrastructure as Code (SAM, Serverless, CDK)**
2. **Implement CI/CD pipelines**
3. **Test locally using SAM CLI**
4. **Use aliases and versions for deployment**
5. **Implement blue/green deployments**

---

## Limitations

### Hard Limits (Cannot be changed)

| Resource | Limit |
|----------|-------|
| Maximum execution time | 15 minutes (900 seconds) |
| Deployment package size (compressed) | 50 MB |
| Deployment package size (uncompressed) | 250 MB |
| /tmp directory storage | 512 MB - 10 GB |
| Environment variables | 4 KB |
| Lambda function layers | 5 layers per function |
| Invocation payload (request/response) | 6 MB (synchronous), 256 KB (asynchronous) |

### Soft Limits (Can be increased)

| Resource | Default Limit |
|----------|---------------|
| Concurrent executions per region | 1,000 |
| Function and layer storage | 75 GB |
| Elastic network interfaces per VPC | 250 |

### Other Constraints

- Maximum function memory: 10,240 MB (10 GB)
- Maximum environment variable size: 4 KB
- Maximum test event size: 10 KB
- Maximum resource-based policy size: 20 KB

---

## Pricing

### Pricing Components

1. **Request Charges:**
   - First 1 million requests per month: **Free**
   - After that: **$0.20 per 1 million requests**

2. **Duration Charges:**
   - Charged per GB-second
   - Calculated from execution start to completion
   - Rounded up to nearest 1 ms
   - **$0.0000166667 per GB-second**

3. **Provisioned Concurrency:**
   - Additional charge for keeping instances warm
   - Per GB-hour for configured concurrency

4. **Data Transfer:**
   - Data transfer out to internet charged
   - Data transfer within same region free

### Free Tier

- **1 million requests per month** (forever free)
- **400,000 GB-seconds of compute time per month** (forever free)

### Pricing Example

**Scenario:**
- Memory: 512 MB (0.5 GB)
- Execution time: 200 ms (0.2 seconds)
- Requests: 5 million per month

**Calculation:**
```
Request cost:
(5,000,000 - 1,000,000) × $0.20 / 1,000,000 = $0.80

Duration cost:
5,000,000 × 0.5 GB × 0.2 s = 500,000 GB-seconds
500,000 × $0.0000166667 = $8.33

Total: $0.80 + $8.33 = $9.13 per month
```

### Cost Optimization Tips

1. Right-size memory allocation
2. Reduce execution time
3. Use Lambda Layers to reduce package size
4. Implement caching
5. Use reserved concurrency wisely
6. Monitor and eliminate idle functions
7. Use AWS Cost Explorer

---

## Advanced Topics

### Lambda Layers

Reusable code packages shared across functions:

**Benefits:**
- Reduce deployment package size
- Promote code reuse
- Separate dependencies from function code
- Version control for shared components

**Use cases:**
- Custom runtimes
- Shared libraries
- Configuration files
- SDKs and tools

### Versions and Aliases

**Versions:**
- Immutable snapshots of function code and configuration
- $LATEST is always mutable
- Each publish creates a new version number

**Aliases:**
- Pointers to specific versions
- Enable traffic shifting between versions
- Support blue/green deployments

**Example workflow:**
```
Code → Publish as v2 → Create alias "PROD" → Point to v2
                     ↓
              (Keep v1 as rollback)
```

### Lambda Destinations

Route asynchronous invocation results:

**On Success:**
- SQS queue
- SNS topic
- Lambda function
- EventBridge event bus

**On Failure:**
- Same options as success
- Better than DLQ (more metadata)

### Lambda Container Images

Deploy Lambda functions as container images:

**Features:**
- Up to 10 GB image size
- Use familiar container tools
- Include large dependencies
- Custom runtime environments

**Requirements:**
- Implement Lambda Runtime API
- Use AWS-provided base images or compatible images

---

**End of Notes**

*Use these notes alongside the interview questions for comprehensive preparation!* 🚀
