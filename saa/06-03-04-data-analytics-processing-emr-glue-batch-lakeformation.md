# Data Processing

## Amazon EMR

- Helps create Hadoop clusters (Big Data) to analyze and process vast amount of data.
- EMR stand for Elastic Map Reduce
- Big data solution for
  - petabyte-scale data processing
  - interactive analytics
  - machine learning using open-source frameworks such as Apache Spark, Apache Hive and Presto.
- Helps you run Apache Spark, Hive, Presto and other big data workloads easily.
- Includes serverless option (Amazon EMR serverless) without managing clusters.
- Output destinations:
  - AWS services (native integration)
    - E.g. [Amazon Redshift](./06-03-01-data-analytics-warehousing-redshift.md#amazon-redshift), [DynamoDB](./06-02-04-data-databases-dynamodb.md#dynamodb), [RDS](./06-02-01-data-databases-rds.md#amazon-relational-database-service-amazon-rds), [S3](./06-01-01-data-s3-overview.md), [OpenSearch Service](./06-03-03-data-analytics-querying-quicksight-opensearch-redshift-spectrum-athena.md#amazon-opensearch-service)
  - External systems (possible but more complex)
    - E.g. external databases/data warehouses, on-premises, third-party cloud
- **Map Reduce**
  - Massive Parallel Processing technique
  - Steps
    1. **Map** with user defined mappers: Get data as key value
    2. *Sort* handled by framework
    3. **Reduce** with user defined reducers: Aggregates data
  - **Example**
    - Documents have words, **map** counts animal words, framework sorts, and **reduce** sums total occurrences of animal words.
    - Map => `cat:10, dog:3` & `cat:24, dog:5`
    - Reduce => `cat: 34 , dog: 8`
- The clusters can be made of hundreds of EC2 instances.
- EMR takes care of all the provisioning and configuration
- Also supports Apache Spark, HBase, Presto, Flink...
- Auto-scaling and integrated with Spot instances for lower price.
- Nodes
- **Master node**: One node that manages cluster e.g. track status & monitor health.
  - 💡 You can SSH into master node and from there to task nodes..
- **Core nodes**: Data (HDFS), compute.
  - **Input splits**: Smaller data chunks of input data split by Hadoop
    - Controls number of Mappers in MapReduce
  - **Blocks**: Physical splitting of data
- **Task nodes**: Optional, only compute
  - Usually runs on spot instances
  - Amazon handles if they get executed through running master operations in core nodes.
- How it works:
  1. Upload your data and processing application to S3
  2. Configure and create your cluster by specifying data inputs, outputs, cluster size, security settings, etc.
     - Launch modes:
       - Cluster: Long running
       - Step execution: Do analysis & shut-down cluster
     - Instance size & number of instances
     - Select EC2 key pair & IAM roles to EC2 instances
  3. Monitor the health and progress of your cluster. Retrieve the output in S3
- 💡 **Use cases**: data processing, machine learning, web indexing, big data
- **EMR vs Glue**
  - Use EMR for big data processing (petabyte scale)
  - Use [Glue](#aws-glue) for simple-moderate ETL jobs, data cataloging an schema discovery
- 🤗 Formerly known as *Amazon Elastic MapReduce*

## AWS Glue

- 📝 Fully managed ETL (Extract, Transform & Load) service
  - Automates time consuming steps of data preparation for analytics.
  - E.g. sorting the data by client timestamp, compressing and storing in a read optimized format.
- Serverless, pay as you go, fully managed
- Components:
  - [ETL](#aws-glue-etl-jobs) jobs created via code or GUI ([AWS Glue Studio](#aws-glue-studio))
  - [DataBrew](#aws-glue-databrew) to clean and normalize data without code
  - [Crawler](#aws-glue-crawler)
    1. Crawls data sources and identifies data formats (schema inference)
    2. Creates [Glue Data Catalog](#aws-glue-data-catalog)
  - [Data Catalog](#aws-glue-data-catalog) allows queries and ETL jobs
- Discover, categorize, save in a catalog from connected sources, e.g:
  - [Amazon Aurora](./06-02-03-data-databases-aurora.md)
  - [RDS](./06-02-01-data-databases-rds.md)
  - [Redshift](./06-03-01-data-analytics-warehousing-redshift.md)
  - [S3](./06-01-01-data-s3-overview.md)
- Sinks:
  - [S3](./06-01-01-data-s3-overview.md)
  - [Redshift](./06-03-01-data-analytics-warehousing-redshift.md)
  - And more ..
- E.g. scenario =>
  - Kinesis (ingest & process) -> S3 (through [Amazon Data Firehose](./06-03-02-data-analytics-streaming-kinesis-data-firehose-kafka-flink.md#amazon-data-firehose)) -> Spark job on AWS Glue (sort, optimize, crawl and catalog) -> Batch process catalog using Amazon EMR (Elastic MapReduce), query with Athena.

### AWS Glue ETL Jobs

- Traditional ETL jobs you create in [AWS Glue Studio](#aws-glue-studio) or via code
- AWS Glue provisions Apache Spark
- Can run Spark and PySpark ETL jobs.
- Uses **job bookmarks** to persist state information and prevent the reprocessing of old data
- It generates customize Apache Spark code automatically

#### AWS Glue Studio

- Graphical interface to create/run/monitor jobs in AWS Glue.
- Compose transformation workflows, run on AWS Glue.
- E.g. gather, transform and clean data.

### AWS Glue Data Catalog

- Serves as an index to the location, schema and runtime metrics of your data.
- Contains metadata (definition & schema) of the Source Tables
- 💡 Use-cases:
  - Can be used by [Amazon EMR](#amazon-emr)
  - Helps ETL (extract, transform load) jobs in Glue.
  - Helps data cataloging for data warehouse or data lake.

### AWS Glue Crawler

- Populates [AWS Glue Data Catalog](#aws-glue-data-catalog) with tables from multiple data stores.
- Primary method used by most AWS Glue users.
- Can crawl multiple data stoers in a single run.
- Upon completion, Creates or updates one or more tables in your Data Catalog.
- Runs through data to infer schemas and partitions from e.g.
  - [S3](./06-01-01-data-s3-overview.md)
  - [Redshift](./06-03-01-data-analytics-warehousing-redshift.md)
  - [RDS](./06-02-01-data-databases-rds.md)

### AWS Glue Databrew

- Visual data preperation tool to clean and normalize data without writing code.
- Includes prebuilt transformations to automate data preparation tasks
- DataBrew runs your recipes against your datasets
  - **DataBrew Datasets**
    - References to data sources
    - E.g. [Data Catalog](#aws-glue-data-catalog), [S3](./06-01-01-data-s3-overview.md)
  - **DataBrew jobs**
    - Not [Glue ETL jobs](#aws-glue-etl-jobs)
    - Runs your visual data preparation recipes

## AWS Batch

- Managed HPC (high performance computing)
- Batch plans, schedules, and executes your batch computing workloads using Amazon EC2 and Spot Instances.

## AWS Lake Formation

- Centrally govern, secure, and share data for analytics and machine learning.
- Centralizes permissions management of your data
- Makes it easier to share data across your organization and externally
- [Amazon S3](./06-01-01-data-s3-overview.md) forms the storage layer for Lake Formation
