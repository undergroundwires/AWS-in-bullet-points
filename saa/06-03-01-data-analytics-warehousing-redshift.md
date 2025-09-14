# Amazon Redshift

- 📝 Redshift = Data warehouse = Analytics / BI
- Uses SQL to analyze structured/semi-structured data across warehouses, databases, data lakes.
- Columnar database e.g. stores columns of data instead of rows.
  - Good for OLAP (online analytical processing), see RDBMS for OLTP (online transaction processing)
  - Makes compression much easier
- For analytics and cloud data warehousing
  - Run complex analytical queries on exabytes of data
  - No warehouse infrastructure needed to run
  - Uses optimized machine learning and AWS hardware
- Used to pull in very large and complex data sets.
- Has a SQL interface for performing the queries
- Heavily modified version of PostgreSQL
- **Scaling**
  - Can scale to PBs of data
  - From 1 node to 128 nodes, up to 160 GB of space per node
    - ***Leader node***: for query planning, results aggregation
    - ***Compute node***: for performing the queries, send results to leader
- It's a **Massively Parallel Query Execution (MPP)** database.
  - Coordinated processing of a single task by multiple processors, each processor using its own OS and memory and communicating with each other using some form of messaging interface.
  - Data and query is automatically distributed and balanced.
- Pay as you go based on the instances provisioned (not serverless)
- 💡 Choose right cluster size & use as much as you can
- 💡 Great to use with BI tools on top of it such as AWS Quicksight, SQL Server Reporting Services, Tableau integrate with it
- Data is loaded from different DBs/services including:
  - S3
  - DynamoDB
  - [DMS](./09-migrations-discovery-migration-hub-migration-evaluator-application-discovery-service-database-migration-service-application-migration-service-app2container-datasync.md#aws-database-migration-service)
- Good practice -> Create read replica from [RDS](./06-02-01-data-databases-rds.md) and pull the data from the read replica and load it into Redshift
- **Copy between regions**: Take snapshot => Copy snapshot to new region => Create cluster from snapshot
- **Security**: VPC / IAM / KMS & CloudWatch Monitoring
  - **Redshift Enhanced VPC Routing**: COPY / UNLOAD goes through VPC
  - Encrypted in transit using SSL
- **Pricing**: Pay per Compute Node hours (not leader node), backup and data transfer (only within a VPC)
- **Cluster subnet group** allows you to specify set of subnets in your VPC for your instances.

## Redshift Spectrum

- Serverless service to perform queries directly against S3 (no need to load)
- Redshift Spectrum vs [Amazon Athena](./06-01-03-data-s3-integrated-services-snowball-athena-storage-gateway-object-lambda-transfer.md#amazon-athena)
  - Athena for ad hoc data discovery and SQL querying
  - Redshift Spectrum for
    - More complex queries and scenarios
    - A large number of data lake users want to run concurrent BI and reporting workloads

## Availability

- No multi AZ, only available in 1 AZ
  - Snapshots can be deployed to new AZ for e.g. DR.
- Backups are enabled by default, max 35 days.
- Attempts to maintain at least three copies of your data.
  - Original, replica on the compute nodes, and a backup in S3.
- Have functionality for **Automated Cross-Region Snapshot**.
