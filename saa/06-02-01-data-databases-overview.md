# Overview of AWS Databases

## OLTP vs OLAP

| Parameters | OLTP | OLAP |
| --------- | ---- | ---- |
| **Meaning** | Online transaction processing | Online Analytics processing |
| **Functionality** | Online database modifying system | Online database query management system |
| **Purpose** | Real time business operations | Analysis of business measures by category and attributes |
| **Characteristic** | Large numbers of short online transactions | Large volume of data |
| **Style** | Fast response time, low data redundancy and is normalized | Created uniquely to integrate different data sources for building a consolidated database |
| **Design** | Application oriented e.g. Banking, ticket booking, SMS delivery etc. | Subject oriented e.g. for sales / marketing / purchasing etc. |
| **Query type** | Standardized and simple | Complex queries involving aggregations |
| **Operation** | Read/write | Only read and rarely write |
| **Tables** | Normalized | Not normalized |
| **Method** | DBMS | Data warehouse |
| **AWS Services** | [Aurora](./06-02-03-data-databases-aurora.md), [RDS](./06-02-01-data-databases-rds.md) | Redshift |

## Choose right database

| Category | Question | Deeper |
| -------- | -------- | ------- |
| Workload | Read-heavy, write-heavy, or balanced workload? | • Throughput needs? High or low throughput? • Will it change, does it need to scale or fluctuate during the day? |
| Size | How much data to store and for how long? | • Will it grow? • Average object size? • How are they accessed? Security needs? |
| Durability | Data durability (e.g. for a week or forever)? | • Source of truth for the data? |
| Latency | Latency requirements? | • Concurrent users? |
| Model | Data model? | • How will you query the data? Primary key? Joins? • Structured? Semi-structured? |
| Schema | Strong schema or more flexibility? | Reporting? Search? RDBMS / NoSQL? |
| Licensing | License costs? | Switch to Cloud Native DB such as [Aurora](./06-02-03-data-databases-aurora.md) |

## Database Types

- All Database types come with backup / restore feature.

