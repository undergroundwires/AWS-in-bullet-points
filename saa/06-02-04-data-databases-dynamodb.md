# DynamoDB

- Fully managed, serverless, key-value NoSQL database
- Offers
  - built-in security,
  - continuous backups,
  - automated multi-region replication,
  - in-memory caching,
  - data import/export tools
- Serverless, provisioned capacity, auto scaling, on demand capacity
- Can replace ElastiCache as a key/value store
  - Advantage is that it's serverless e.g. no provisioning & maintenance
  - Not sub millisecond (as ElastiCache) performance but single digit (1-9 millisecond) performance.
- Read & Writes are decoupled: you can provision read and write capacity units
- **Consistency**
  - Reads can be eventually consistent or strongly consistent
    - Eventually consistent: accept stale data
    - Strongly consistent: get always newest data
  - *(Optionally)* **Transactions** for writes
    - All or nothing type of operations
    - Coordinated Insert, Update & Delete across multiple tables
    - ❗ Include up to 10 unique items or up to 4 MB of data
- Monitoring through CloudWatch
- ❗ Can only query on primary key, sort key or indexes
- 💡 Use cases:
  - Serverless applications development (small documents 100s KB)
  - Distributed serverless cache
  - Doesn't have SQL query language available
  - Has transactions capability
