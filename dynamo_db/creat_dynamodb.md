# Steps to Create DynamoDB Table

## Using AWS Management Console

1. **Sign in** to AWS Management Console.
2. Navigate to **Services → DynamoDB → Tables**.
3. Click **Create table**.
4. Enter **Table name** and **Primary key**:
   - Partition key (required)
   - Sort key (optional)
5. Configure **Table settings**:
   - Read/write capacity mode (On-Demand or Provisioned)
   - Enable encryption (default: AWS owned)
6. (Optional) Enable **Auto Scaling** for provisioned mode.
7. (Optional) Add **Secondary Indexes**:
   - Global Secondary Index (GSI)
   - Local Secondary Index (LSI)
8. Click **Create table**.
9. Wait until status shows **Active**.
10. Use **Items tab** to add data or connect via **AWS CLI/SDK**.

## Using AWS CLI

```bash
aws dynamodb create-table \
    --table-name MyTable \
    --attribute-definitions \
        AttributeName=Id,AttributeType=S \
    --key-schema \
        AttributeName=Id,KeyType=HASH \
    --billing-mode PAY_PER_REQUEST
