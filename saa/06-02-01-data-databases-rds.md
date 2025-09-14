# Amazon Relational Database Service (Amazon RDS)

- Managed [Aurora](./06-02-03-data-databases-aurora.md) / PostgreSQL / MySQL / MariaDB / Oracle / SQL Server
- 💡 Use cases: Relational datasets (RDBMS, OLTP), transactional SQL queries.
- Traditional database instances
  - Requires you to choose instance size (not automatic)
  - For fully managed alternative see [Aurora](./06-02-03-data-databases-aurora.md)
- Must provision an EC2 instance & EBS Volume type and size
  - You can reserve RDS Reserved Instances from 1 to 3 years.
- ***Maintenance***: Auto minor version upgrade, Maintenance window (when)
- ***DB parameter group*** acts as a container for engine configuration values that are applied to one or more DB instances.
  - Changing groups = recreate a new group & assign (you can copy existing)
- **MySQL & PostgreSQL engines**
  - MyISAM (storage engine)
    - Use for intense, full-text search capability,
    - Not recommended overall
  - InnoDB is recommended (better crash recovery)
    - Compatible with [Aurora](./06-02-03-data-databases-aurora.md)
- **Stopping a database**
  - Main use-case: Cost optimization
  - ❗ DB instance will restart automatically after 7 days by default.
  - Pricing:
    - No longer charged for DB instance hours
    - ❗ Still charged for provisioned storage, backup storage (snapshots, backups etc.)
    - 📝 Cheaper option: Save snapshot -> Terminate instance -> Restore from snapshot

## Monitoring

- **Monitoring dashboards**
  - CloudWatch monitors metrics from its hypervisor, very high level.

### Enhanced Monitoring

- Collects real-time metrics within the agent
- 📝 For each process or thread using CPU, memory, disk, their virtual size etc.
- Different from CloudWatch as CloudWatch gathers metrics from hypervisor, instead of specialized agent.
  - Offering a more accurate representation of resource usage at the OS level
- Pros
  - Smaller monitoring interval results
  - Pay only for monitoring that exceeds the free tier provided by Amazon CloudWatch Logs.
- Cons
  - Compute-intensive workload have more OS process activity to report and higher costs.

### Performance Insights

- Database performance tuning and monitoring feature for RDS
- Helps assessing the load on your database, and determine when and where to take action
- Visualizes with breakdowns for performance, database load such as SQL queries, uses, DB objects, wait events (I/O, locks, CPU).
- Offers recommendations for performance optimization

### RDS Event Notifications

- Delivers events
  - ❗ Amazon RDS doesn't guarantee the order of events sent in an event stream.
