# S3 Integrated Services

## AWS Snowball Edge

- Physical data transport to/from S3.
- Pay per data transfer job
- Snowball Edge add computational capability (e.g. Lambda) to the device
  - You can use as *local storage only* if e.g. your region doesn't have lambda or for durability & capacity.
- Two options:
  - Storage optimized
  - Compute optimized
- Can run many AWS services for processing on the go.
  - E.g. EC2, Lambda, EKS Anywhere
- 📝💡 Use it if data takes longer than a week to upload
  - E.g. >2TB for 45 Mbps (T3: 269 days), >5TB 100 Mbps (120 days), >60TB 1000 Mbps (12 days).
- Use cases:
  - Large data cloud migrations
  - Image collation
  - IoT capture
  - Machine Learning
- Flow
  1. Request devices from AWS Snow Family Management Console
     - AWS Snow Family Management Console → Plan your job (Import into S3, Export from S3, Local compute and storage only) → Choose shipping & device variant → Configure compute options → IAM Role → KMS encryption → SNS notifications  2. Install the Snowball client on your servers
  2. Install Snowball Edge client (or use AWS OpsHub)
  3. Connect device using manifest file and unlock code, transfer data
  4. Ship back device (E-ink label auto-updates)
  5. Data imported into your S3 bucket
  6. Device completely wiped (NIST 800-88 standards)
  7. Tracking via SNS, console, job completion reports
- 🤗 AWS Snow Family had other members, all discontinued:
  - 🤗 AWS Snowball (original) without edge processing capabilities [DISCONTINUED 2024]
  - 🤗 AWS Snowcone was small device up to 8TB [DISCONTINUED 2024]
  - 🤗 AWS Snowmobile was truck with millions of TBs [DISCONTINUED 2024]

## AWS Storage Gateway

- 📝 Allows you to expose S3 to on-premises.
- Bridge between on-premise data and cloud data in S3
- 💡 Use cases: disaster recovery, backup & restore, tiered storage
- ***Flow***
  - Can be installed on Hyper-V and VMware.
  - You then create gateway on Console
- 📝 Three types of Storage Gateway:
  - [File gateway](#amazon-s3-file-gateway) -> S3
  - [Volume gateway](#volume-gateway) -> EBS
  - [Tape Gateway](#tape-gateway) -> Glacier
- **Security**: Your data is encrypted at rest by default in S3.
- Flow
  - AWS Console -> Storage Gateway -> Choose gateway type -> Select host platform (VMware / hyper-v / EC2) -> IP address of gateway VM

### Amazon S3 File Gateway

- Store & update files as objects in S3 asynchronously as you change them.
- Ownership/permissions and timestamps are stored in S3 in the user-metadata of the object.
- Accessible using the NFS (network file system) and SMB protocol
- Supports S3 standard, & S3 IA, S3 One Zone IA & Glacier.
- Bucket access using IAM roles for each File Gateway
- Most recently used data is cached locally in the file gateway -> Low latency access
- Can be mounted on many servers
- 📝 File access / NFS => File Gateway (backed by S3)

### Volume Gateway

- Block storage in Amazon S3 with point-in-time backups as compressed Amazon EBS snapshot.
- Allows you to view S3 bucket as mountable volume
- Block storage using iSCSI protocol backed by S3
- Two options:
  - **Cached volumes**: most frequently data are cached on premise on the volumes
  - **Stored volumes**: entire dataset is on premise, scheduled backups to S3 as EBS snapshots
- 📝 Volumes / Block Storage / iSCSI => Volume gateway (backed by S3 with EBS snapshots)

### Tape Gateway

- *Input:* Your existing tape-based processes
- *Output:* As virtual tapes (using Virtual Tape Library, i.e. VTL) in S3 or Glacier.
- Uses iSCSI which is interface & networking standard for linking data storage facilities.
- 📝 VTL Tape solution / Backup with iSCSI => Tape gateway (backed by S3 and Glacier)

## Amazon Athena

- Provides serverless, scalable SQL queries directly on stored S3 data.
- Uses SQL language to query the files
- Has a JDBC / ODBC driver that allows you to connect BI tools to it.
- Charged per query and amount of data scanned
- Supports CSV, JSON, ORC, Avro and Parquet (built on Presto)
- 💡 Use cases: Business intelligence / analytics / reporting, analyze & query VPC Flow Logs, ELB Logs, CloudTrail trails, etc..
- 📝 Analyze data directly on S3 -> Use Athena
- Usage
  1. Go to Athena on Console
  2. Create database with a query
  3. Create table & link to a bucket
     - It's a link so you don't pay anything to have tables
  4. You can then write SQL queries against the table
- Security
  - Allows you to
    - query encrypted data in S3
    - write encrypted results back.
  - Supports
    - server-side encryption
    - client-side encryption
- Replaces older *S3 Select* and *Glacier Select*
  - Use [Amazon Athena](#amazon-athena) instead

## S3 Object Lambda

- Can run custom Lambda on GET/HEAD/LIST requests to S3
- Allows real-time data transformation
- No storage overhead - transforms data in-flight
- Access via: Object Lambda Access Points (not direct bucket)
  - Direct access: `Application → s3://my-bucket/file.csv`
  - Object Lambda access: `Application → Object Lambda Access Point ARN → Lambda Function → Transformed Data`
- **Timeout**: 60 seconds max
- 💡 **Use cases**:
  - PII redaction for analytics/dev environments
  - Format conversion (XML→JSON, CSV→Parquet)
  - Image resizing/watermarking
  - Dynamic content generation
- **CloudFront integration**: Use as origin for CDN transformations

## AWS Transfer

- Fully managed AWS service that you can use to transfer files into and out of:
  - [S3](./06-01-01-data-s3-overview.md)
  - [EFS](./05-01-06-compute-ec2-ebs-efs-instance-store.md#amazon-elastic-file-system)
- Multi-AZ, regional service
- 💡 **S3 Use cases**:
  - Data lakes in AWS for uploads from third parties
  - Subscription-based data distribution
  - Internal transfers
- 💡 **EFS Use cases**:
  - Data distribution
  - Supply chain
  - Content management
  - Web serving applications
- Supports:
  - SFTP
  - FTPS
  - FTP
  - AS2
  - Browser-based transfers
