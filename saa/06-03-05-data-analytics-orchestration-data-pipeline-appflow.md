# Data Orchestration

- [AWS DataSync](./09-migrations-discovery-migration-hub-migration-evaluator-application-discovery-service-database-migration-service-application-migration-service-app2container-datasync.md#aws-datasync) can also handle scheduled data synchronization for ongoing workflows.

## AWS Data Pipeline

- Move data between different AWS compute and storage services, as well as on-premises data sources, at specified intervals.
- Output to
  - [Amazon S3](./06-01-01-data-s3-overview.md)
  - [Amazon RDS](./06-02-01-data-databases-rds.md)
  - [Amazon DynamoDB](./06-02-04-data-databases-dynamodb.md)
  - [Amazon EMR](./06-03-04-data-analytics-processing-emr-glue-batch-lakeformation.md#amazon-emr).
- A pipeline consists of
  - Data nodes for source or destination storages (s3, mysql, dynamodb...)
  - E.g. import, copy, export.
- Differences from ETL
  - ETL pipelines are a subset of data pipelines.
  - ETL may modify data, data pipeline won't.
  - ETLs end in warehouses, built for warehousing.

## Amazon AppFlow

- Fully managed integration service
- 📝 Automates bi-directional data flows betwen SaaS apps and AWS services.
- SaaS (input) examples: Salesforce, SAP, Google Analytics, Facebook Ads, ServiceNow.
- AWS (output) examples: [S3](./06-01-01-data-s3-overview.md), [Redshift](./06-03-01-data-analytics-warehousing-redshift.md)
- Features:
  - Incremental transfers: Only transfers changed data (not full datasets each time)
  - Data preparation with transformations (e.g. masking), partitioning, and aggregation.
  - Can run on a schedule, in response to a business event in real time, or on demand
  - Automates registrations of schema with [AWS Glue Catalog](./06-03-04-data-analytics-processing-emr-glue-batch-lakeformation.md#aws-glue-data-catalog) to discover and share data with AWS
