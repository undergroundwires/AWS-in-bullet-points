# Amazon Aurora

- Database engine part of [Amazon RDS](./06-02-01-data-databases-rds.md) family
  - It has significant architectural differences that set it apart from traditional RDS.
  - See  [Amazon RDS (vs Aurora)](./06-02-01-data-databases-overview.md#aurora-vs-rds)
- Compatible API for PostgreSQL / MySQL
- Other features: backup and recovery, industry compliance, push-button scaling, automated patching with zero downtime, advanced monitoring, routine maintenance
- **Capacity types**
  - **Provisioned**
    - Provision and manage the instance sizes
    - Define EC2 instance type for Aurora instances (not for Aurora serverless)
      - DB engine can be MySQL / Postgres with version
      - DB instance must have unique identifier unique in AWS region
  - [**Serverless**](#aurora-serverless)
    - Specify min and max resources and Aurora auto-scales
  - ***Multi-AZ deployment***
    - Through replicas in different zone
- **Monitoring**
  - Export log types to publish to Amazon CloudWatch (Audit, Error, General, Slow query)
  - Can enable [Enhanced monitoring](./06-02-01-data-databases-rds.md#enhanced-monitoring) for more metrics.
- You can handle parameters using
  - DB Parameter Group
  - DB Cluster Parameter Group
  - Option groups

## Aurora Pricing

- Aurora costs more than [RDS](./06-02-01-data-databases-rds.md) (20% more) but more efficient
  - See [Aurora vs RDS](./06-02-01-data-databases-overview.md#aurora-vs-rds)
- Does not have free tier.
- Billed per IO.
  - **Read IO**: Every database page read operation is 1 IO.
  - **Write IO**: Pushing transaction logs to shared storage for high availability.

## Aurora Event Notifications

- Infrastructure-related events:
  - E.g. Database failovers, backup completions, maintenance windows, parameter group changes
  - Use RDS event subscriptions
- 📝 Data modifications events
  - E.g. `INSERT`, `DELETE`, or `UPDATE`
  - Create a native function or a stored procedure that invokes an AWS Lambda function

## Aurora Performance

- 💡 Improve query performances
  - **Hash Joins**: If you need to join a large amount of data by using an equijoin.
  - **Asynchronous key prefetch (AKP)** Improve the performance of queries that join tables across indexes
- [Aurora Parallel Query](#aurora-parallel-query)
  - Distributes query computational load across thousands of CPUs for analytical workloads.
- [Shared storage architecture](#aurora-scalability)
  - Provides fast scaling and eliminates data duplication overhead.
- [Amazon RDS Proxy](#amazon-rds-proxy-for-aurora)
  - Reduces connection overhead and improves performance during traffic surges.

## Aurora DB Cluster

### Aurora DB Cluster Types

- **Multi-master clusters**
  - All DB instances have read-write capability.
- **Single-master clusters**
  - Has two instances types:
    1. **Primary DB instance**: master
       - Each Aurora DB cluster has one primary DB instance.
       - You can migrate from RDS MySQL/PostgreSQL to Aurora by creating Aurora read replicas and promoting them.
    2. **Aurora Replica**
       - Read-only replicas of master

### Aurora DB Cluster Endpoints

- **Reader endpoint**
  - Read replicas
- **Writer endpoint**
  - Master
  - If master fails, clients still talk to the endpoint and will be redirected to the right instance
- **Custom Endpoints**
  - 📝 Abstract connections point to specific DB instances in cluster
  - E.g. point analytics workload to higher memory capacity replicas
- 💡 Best practice: connect writer endpoint to write and reader endpoint to read.

## Aurora Availability

- **Automatic fail-over**:
  - Failover in Aurora in less than 30 seconds.
- Automated **backups** are always enabled.
  - Or use ***Backtrack*** to restore data at any point of time without using backups
  - You can also take **snapshots** & share with other AWS accounts without impacting performance.
    - User-created snapshots are retained after deleting the DB.
    - You can share snapshots across accounts.
- Faster RTO than other [RDS](./06-02-01-data-databases-rds.md) engines by making buffer cache of DB immediately available.
- **Multi-AZ**
  - By default with 6 copies (replicas) across 3 AZ:
    - 4 copies out of 6 needed for writes
    - 3 copies out of 6 need for reads
  - You can add additional MySQL or Aurora (recommended) replicas.
    - External MySQL databases as replicas are supported.
    - ❗ Global databases are recommended instead.
  - 💡 Can configure availability zone (specify where)
- You can have **multi-region** replicas.
- Self healing with peer-to-peer replication
  - Auto healing i.e. Aurora will recover if we lose one replica
- One Aurora Instance takes writes (master)
- Automated failover for master in less than 30 seconds
- **Failover behavior**
  - You can failover to any replica -> Secondary becomes the master, replication breaks.
    - You can assign priority for replicas, or the one which has same size as primary will be more prioritized.
  - **Single instance**:
    - Create a new DB in the same AZ, if it fails => try different AZ
    - RTO: 15 min
  - **Multi replicas**:
    - Flips the canonical name record (CNAME) for your DB Instance to point at the healthy replica
    - RTO: 30 seconds or less
- **Aurora Global databases**
  - Span multiple regions (global)
  - One primary region, one DR region (can be used for lower latency read)
  - < 1 second replica lag on average
  - 📝If not using Global Databases, you can create cross-region Read Replicas
    - 💡 Global databases are recommended: much faster replicas.
- [Amazon RDS Proxy for Aurora](#amazon-rds-proxy-for-aurora)
  - Automatically handles failover by preserving application connections during database failures.

## Aurora Scalability

- **Auto-expanding**
  - Aurora automatically grows in increments of 10 GB, up to 64 TB
- **Vertical Scaling**
  - **Storage Scaling**
    - No need to provision
    - Auto-scales up to 64 TB, in 10GB increments
    - Zero-impact to database performance when scaling.
  - **Compute resources**
    - Can happen during maintenance window
    - Or immediately (a few minutes of downtime)
- **Horizontal Scaling**
  - **Write Scalability**
    - **Multi-Master**: create multiple read-write instances across multi-AZ.
  - **Read Scalability**
    - Aurora can have 15 replicas (+ master)
      - MySQL has 5, and the replication process is faster (sub 10ms replica lag).
    - Support for Cross Region Replication.
    - All replicas and master uses a shared storage that's auto expandable.
    - Read Replicas can be global.
    - Read replicas can have their own read replicas.
    - 💡 You can use Aurora Replicas as failover targets.
    - Auto-scaling is done through **Auto Scaling policy**
      - Aurora Auto Scaling  adjusts the number of Aurora Replicas (reader DB instances) provisioned for an Aurora DB cluster
      - You can scale based on metrics e.g. average CPU utilization of Aurora Replicas
      - For target value e.g. 60% of CPU utilization, specify minimum & maximum replicas and cooldown periods (Seconds between scale-in and out), and enable/disable "Scale in"
- **Shared storage architecture**
  - Makes your data independent from the DB instances in the cluster.
  - You can add a DB instance quickly because Aurora doesn't make a new copy of the table data
  - Storage is striped across 100s volumes
  - You have auto-scaling for all replicas
  - All read replicas connects to **Reader Endpoint** (e.g. `dbname-ro.id.region.amazonaws.com`)
    - Reader endpoint is connection load balancing
    - Load balancing happens at the connection level not the statement level.
      - Anytime client connects to reader endpoint, the client will get connected to the one of the reader endpoint.
- [Amazon RDS Proxy for Aurora](#amazon-rds-proxy-for-aurora)
  - Pools and reuses database connections to handle traffic surges without overwhelming Aurora.

## Aurora Cloning

- Faster and more space-efficient than other techniques, such as restoring a snapshot.
- Flow:
  - Data is cloned initially
  - Aurora keeps a single copy of the data used by source and cloned cluster
  - Data is updated only during changes
- Use-cases: Test test environments using production data without risking data corruption
- Allows
  - cross-VPC cloning
  - cross-account cloning (via [AWS RAM](./01-04-aws-management-resource-groups-resource-access-manager-ram-license-manager-service-catalog.md#aws-resource-access-manager-ram))

## Aurora Security

- **Deletion protection**
  - You can't delete the database
  - Protects from accidental deletions
- **Aurora Encryption**
  - Encryption at rest using KMS
    - Automated backups, snapshots and replicas are also encrypted
  - Encryption in flight using SSL (same process as MySQL or Postgres)
  - ❗ You cannot encrypt an existing unencrypted database.
- **Aurora IAM**
  - 📝 Decides WHO can connect to database (replaces passwords)
  - ❗IAM does NOT work inside Database level:
    - Table/columns permissions are handled by database engine (MySQL/PostgreSQL)
- **Aurora Network Security**
  - Disable public access via VPC with security group.
- ❗ You cannot access the SSH into underlying instances directly by design.
  - Create a new database with encryption enabled and migrate your data instead.

## Aurora Serverless

### Aurora Serverless v1

- Auto-scales compute capacity based on application demand
- DB cluster pauses during inactivity, resumes automatically
- Measured in ACU (Aurora Capacity Units), billed per 5-minute increments
- ❗ Limited features: no read replicas, global databases, or custom endpoints

### Aurora Serverless v2

- 📝 **Current generation** with instant scaling (sub-second)
- Compatible with all Aurora features: read replicas, global databases, Multi-AZ
- Scales from 0.5 to 128 ACUs automatically
- Pay only for resources consumed per second
- 💡 Recommended over v1 for new workloads

## Aurora Parallel Query

- Ability to push down and distribute the computational load of a single query across thousands of CPUs in Aurora’s storage layer
- Good fit for analytical workloads requiring fresh data and good query performance, even on large tables.
- Can be enabled/disabled globally and on session level with `aurora_pq` parameter.
- Causes higher I/O costs as it scans all data.
- ❗ Not available for serverless, Backtrack-enabled instances, and non R-* instances.
- 💡 You can extend your own MySQL databases to Aurora
- You can use Aurora to scale reads for external MySQL databases
- You can use Aurora for DR with external MySQL databases

## Amazon RDS Proxy for Aurora

- Fully managed database proxy service that sits between applications and Aurora databases.
- 📝 Acts as an intermediary to manage database connections efficiently.
- Integrates seamlessly with Aurora clusters to enhance performance and reliability.
- Works also for, more details: [Amazon RDS Proxy](./06-02-01-data-databases-rds.md#amazon-rds-proxy)
