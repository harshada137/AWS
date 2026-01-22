# AWS Lambda Practical Guide

## What Are We Creating?

In this practical, we are **creating an AWS Lambda function** using the **AWS Management Console**.

AWS Lambda is a **serverless compute service** that allows you to run code **without provisioning or managing servers**. You simply write your code, upload it to Lambda, and AWS automatically takes care of:

* Server management
* Scaling
* High availability
* Execution

In this practical, we will:

* Create a Lambda function
* Write a simple Lambda function code
* Test the function
* Understand where Lambda is used in real projects

---

## Prerequisites

Before starting this practical, ensure:

* You have an **AWS account**
* You can access the **AWS Management Console**
* Basic understanding of **Python** or **Node.js** (Python used here)

---

## Step 1: Login to AWS Management Console

1. Open browser and go to **AWS Console**
2. Sign in using your AWS credentials
3. From the search bar, type **Lambda**
4. Click on **AWS Lambda**

---

## Step 2: Create a New Lambda Function

1. Click on **Create function**

2. Choose **Author from scratch**

3. Fill the details:

   * **Function name**: `my-first-lambda`
   * **Runtime**: Python 3.x
   * **Architecture**: x86_64 (default)

4. Under **Permissions**:

   * Choose **Create a new role with basic Lambda permissions**

5. Click **Create function**

---

## Step 3: Write Lambda Function Code

After function creation, scroll to **Code source** section.

Replace existing code with the following:

```python
def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'body': 'Hello from AWS Lambda!'
    }
```

Click **Deploy** to save changes.

---

## Step 4: Test the Lambda Function

1. Click on **Test**
2. Choose **Create new test event**
3. Event name: `test-event`
4. Use default JSON
5. Click **Save**
6. Click **Test** again

### Expected Output:

* Status: **Succeeded**
* Response:

```json
{
  "statusCode": 200,
  "body": "Hello from AWS Lambda!"
}
```

---

## Step 5: Understand Execution Details

After execution, you can see:

* Execution duration
* Memory usage
* Logs in **CloudWatch**

To view logs:

1. Go to **Monitor** tab
2. Click **View logs in CloudWatch**


---

## Conclusion

In this practical, we successfully:

* Created an AWS Lambda function
* Wrote and deployed code
* Tested the function
* Understood real-world use cases

This practical is a **foundation step** for building **serverless applications on AWS**.

---

✅ Practical Completed Successfully
