# AWS DynamoDB Interview Questions & Answers

## Q1. What is Amazon DynamoDB?
**Answer:** A fully managed NoSQL database service that supports key-value and document data models, offering fast and predictable performance at any scale.

## Q2. What are DynamoDB's primary key types?
**Answer:**
- **Partition Key (Hash Key):** Unique identifier for each item.
- **Sort Key (Range Key):** Optional, allows composite keys for multiple items with the same partition key.

## Q3. Difference between GSI and LSI?
**Answer:**
- **GSI (Global Secondary Index):** Can have a different partition key and sort key from the main table. Query across all table data.
- **LSI (Local Secondary Index):** Same partition key as the main table, but different sort key. Limited to 5 per table.

## Q4. What are DynamoDB Streams?
**Answer:** Feature to capture item-level changes (insert, update, delete) for real-time processing via AWS Lambda or other consumers.

## Q5. Explain DynamoDB capacity modes.
**Answer:**
- **Provisioned:** Manually set read/write capacity units.
- **On-Demand:** Automatically scales with traffic; pay per request.

## Q6. How does DynamoDB handle consistency?
**Answer:**
- **Eventually Consistent Reads:** Default, may return stale data.
- **Strongly Consistent Reads:** Guarantees up-to-date data at the cost of higher latency.

## Q7. Can DynamoDB tables be encrypted?
**Answer:** Yes, DynamoDB supports encryption at rest using AWS KMS.

## Q8. What are common use cases for DynamoDB?
**Answer:** Real-time apps (gaming, messaging), IoT, serverless backends, session management.

## Q9. How do you query a DynamoDB table?
**Answer:** Using `Query` operation for primary key or secondary index, and `Scan` to read the entire table (less efficient).

## Q10. Difference between Scan and Query?
**Answer:**  
- **Query:** Efficient, uses keys to filter results.  
- **Scan:** Reads the whole table, consumes more read capacity.
