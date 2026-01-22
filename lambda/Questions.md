# 🎯 AWS Lambda Interview Questions & Answers

## Basic Questions

### 1. What is AWS Lambda?
**Answer:** AWS Lambda is a serverless compute service that runs your code in response to events without requiring you to provision or manage servers. You only pay for the compute time you consume, and there is no charge when your code is not running.

### 2. What are the key features of AWS Lambda?
**Answer:** 
- No server management required
- Automatic scaling based on demand
- Pay-per-use pricing model
- Supports multiple programming languages
- Built-in fault tolerance and high availability
- Integration with other AWS services
- Subsecond metering

### 3. Which programming languages does AWS Lambda support?
**Answer:** AWS Lambda supports:
- Python
- Node.js
- Java
- C# (.NET Core)
- Go
- Ruby
- PowerShell
- Custom runtimes using Runtime API

### 4. What is a Lambda function?
**Answer:** A Lambda function is the code you upload to AWS Lambda. It consists of your code and any associated dependencies. Lambda runs your function only when needed and scales automatically.

### 5. What is the maximum execution timeout for a Lambda function?
**Answer:** The maximum execution timeout for a Lambda function is 15 minutes (900 seconds).

### 6. What are Lambda triggers?
**Answer:** Lambda triggers are AWS services or resources that can invoke your Lambda function. Common triggers include S3, API Gateway, DynamoDB, SNS, SQS, CloudWatch Events, and more.

### 7. What is a Lambda Layer?
**Answer:** Lambda Layers are a distribution mechanism for libraries, custom runtimes, and other function dependencies. Layers help you manage your code and dependencies separately, reduce deployment package size, and promote code sharing and reuse across functions.

### 8. What is the cold start problem in Lambda?
**Answer:** A cold start occurs when a Lambda function is invoked after being idle or for the first time. AWS needs to provision a new execution environment, load your code, and initialize it, which adds latency. Subsequent invocations use warm containers and execute faster.

### 9. What is the maximum memory you can allocate to a Lambda function?
**Answer:** You can allocate between 128 MB to 10,240 MB (10 GB) of memory to a Lambda function in 1 MB increments.

### 10. How does Lambda pricing work?
**Answer:** Lambda pricing is based on:
- Number of requests (first 1 million requests per month are free)
- Duration of execution (charged per GB-second)
- Data transfer out (if applicable)

---

## Intermediate Questions

### 11. What is the difference between synchronous and asynchronous invocation in Lambda?
**Answer:**
- **Synchronous:** The caller waits for the function to complete and returns a response. Used by API Gateway, ALB, and SDK invocations.
- **Asynchronous:** Lambda queues the event and returns immediately. Lambda retries failed invocations. Used by S3, SNS, CloudWatch Events.

### 12. What is a Lambda execution role?
**Answer:** An execution role is an IAM role that grants the Lambda function permissions to access AWS services and resources. The function assumes this role when it executes.

### 13. How can you reduce cold start times in Lambda?
**Answer:**
- Keep functions warm using scheduled CloudWatch Events
- Reduce deployment package size
- Use Lambda Provisioned Concurrency
- Optimize initialization code
- Use compiled languages (Java, Go) with GraalVM
- Minimize dependencies

### 14. What is Provisioned Concurrency in Lambda?
**Answer:** Provisioned Concurrency keeps a specified number of function instances initialized and ready to respond immediately. This eliminates cold starts for those instances but incurs additional costs.

### 15. What is the difference between Lambda Layers and deployment packages?
**Answer:**
- **Deployment Package:** Contains your function code and all dependencies bundled together
- **Lambda Layers:** Separate ZIP archives containing libraries and dependencies that can be shared across multiple functions, reducing deployment package size

### 16. How does Lambda handle concurrent executions?
**Answer:** Lambda automatically scales by creating multiple instances of your function to handle concurrent requests. By default, there's a regional concurrent execution limit (default 1,000), which can be increased. You can also set reserved concurrency for specific functions.

### 17. What are Lambda environment variables?
**Answer:** Environment variables are key-value pairs that you can configure for your Lambda function. They allow you to pass dynamic configuration to your code without changing the code itself. They can be encrypted using KMS.

### 18. What is a Dead Letter Queue (DLQ) in Lambda?
**Answer:** A DLQ is an SQS queue or SNS topic where Lambda sends event information when a function fails after exhausting all retry attempts (for asynchronous invocations). This helps in debugging and handling failed events.

### 19. How can you monitor Lambda functions?
**Answer:**
- CloudWatch Logs for function logs
- CloudWatch Metrics for invocations, duration, errors, throttles
- AWS X-Ray for distributed tracing
- CloudWatch Insights for log analysis
- Third-party monitoring tools

### 20. What is Lambda@Edge?
**Answer:** Lambda@Edge lets you run Lambda functions at AWS Edge locations (CloudFront edge locations) to customize content delivery. It's useful for low-latency responses, A/B testing, and request/response manipulation.

---

## Advanced Questions

### 21. Explain Lambda's execution context and how to optimize it.
**Answer:** The execution context is the runtime environment where your Lambda function runs. It includes:
- Temporary runtime environment
- Resources like /tmp directory (512 MB to 10 GB)
- Connections and variables that remain initialized between invocations

**Optimization:**
- Initialize SDK clients and database connections outside the handler
- Reuse execution context for warm starts
- Cache static data in the execution context
- Use /tmp for caching files

