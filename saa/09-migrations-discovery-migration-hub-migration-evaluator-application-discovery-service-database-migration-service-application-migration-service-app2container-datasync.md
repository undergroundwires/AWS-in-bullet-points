# AWS Migrations

## Portfolio Management

### AWS Migration Hub

- Discover the tools that you need to simplify your migration and modernization
- Provides central location to collect server and application inventory data for the assesment, planning, and tracking AWS migrations.
- Helps discovers apps to determine whether they can be migrated
- Provides modernization strategy recommendations

### Migration Evaluator

- Helps build data-driven business case for AWS
- Provides the insights you need to help define next steps in migration journey.
- Provides visibility into multiple cost-effective cloud migration scenarios.
- Provides insights on reusing existing software licensing to further reduce costs.
- Works by installing Agentless Collector to conduct discovery and upload exports.
- Helps analyzing current state, defining target state, developing migration readiness plan with projected cloud costs.

### AWS Application Discovery Service

- Discover on-premises server inventory and behavior plan cloud migrations
- Helps you plan your migration to AWS cloud by collecting usage and configuration data about on-premises services and databases
- Integrated with:
  - [AWS Migration Hub](#aws-migration-hub)
    - Migration Hub simplifies migration status information into a single console.
    - Helps you group discover servers into applications, and track migrations tatus of each application.
  - AWS Database Migration Service Fleet Advisor
    - Also known as *DMS fleet Advisor*
    - Help asses migrations options for database workloads.
- Discovered data
  - Stored at Migration Hub in same region.
  - Can be exported as Excel or AWS analysis tools such as Athena or QuickSight

## Migration Execution

### AWS Database Migration Service

- Also known as *AWS DMS*
- Managed migration and replication service for databases
- Migrate from legacy or on-premises databases and data warehouses to managed cloud services.
- Minimal downtime and zero data loss.
- 📝 Can do:
  - **Full Load:** One-time complete data migration
  - **Change Data Capture (CDC):** Ongoing replication of changes after initial migration
  - **Full Load + CDC:** Complete migration followed by continuous synchronization
- Supported directions:
  - **Cloud to cloud**
    - E.g. you can stream to [Amazon Kinesis Data Streams](./06-03-02-data-analytics-streaming-kinesis-data-firehose-kafka-flink.md#amazon-kinesis-data-streams) through AWS Database Migration Service
  - **On-premises to cloud**
    - You can migrate SQL databases
      - 💡 Instead: native SQL DB migration through `.bak` file is recommended.
      - 💡 Only recommended if your database cannot be offline while back-up is created/copied and restored.
  - **On-premises to on-premises**
    - E.g. upgrade a minor version with on-prem SAP ASE to on-prem SAP ASE
- Migration products:
  - From:
    - ***On-premises/EC2/Azure***: Oracle, SQL Server, Azure SQL, PostreSQL, MySQL, SAP ASE, MongoDB, S3, DB2
    - ***AWS Native***: [RDS](./06-02-01-data-databases-rds.md) & [Aurora](./06-02-03-data-databases-aurora.md) & S3
  - To:
    - ***AWS-native***: [RDS](./06-02-01-data-databases-rds.md), S3, DynamoDB, Redshift, Kinesis Data Streams, Elasticsearch, DocumentDB
    - ***On-premises/EC2***: Oracle, SQL Server, PostgreSQL, MySQL, SAP ASE
- Examples:
  - Oracle to Amazon Aurora MySQL-Compatible Edition
  - MySQL to RDS for MySQL
  - Microsoft SQL Server to Aurora PostgreSQL-Compatible Edition
  - MongoDB to DocumentDB (with MongoDB compatibility)
  - Oracle to Redshift and S3
- **Schema Conversion Tool**
  - E.g. from NoSQL to SQL, SQL to NoSQL or NoSQL to NoSQL.
- Supports SSL connections for both source and target endpoints
  - Certificates managed through DMS console or API
- **Data Extractor**:
  - Runs on-prem, pulls data from DB, uploads to S3 so other DBs can pull it from S3.
- 💡 Other use-cases:
  - Classic to VPC
  - Data warehouse to Redshift
  - Consolidate shards into [Aurora](./06-02-03-data-databases-aurora.md)
  - Archive old data, migrate from NoSQL <=> SQL or NoSQL <=> NoSQL.
- **Pricing**
  - Ingress into AWS Database Migration Service is free
  - Egress to [RDS](./06-02-01-data-databases-rds.md) and EC2 in same AZ is also free.

### AWS Application Migration Service

- Simplifies application migration and modernization
- Can help migrate e.g. SAP, Oracle, SQL server, VMware vSphere, Hyper-V
- Supports both third-party/AWS to AWS and on-premises to AWS migrations
- Helps migrate EC2 across regions, AZ or accounts.
- Features:
  - Set-up cross-region DR
  - Convert Windows MS-SQL BYOL to AWS licenses
  - Upgrade Windows Server versions
- 🤗 Successor to *AWS SMS (AWS Server Migration Service)* [DISCONTINUED 2022]

### AWS App2Container

- CLI tool for migrating and modernizing Java and .NET apps to container format.
- Analyzes and builds an inventory of apps running in bare metal / VMS / EC / cloud
- Helps migrating and modernizing legacy applications
- Provides
  - CloudFormation required for compute, network and security.
  - CI/CD pipelines for AWS DevOps services

### AWS DataSync

- Fully managed data transfer service.
- 📝 Automates and accelerates moving data between on-premises and AWS.
- Can copy data between NFS shares, SMB shares, HDFS, S3, AWS Snowcone, EFS, FsX
- **Sources:**
  - On-premises: NFS, SMB/CIFS file systems
  - AWS:
    - [S3](./06-01-01-data-s3-overview.md)
    - [EFS](./05-01-06-compute-ec2-ebs-efs-instance-store.md#amazon-elastic-file-system)
    - [FSx](./05-01-06-compute-ec2-ebs-efs-instance-store.md#amazon-fsx)
  - **Other clouds**: GCP, Azure, Oracle, DigitalOcean, Wasabi, Backblaze, Cloudflare.
- **Destinations:** AWS storage services (S3, EFS, FSx)
- 💡 Common Use Cases:
  - Data migration to AWS (lift-and-shift)
  - Disaster recovery and backup
  - Data lake ingestion
  - Hybrid workflows (keeping on-premises and AWS in sync)
- **Task Modes:**
  - **Enhanced mode**:
    - S3-to-S3, other clouds-to-S3 transfers
    - Virtually unlimited objects, parallel processing, higher performance
  - **Basic mode**: All other transfers. Sequential processing, 50M files/objects/directories limit per task
- **Key Features:**
  - **Cross-region transfers**: Sync EFS to EFS or FSx to FSx across regions
  - **Multi-cloud migration**: Move data from Google Cloud, Azure, or other providers to AWS
  - **Enhanced mode**: Agent-less transfers from other clouds with higher performance
  - **Scheduling**: One-time or recurring transfers
  - **Bandwidth throttling**: Control network usage during transfers
  - **Data integrity**: Built-in verification and retry mechanisms
  - **Encryption**: In-transit and at-rest encryption
  - **Incremental transfers**: Only copies changed data after initial sync
  - **Manifests**: Specify exact files/objects to transfer (useful for large datasets)
- **Agent Deployment:**
  - **Enhanced mode**: No agent required for cloud-to-AWS transfers
  - **Basic mode**: Agent required for on-premises and some cloud transfers
  - **AWS-to-AWS**: No agent required
  - **Agent specs**: VMware ESXi, KVM, Hyper-V, or EC2 instance (≥m5.2xlarge)
- 💡 More cost-effective than running custom scripts or third-party tools for large-scale data transfers
- ❗ Different from [Storage Gateway](./06-01-03-data-s3-integrated-services-snowball-athena-storage-gateway-object-lambda-transfer.md)
  - [DataSync](#aws-datasync) is for one-time/scheduled transfers
  - Storage Gateway provides ongoing hybrid access.