| Type | Databases | Use cases |
| ---- | --------- | --------- |
| RDBMS | [RDS](./06-02-01-data-databases-rds.md), [Aurora](./06-02-03-data-databases-aurora.md) | SQL, OLTP, Joins, Data in tabular form |
| Key-value | DynamoDB (also Document ≈JSON), ElastiCache | No joins & more performance and scalability |
| Object Store | S3, Glacier | Big objects (S3), backups / archives (Glacier) |
| Data Warehouse | Redshift, [Athena](./06-01-03-data-s3-integrated-services-snowball-athena-storage-gateway-object-lambda-transfer.md#amazon-athena) | SQL Analytics, BI, OLAP (Redshift) |
| Search | OpenSearch (JSON) | Free text, unstructured searches |
| Graphs | Neptune | Relationships between data. |

## Comparison

| Database | Type | Use-case | Operations | Security | Reliability | Performance | Cost |
| -------- | ---- | -------- | ---------- | -------- | ----------- | ----------- | ---- |
| [**RDS**](./06-02-01-data-databases-rds.md) | RDBMS | Relational datasets (RDBMS, OLTP), transactional SQL queries. | • Small downtime if failover, maintenance, scaling replicas / EC2, restore EBS • Changes requires application changes | • AWS -> OS / EC2 • Users -> KMS, SG, IAM policies, user auth with e.g. IAM, SSL | Multi AZ feature with auto-failover | • Up to 5 read replicas • Depends on EC2, EBS, read replicas • no auto-scaling | Pay per hour based on provisioned EC2 and EBS |
| [**Aurora**](./06-02-03-data-databases-aurora.md) | RDBMS | Same as RDS with less maintenance and more performance & flexibility | Less operations than RDS e.g. auto scaling storage | Same as RDS | • Multi AZ • More HA than RDS • Serverless option for more reliability | • 5x than RDS • Up to 15 read replicas | • Pay per hour for EC2 + storage • Lower than Oracle • Higher than RDS |
| [**Redshift**](./06-03-01-data-analytics-warehousing-redshift.md) | Columnar | • Analytics • Data Warehousing  | Similar to RDS | IAM, VPC, KMS, SSL (similar to RDS) | highly available, auto healing features | • Massively Parallel Query Execution • compression • scale to PBs of data | Pay per node provisioned, 10% of cost vs other warehouses |
| [**ElastiCache**](./06-02-05-data-databases-elasticache-redis-memcached.md#elasticache) | Key-value | • caching, user sessions, leaderboard, distributed states, pub / sub messaging, recommendation data | Same as RDS | Same as RDS (but auth through Redis Auth) | Clustering (sharding), Multi-AZ | Sub-millisecond, in-memory, read replicas for sharding | Pay per hour based on EC2 and storage usage. |
| [**DynamoDB**](./06-02-04-data-databases-dynamodb.md) | Key-value & document | • no SQL with transactions • serverless development • cache • small objects | • no operations needed • auto scaling capability • serverless | through IAM policies, KMS encryption, SSL in flight | • Multi AZ • Backups | • Single digit (1-9 millisecond) • DAX for read caching • Performance doesn't degrade if usage scales | Pay per provisioned capacity and storage usage (can use auto-scaling) |
| [**S3**](./06-01-01-data-s3-overview.md) | Key-value object store | • big objects • static files • website hosting | no operations needed | • IAM • Bucket Policies • ACL • Encryption (Server/Client) • SSL | • 99.999999999% durability / 99.99% availability • Multi AZ • Cross-region replication | • Scales to thousands of read / writes per second • Transfer acceleration with [CloudFront](./04-06-02-networking-edge-cloudfront.md) • Multi-part for big files | Pay per • Storage usage • Network cost: bandwidth to transfer and retrieve data • Requests number |
| [**Athena**](./06-01-03-data-s3-integrated-services-snowball-athena-storage-gateway-object-lambda-transfer.md#amazon-athena) | S3 query engine | • Serverless *(one time)* queries on S3 • Log analytics • Lightweight queries | no operations needed | IAM + S3 security | • managed service • uses presto engine • highly available | Queries scale based on data size | pay per query / per TB of data scanned |
| [**Neptune**](./06-02-06-data-databases-other-databases-documentdb-elasticsearch-neptune-timestream.md#amazon-neptune) | Graph | High relationship data e.g. social networking, knowledge graphs (wikis) | similar to RDS | IAM, VPC, KMS, SSL (similar to RDS) + IAM Authentication | Multi-AZ, clustering with read replicas | best suited for graphs, clustering to improve performance | pay per node provisioned (similar to RDS) |

### RDS vs DynamoDB Availability and Scalability

| | [Aurora](./06-02-03-data-databases-aurora.md) | [DynamoDB](./06-02-04-data-databases-dynamodb.md) |
| - | ----- | ------- |
| Roll-back impact | No downtime, more IO (uses snapshots), requires re-establishing connection after DB change | No downtime, no IO (uses Backup and Restore), requires re-establishing connection after DB change |
| Multi-master | Across AZ in single region (even with global database) | Only across regions with global tables |
| Multi-reader | Across AZ & regions | Automagically, only can set-up with DAX |
| Failover to replicas | Any replica | Any replica |

## RDS Migration

- (Manually) Native SQL DB migration:
  - From SQL Server => upload `.bak` to S3 => restore from `.bak` with SQL statement in [RDS](./06-02-01-data-databases-rds.md)
- (Automatically) Use [Database Migration Service](./09-migrations-discovery-migration-hub-migration-evaluator-application-discovery-service-database-migration-service-application-migration-service-app2container-datasync.md#aws-database-migration-service)

## Aurora vs RDS

- [RDS](./06-02-01-data-databases-rds.md) is a managed service for traditional databases like MySQL and PostgreSQL
  - [Aurora](./06-02-03-data-databases-aurora.md) is a proprietary, high-performance, and highly available database built on top of [RDS](./06-02-01-data-databases-rds.md).
- Choose
  - Aurora for high availability, global scale, or variable workloads
  - RDS for lower prices, need specific engines (Oracle/SQL Server), stable/predictable workloads
- Similarities:
  - Both use parameter groups and option groups
  - Both offer [enhanced monitoring](./06-02-01-data-databases-rds.md#enhanced-monitoring) and [Performance Insights](./06-02-01-data-databases-rds.md#performance-insights)
  - Both can use RDS Proxy ([for RDS](./06-02-01-data-databases-rds.md#amazon-rds-proxy), [for Aurora](./06-02-03-data-databases-aurora.md#amazon-rds-proxy-for-aurora)) for connection management
  - Both provide automated backups and maintenance
- Differences:

  | Feature                      | RDS                                                                 | Aurora                                                                 |
  |------------------------------|---------------------------------------------------------------------|------------------------------------------------------------------------|
  | **Database Engines**         | MySQL, PostgreSQL, MariaDB, Oracle, SQL Server                      | MySQL-compatible, PostgreSQL-compatible (proprietary engine)           |
  | **Storage Architecture**     | EBS volumes (single AZ) with Multi-AZ standby                       | Distributed storage across 3 AZs (6-way replicated)                   |
  | **Replication**              | Asynchronous read replicas (high lag)                               | Synchronous storage replication + sub-10ms replica lag                 |
  | **Failover Time**            | 60-120 seconds                                                      | <30 seconds                                                            |
  | **Storage Scaling**          | Manual (with RDS Storage Auto Scaling)                              | Automatic (10GB increments to 64TB)                                    |
  | **Read Replicas**            | Up to 5 (MySQL)                                                     | Up to 15 + Multi-master option                                         |
  | **Serverless**               | ❌ Not available                                                     | ✅ Aurora Serverless v1/v2                                              |
  | **Backtrack**                | ❌ Not available                                                     | ✅ Rewind DB without backups                                            |
  | **Database Cloning**         | ❌ Not available                                                     | ✅ Instant, space-efficient cloning                                     |
  | **Global Distribution**      | Cross-region read replicas only                                     | ✅ Aurora Global Database (<1 sec lag)                                  |
  | **Performance**              | Baseline performance                                                | Up to 5x faster than MySQL for specific workloads                      |
  | **Cost**                     | Lower base cost (~20-30% cheaper)                                   | Higher base cost but better efficiency                                  |
  | **Connection Management**   | RDS Proxy available                                                 | ✅ RDS Proxy with enhanced failover                                     |
  | **Advanced Features**        | Engine-specific (TDE for Oracle/SQL Server)                         | Parallel Query, Hash Joins, AKP, ML integration                        |
  | **High Availability**       | Multi-AZ deployment (standby replica)                              | Built-in Multi-AZ with self-healing storage                             |
  | **Backup/Recovery**          | Point-in-time recovery (35-day retention)                           | Point-in-time + Backtrack + Instant cloning                            |
  | **IAM Authentication**       | ✅ MySQL/PostgreSQL                                                 | ✅ MySQL/PostgreSQL                                                    |
  | **Encryption at Rest**       | ✅ KMS-based                                                         | ✅ KMS-based                                                           |
  | **Enhanced Monitoring**      | ✅                                                                  | ✅ + Aurora-specific metrics                                            |
  | **Performance Insights**     | ✅                                                                  | ✅                                                                     |
  | **Automated Patching**       | ✅                                                                  | ✅                                                                     |
  | **Multi-AZ Support**         | ✅                                                                  | ✅ (Built-in by default)                                               |
  | **Read Replica Scaling**     | ❌ Auto-scaling not available                                       | ✅ Auto-scaling policies supported                                     |
  | **Cross-Region Replication**| ✅                                                                  | ✅                                                                     |
  | **Deletion Protection**      | ✅                                                                  | ✅                                                                     |