- Events are database instance-level.
  - They're operational-level changes on infrastructure level.
  - It does not cover database or data modifications like adding, removing, updating data.
    - However, [S3 Event Notifications](./06-01-01-data-s3-overview.md#s3-event-notifications) supports it.
  - E.g.
    - Automatic backups
    - Instance scaling
    - Maintenance schedules
    - Security-related events such as access control changes
- Can send events to:
  - Amazon SNS
    - Traditional approach where you create an RDS event notification subscription
  - EventBridge
    - More modern, flexible approach where EventBridge can be configured to send information to multiple targets

## RDS Event Integration

- Infrastructure events:
  - [RDS Event Notifications](#rds-event-notifications) → SNS or EventBridge
- **Data events**:
  - [Direct Lambda invocation](#rds-lambda-invocation) (PostgreSQL, Aurora only)
  - CloudWatch Logs + audit logs → CloudWatch subscription filter → Lambda
  - EventBridge scheduled rules → Lambda (polling pattern)

### RDS Lambda Invocation

- Direct invocation supported only for specific database engines
- Can invoke Lambda functions directly from the database using SQL functions
- Requires IAM permissions for the database to invoke Lambda

## RDS Security

### RDS Network Security

- Usually deployed within a private subnet, not in a public one.
- Network level
  - Are into **VPC**
  - **DB subnet group** allows you to specify set of subnets in your VPC for your instances.
  - **Security groups**
- Can allow public accessibility (outside VPC access with public IP)
  - Gets an endpoint like `sqldb.c2bqxjtofpnk.eu-west-3.rds.amazonaws.com`
  - I.e. does not & cannot have an IP address.
  
### RDS IAM

- IAM policies Controls WHO can connect to the database (authentication)
- Replaces passwords with temporary AWS credentials
- ❗ IAM does NOT work inside database level:
  - It's handle traditional database permissions
  - Database level = What they can do once connected
- ❗ You cannot access the SSH into underlying instances directly by design.
- To connect:
  1. Traditional username and password
  2. [IAM DB Authentication](#iam-database-authentication) (for MySQL & PostgreSQL & Aurora)

#### IAM Database Authentication

- 📝 Generates temporary authentication tokens via AWS APIs (no agent required)
- Works with any AWS service that has IAM roles (EC2, Lambda, ECS, etc.)
- Requires AWS SDK/CLI to generate tokens via `rds:GenerateDBAuthToken`
- Lifespan of an authentication token is 15 minutes (short-lived)
- 💡 Recommended over username & password
- ❗ Ensures SSL must be used when connecting to the database
- ❗ Application must call AWS API to generate token, then use token as password in standard DB connection

### RDS Encryption

- **Encryption at rest** capability with AWS KMS - AES-256 encryption
  - ❗ Can only be enabled when you first create the DB instance
    - 💡 You can however: unencrypted DB -> snapshot -> copy snapshot as encrypted -> create DB from snapshot
  - **Transparent Data Encryption**
    - TDE is an encryption type which encrypts the ***data*** and ***log files*** of a database.
      - It means encrypting databases both on the hard drive and consequently on backup media.
      - Key is stored in master database -> Protection of backup files as they don't have the DEK (***database encryption key***)
        - Restoring from back-up requires same DEK in the destination database.
      - Enabling TDE also encrypts your `tempdb`.
    - ❗ Only for Oracle or SQL Server DB instance only as its their technology.
      - Cannot be used on Postgres or MySQL but similar feature is KMS Encryption
    - TDE can be used on top of KMS - may affect performance.
- **SSL certificates** to encrypt data to RDS in flight
  - 📝To ***enforce*** SSL
    - *PostgreSQL*: `rds.force_ssl = 1` in the AWS RDS Console (Parameter Groups)
    - *MySQL*: Within the DB: `GRANT USAGE ON *.* TO 'mysqluser'@'%' REQUIRESSL;`
  - To ***connect*** using SSL
    - Provide the SSL Trust certificate (can be downloaded from AWS).
    - Provide SSL options when connecting
- **Encryption of [Read Replicas](#read-replicas)**
  - If in same region => encrypted using same key as the master instance
  - Different regions => you encrypt using the encryption key for that region

## RDS Scalability

- Vertical scaling with EC2 sizing
  - ❗ Resizing makes database temporarily unavailable (generally a few minutes).
    - 💡 For minimal downtime: resize reader instance and then promote it to its own DB.
  - Occur during the maintenance window if you don't choose to apply it directly.
  - **Amazon RDS Storage Types**
    - **General Purpose SSD**: Cost effective, can burst to 3.000 for limited time.
    - **Provisioned IOPS**: I/O-intensive workloads, particularly database workloads, that require low I/O latency and consistent I/O throughput.
    - **Magnetic**: for backward compatibility, lower storage, not recommended
- **Auto-scaling**
  - *For horizontal scaling*: Not available for read replicas
  - *For vertical scaling*: **RDS Storage Auto Scaling** automatically scales storage capacity in response to growing database workloads, with zero downtime.

### Read replicas

- Enables horizontal read scaling.
- Cannot be auto-scaled.
- Replicate master to replicas asynchronously
- Each read replica will have separate DNS endpoint.
- Improves read performance
- Within AZ, Cross AZ or Cross Region
- Replicas can be promoted to their own DB
  - ❗ Breaks replication with the read replica.
- Can have own Multi-AZ copies.
- MySQL & MariaDB read replicas can have read replicas
- You can migrate to Aurora by creating Aurora Read Replica (single click).
- ❗ Must enable back-ups to enable read replica (not Aurora).
  - The replica is built by 1st restoring from the backup, and then only replicating blocks / record of changes since the backup was taken
- ❗ You can create up to 5 read replicas
- ❗ Not available for SQL server
- ❗ `SELECT` (=read) only kind of statements (not `INSERT`, `UPDATE`, `DELETE`)
- ❗📝 Read replicas do not help with disaster recovery
  - You cannot failover to a read replica, use Multi-AZ instead

## RDS Availability

- **Point in Time Restore**: Continuous backups and restore to specific timestamp
  - **RDS Backups**: Automatically enabled
    - **Automated backups**
      - Daily full snapshot of the database
      - Capture transaction logs in real time -> ability to restore to any point in time by applying transaction logs to the latest snapshot
      - Retention between 1 to 35 (❗ max) days (default: 7 days)
      - 💡 If you want to copy database: Restore from a snapshot, the database will have schemas and ready & will be quicker than big create query.
    - **DB Snapshots**
      - Manually triggered by the user
      - Retention of backup for as long as you want
      - When you delete an instance you can choose whether to create a final snapshot of the DB instance.
        - 💡 Should be set as default with [CloudFormation](./07-05-resiliency-infrastructure-as-code-cloudformation-cdk-sam.md#cloudformation).
    - ❗ I/O may be briefly suspended while the backup process initializes (typically under a few seconds), and you may experience a brief period of elevated latency.
    - Restored DBs will always be a new RDS instance with a new DNS endpoint.
    - You can restore up to the last 5 minutes.
- **Backtrack** rewinds the DB cluster to the time you specify
  - Does not replace point-of-time restore to back-ups
  - You can easily undo mistakes.
- **Multi regions** for DR (Disaster Recovery).

### Multi AZ

- Seamless sync replication of master DB to standby replica in another AZ
  - You cannot read from it, for it see [read replicas](#read-replicas).
- One DNS name - automatic app failover to standby
- Not used for scaling but for DR.
- 📝 When rebooting you can select to fail-over to other AZ.
- **Multi AZ vs Read Replicas**

  | Attribute | Multi-AZ Deployments | [Read Replicas](#read-replicas) |
  | --------- | -------------------- | ------------- |
  | **Replication** | Synchronous replication, Durable | Asynchronous replication, Scalable |
  | **Accessibility** | 👎 Only primary instance is active | 👍 Accessible and can be used for read scaling |
  | **Backups** | 👍 Automated backups are taken from standby (no I/O on primary) | 👎 No backups configured by default |
  | **Zone** | Always span two Availability Zones within a single Region | Can be within an Availability Zone, Cross-AZ, or Cross-Region |
  | **DR** | 👍 Automatic failover to standby when a problem is detected | 👎 Can be manually promoted to a standalone database instance |

- **Single AZ -> Multi AZ**
  - AWS does: snapshot primary instance, create new standby instance from snapshot in different AZ, configure synchronous replication between primary and snapshot.

### Amazon RDS proxy

- Manages pool of established connections to RDS databases instances
- Makes apps more scalable, more resilient to database failures and more secure
- Allows apps to pool and share connections
  - Reduces connections that DB exhausts database memory and compute
  - Improves database efficiency and application scalability.
- Database access can be managed via AWS Secrets Manager and AWS IAM.
- See also [Amazon RDS Proxy for Aurora](./06-02-03-data-databases-aurora.md#amazon-rds-proxy-for-aurora).
- **Availability**
  - Automatically connects to standby DB instance during failover.
  - Preserves application connections during database failures.
  - Reduces connection disruption when Aurora fails over to replicas.
- **Scalability**
  - Pools and shares database connections across applications.
  - Reuses connections to avoid memory and CPU overhead.
  - Handles unpredictable traffic surges without overwhelming the database.
  - Queues or throttles connections when pool is full.
  - Protects against connection oversubscription.
  - Creates read-only proxy endpoints for load balancing across Aurora read replicas.
- **Security & Efficiency**
  - Enforces IAM authentication for database access.
  - Stores credentials securely in AWS Secrets Manager.
  - Reduces credential processing overhead.
  - Handles secure connection establishment on behalf of applications.
- 💡 Use-cases:
  - High-traffic web applications
  - Microservices with variable connection patterns
  - Applications requiring zero-disruption failover

## RDS Custom

- ❗ You cannot access underlying operating system of RDS
- 📝 Amazon RDS Custom allows you to access operating system and database environment.
- Allows installing custom patches, configuring file systems and deep databse settings
- Services include: RDS Custom for Oracle, RDS Custom for SQL Server
- ❗ You bear most or all of the responsibility for database management
- 💡 Use-case: legacy, custom, and packaged business applications
