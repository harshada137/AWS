# AWS DynamoDB Notes

## Overview
Amazon DynamoDB is a fully managed NoSQL database service provided by AWS. It supports key-value and document data models, offering fast and predictable performance with seamless scalability.

## Key Features
- **Fully Managed:** AWS handles server provisioning, patching, and replication.
- **High Performance:** Single-digit millisecond latency at any scale.
- **Scalable:** Auto-scaling for throughput and storage.
- **Serverless:** No infrastructure management required.
- **Durable:** Data is automatically replicated across multiple Availability Zones (AZs).
- **Flexible Data Models:** Supports key-value and document formats (JSON).

## Core Concepts
1. **Table:** Collection of items (similar to rows in RDBMS).
2. **Item:** Single record in a table (similar to a row).
3. **Attributes:** Fields within an item (similar to columns).
4. **Primary Key:** Unique identifier for each item.
   - **Partition Key (Hash Key):** Required
   - **Sort Key (Range Key):** Optional, allows composite keys.
5. **Secondary Indexes:** Enable querying on non-primary key attributes.
   - **Global Secondary Index (GSI)**
   - **Local Secondary Index (LSI)**

## Data Consistency
- **Eventually Consistent Reads:** Default, may return stale data.
- **Strongly Consistent Reads:** Guarantees up-to-date data but higher latency.

## Read/Write Capacity Modes
1. **Provisioned:** Set specific read/write capacity units.
2. **On-Demand:** Auto scales based on traffic, pay per request.

## DynamoDB Streams
- Captures item-level changes (insert, update, delete).
- Can trigger AWS Lambda for real-time processing.

## Security
- **IAM Policies:** Control access.
- **Encryption:** At rest using AWS KMS.
- **VPC Endpoints:** Private connectivity without exposing to the internet.

## Use Cases
- Real-time applications (e.g., gaming leaderboards, messaging)
- IoT applications
- Serverless backends
- Session management