- [AWS DMS](./09-migrations-discovery-migration-hub-migration-evaluator-application-discovery-service-database-migration-service-application-migration-service-app2container-datasync.md#aws-database-migration-service) can be used to migrate to DynamoDB from Mongo, Oracle, MySQL, S3, etc...
- You can launch a local DynamoDB on your computer for development purposes.
- 💡 **Best Practices**
  - Keep item sizes small
  - If you are storing serial data in DynamoDB that will require actions based on data/time use separate tables for days, weeks, months
  - Store more frequently and less frequently accessed data in separate tables
    - Allows you to set appropriate provisioning levels independently for the tables
  - If possible compress larger attribute values
  - Store objects larger than 400KB in S3 and use pointers (S3 Object ID) in DynamoDB

## DynamoDB Availability

- Multi AZ in 3 data centers by default
- Highly available
- **Backup and Restore**
  - Point in time restore like [RDS](./06-02-01-data-databases-rds.md) without performance impact
  - ❗ On restored table, you need to manually setup the following: ASG, IAM, CloudWatch metrics/alarm, tags, stream & TTL settings.
- **Point-in-time recovery (PITR)**
  - Enables you to back up your table data continuously
  - Per-second granularity
  - Recovery period between 1 and 35 days
  - Protect against accidental writes and deletes
  - No negative impact on the performance or availability
  - Fully managed: automatically encrypted and catalogued

## DynamoDB Security

- Supports [VPC Endpoints](./04-02-networking-vpc.md#vpc-endpoints) for VPC access without Internet.
- Access fully controlled by IAM
  - Possible to set IAM permissions on table, row and attribute levels.
- Encryption at rest using KMS
- Encryption in transit using SSL / TTLS
- Auditing
  - CloudTrail tracks API calls (who, when and what action)
  - Can create CloudTrail Data Events to track more information during access.
    - E.g: `requestParameters: {"tableName": "MyTable"}` for table access.
  - Can use VPC Flow Logs (for network-level monitoring)
  - Can use AWS Config for data configuration changes (e.g. table alteration)
  - Auditing who, using another component (e.g. Lambda) accesses data:
    - Other resources get their own IAM role, so it would not show user by default.
    - Best practices:
      - *Use API Gateway for Lambda:*
        - Pass User Context Through Event from gateway to application layer.
        - Because the Lambda in the middle calling DynamoDB does not know by default who's invoking it.
      - Then data access (application) layer:
        - *X-Ray traces:* Use X-Ray for Request Tracing.
        - *Custom metrics:* Custom CloudWatch Metrics
        - *Application logs:* Write audit logs.

## DynamoDB Scaling

1. On-demand
   - Auto scales by Application Auto Scaling
     - **Scaling policy**
       - Whether to scale read or/and write capacity.
       - Minimum and maximum provisioned capacity unit
       - Supports *setting target utilization* with *target tracking*.
   - 2.5x more expensive than provisioned capacity (use with care!)
   - 💡 Good when:
     - Creating new tables with unknown workloads.
     - Unpredictable application traffic.
       - ❗ Auto scaling typically takes 5-15 minutes to respond
       - ❗ Provisioning for unpredictable traffic have under/over provisioning issues.
2. **Provisioned** RCU (read compute unit) & WCU (write compute unit)
   - They're decoupled can be increased / decreased individually
   - 💡 Allows you to control costs for predictable workloads.
   - 1 RCU
     - 1 strongly consistent read of 4 KB per second
     - 2 eventually consistent read of 4 KB per second
   - 1 WCU = 1 write of 1 KB per second
   - Throughput can be exceeded temporarily using ***burst credit***
     - If burst credit are empty, you'll get `ProvisionedThroughputException`.
     - It's then advised to do an exponential back-off retry

## DynamoDB Tables

- DynamoDB is made of **tables**
  - You don't create database, database is available and you just create tables
- Each table has a **primary key** (must be decided at creation time)
  - Primary key values are unique identifiers across all partitions
- Each table have optional ***secondary indexes***.
  - Efficient access to data with attributes other than the primary key
  - **Global secondary index**
    - = Partition key (optionally + sort key) that can be different from those on the base table.
    - Index can span all of the data in the base table, across all partitions
    - No size limitations: Has its own provisioned throughput settings for read + write separate from table.
  - **Local secondary index**
    - = Partition key same as the base table + different sort key.
      - Sort key e.g. timestamp to query for time ranges.
    - Every partition of a local secondary index is scoped to a base table partition that has the same partition key value.
    - ❗ The total size of indexed items for any one partition key value can't exceed 10 GB.
  - 💡 All indexes requires unique partition key and a sort key
    - ❗ So A limitation is e.g. when you need to perform an indexed lookup of records by time across multiple primary keys:
      - DynamoDB might not be the ideal service for you to use
      - You might need to utilize a separate table (either in DynamoDB or a relational store) to store item metadata that you can perform an indexed lookup against.
- Each table can have an infinite number of items (=rows)
- Each item has ***attributes*** (can be added over time - can be null)
- ❗ Maximum size of an item is 400 KB
- Data types supported are:
  - Scalar types: `string`, `number`, `binary`, `boolean`, `null`
  - Document types: `list`, `map`
  - Set types: `string set`, `number set`, `binary set`

## DynamoDB Streams

- Captures a time-ordered sequence of item-level modifications in any DynamoDB table.
  - E.g. create, update, delete
- Generates changelogs for every operation
  - Data rention: 24 hours
  - Apps can access this log and view data items as they appeared before and after they were modified in near real-time.
- Enables event-driven programming by integrating to **AWS Lambda**.
  - ❗ Must enable it to be able trigger a lambda.
- The log is created AFTER the action has taken place.
  - To subscribe BEFORE the action (before writing to database):
    - Use [Kinesis Data Streams](./06-03-02-data-analytics-streaming-kinesis-data-firehose-kafka-flink.md#amazon-kinesis-data-streams) for transforming data.
- Can implement cross region replication using Streams
  - Streams enable DynamoDB to get a changelog and use that changelog to replicate data across regions

## DAX: DynamoDB Accelerator

- Seamless cache for DynamoDB, no application rewrite
  - Application talks directly to DynamoDB accelerator with same connection string
- Writes goes through DAX to DynamoDB
- Micro second latency for cached reads & queries
- 📝 Solves Hot Key problem (too many reads)
- 5 minutes TTL for cache by default
- ❗ Up to 10 nodes in the cluster
- Multi AZ (3 nodes minimum recommended for production)
- Security with encryption at rest with KMS, VPC integration, IAM, CloudTrail...

## DynamoDB Global Tables

- Global service with region specific tables
- ❗ You must first enable **DynamoDB Streams**
- **Global Tables**
  - Multi region, fully replicated, high performance
- **Multi-Master**
  - Create multiple read-write instances per region
  - All changes are replicated to all tables.
  - Allows write-scaling.
- If data is updated from multiple regions
  - ***last writer wins*** to ensure eventual consistency