### 22. What are the Lambda function states?
**Answer:**
- **Pending:** Function is being created
- **Active:** Function is ready to be invoked
- **Inactive:** Function hasn't been invoked recently
- **Failed:** Function deployment or execution failed

### 23. How does Lambda integrate with VPC?
**Answer:** Lambda can access resources in a VPC by configuring VPC settings (subnet and security groups). Lambda creates Elastic Network Interfaces (ENIs) in your VPC. Note that VPC configuration can increase cold start times, though AWS has improved this with Hyperplane ENIs.

### 24. What is the /tmp directory in Lambda and its limitations?
**Answer:** The /tmp directory provides temporary storage for Lambda functions. 
- Size: 512 MB by default, configurable up to 10 GB
- Persists during the execution context
- Not shared between function instances
- Cleared when the execution context is terminated

### 25. Explain Lambda Reserved Concurrency vs Provisioned Concurrency.
**Answer:**
- **Reserved Concurrency:** Guarantees a set number of concurrent executions for a function and prevents it from using account-level concurrency. Can be set to zero to disable the function.
- **Provisioned Concurrency:** Keeps function instances initialized and ready, eliminating cold starts. Incurs charges even when not invoked.

### 26. How do you handle errors and retries in Lambda?
**Answer:**
- **Synchronous:** Errors returned to caller; no automatic retry by Lambda
- **Asynchronous:** Lambda automatically retries twice with delays, then sends to DLQ if configured
- **Stream-based (Kinesis/DynamoDB):** Retries until data expires or processed successfully
- Use try-catch blocks in code for custom error handling

### 27. What are Lambda destinations?
**Answer:** Destinations allow you to route asynchronous function results to AWS services without writing additional code. You can configure:
- On Success: Send to SQS, SNS, Lambda, or EventBridge
- On Failure: Send to SQS, SNS, Lambda, or EventBridge
This provides better visibility than DLQs with more routing options.

### 28. How does Lambda handle versioning and aliases?
**Answer:**
- **Versions:** Immutable snapshots of function code and configuration. $LATEST is the only mutable version.
- **Aliases:** Pointers to specific versions (e.g., PROD, DEV). Enable blue/green deployments and weighted traffic shifting between versions.

### 29. What is the Lambda execution lifecycle?
**Answer:**
1. **Init Phase:** Download code, start runtime, run initialization code
2. **Invoke Phase:** Execute handler function
3. **Shutdown Phase:** Runtime terminated (after period of inactivity)
4. Execution context may be reused (warm start) or recreated (cold start)

### 30. How can you implement CI/CD for Lambda functions?
**Answer:**
- Use AWS SAM or Serverless Framework for infrastructure as code
- Store code in CodeCommit, GitHub, or GitLab
- Use CodePipeline for orchestration
- Use CodeBuild for building and testing
- Use CodeDeploy for deployment with traffic shifting
- Implement automated testing (unit, integration)
- Use Lambda aliases for blue/green deployments

---

## Scenario-Based Questions

### 31. You have a Lambda function processing S3 events, but it's timing out. How would you troubleshoot?
**Answer:**
1. Check CloudWatch Logs for errors
2. Increase timeout setting (max 15 minutes)
3. Increase memory allocation (also increases CPU)
4. Optimize code for performance
5. Check external service latency (database, API calls)
6. Consider breaking into smaller functions
7. Use async processing if possible
8. Check for cold start issues

### 32. Your Lambda function needs to process large files. What's your approach?
**Answer:**
1. Use streaming rather than loading entire file
2. Utilize /tmp directory (up to 10 GB)
3. Consider EFS mounting for larger storage needs
4. Process files in chunks
5. Use S3 Select or S3 Batch Operations
6. Consider using EC2 or ECS for very large files
7. Implement pagination if processing results

### 33. How would you design a serverless API using Lambda?
**Answer:**
1. Use API Gateway as the front end
2. Create Lambda functions for business logic
3. Use Lambda Proxy Integration for request/response
4. Implement authentication (Cognito, Lambda Authorizers)
5. Store data in DynamoDB or RDS
6. Use Lambda Layers for shared code
7. Implement caching at API Gateway level
8. Use CloudWatch for monitoring
9. Implement proper error handling and retries
10. Use VPC if accessing private resources

### 34. How would you handle database connections in Lambda?
**Answer:**
1. Use RDS Proxy for connection pooling
2. Initialize connections outside handler
3. Reuse connections across invocations
4. Implement connection retry logic
5. Set appropriate timeout values
6. Use connection pooling libraries
7. Consider DynamoDB for serverless databases
8. Monitor connection metrics
9. Set reserved concurrency to prevent overwhelming database

### 35. Your Lambda function needs to call multiple external APIs. How do you optimize?
**Answer:**
1. Make parallel API calls using Promise.all() or async libraries
2. Implement caching for repeated requests
3. Use connection keep-alive
4. Set appropriate timeouts
5. Implement circuit breaker pattern
6. Use exponential backoff for retries
7. Consider API Gateway caching
8. Store frequently accessed data in DynamoDB
9. Use Step Functions for complex orchestration

---

## 💡 Pro Tips for Interviews

- Always explain the reasoning behind your answers
- Mention real-world use cases when possible
- Discuss trade-offs and alternative approaches
- Show awareness of costs and performance implications
- Demonstrate understanding of AWS Well-Architected Framework
- Be ready to draw architecture diagrams
- Practice explaining concepts in simple terms

---

**Good luck with your interview! 🚀**
