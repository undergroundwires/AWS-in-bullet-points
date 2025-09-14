# Data Querying

## S3 Query Options

| Attribute | [S3 Object Lambda](./06-01-03-data-s3-integrated-services-snowball-athena-storage-gateway-object-lambda-transfer.md#s3-object-lambda) | [Athena](./06-01-03-data-s3-integrated-services-snowball-athena-storage-gateway-object-lambda-transfer.md#amazon-athena) | [Redshift Spectrum](./06-03-01-data-analytics-warehousing-redshift.md#redshift-spectrum) |
| --------- | ---------------- | ------ | ----------------- |
| **Functionality** | Real-time data transformation with custom Lambda code | Query & extract data from S3 (ETL) | SQL queries on S3 using your Redshift cluster |
| **Operation Types** | GET, HEAD, LIST with transformations | Complex SQL queries (GROUP BY, JOIN, window functions) | ANSI SQL queries |
| **Data Scope** | Single object transformation | Multiple objects, cross-bucket queries | Multiple objects via Redshift cluster |
| **S3 Glacier Support** | ✓ (restored objects) | ✖ | ✓ |
| **Read compressed files** | ✓ | ✓ | ✓ |
| **Schema Required** | ✖ (ad-hoc transformations) | ✓ (AWS Glue Data Catalog) | ✓ (Redshift schema) |
| **Integrations** | **CloudFront**, Lambda functions, any S3-compatible application | **AWS Glue**, QuickSight, VPC Flow logs, ELB logs, [CloudFront](./04-06-02-networking-edge-cloudfront.md) & CloudTrail logs | S3, Redshift cluster |
| **Infrastructure** | Serverless (Lambda-based) | Serverless | Serverless |
| **Execution** | Synchronous (real-time) | Asynchronous (results to S3) | Query via Redshift |
| **Access Method** | Object Lambda Access Point ARN | AWS Console, API, JDBC/ODBC | Redshift SQL interface |
| **Pricing** | Lambda compute + requests + data returned + S3 requests | Per query: Data Scanned + *(data transfer + requests)* | Data scanned + S3 transfer & request + Redshift cluster |
| **When to use** | **Real-time transformations**, PII redaction, format conversion, image processing, content personalization | **ETL, Analytics, Logs**, Complex queries (ANSI SQL compliant) | **Data warehouse** queries when using Redshift cluster |

## Amazon QuickSight

- Cloud-based BI tool offerring.
- Helps deliver easy-to-understand insights.
- Connects to your data in the cloud and combines data from different sources including third-party.
- Integrations (from/into) include:
  - Redshift
  - [S3](./06-01-01-data-s3-overview.md)
  - [Athena](./06-01-03-data-s3-integrated-services-snowball-athena-storage-gateway-object-lambda-transfer.md#amazon-athena)
  - [Aurora](./06-02-03-data-databases-aurora.md)
  - [RDS](./06-02-01-data-databases-rds.md)
  - [IAM](./02-01-security-iam.md)

## Amazon OpenSearch Service

- A distributed search and analytics engine built on Apache Lucene.
- Provides ElasticSearch APIs
- Indexes & allows you to search any field, even partially matches in unstructured data.
- It's common to use OpenSearch Service as a complement to another database
- 💡 **Use cases**
  - Full-text search
  - Log analytics and monitoring
  - Real-time application monitoring
  - Security analytics
- **Features**
  - Multi-AZ deployment with dedicated master nodes
  - Built-in Kibana (OpenSearch Dashboards)
  - Integration with CloudWatch, S3, Kinesis
  - VPC support with fine-grained access control
- **Security**
  - IAM integration
  - Encryption at rest/in transit using KMS
  - VPC endpoints
  - [Cognito](./02-02-security-identity-federation-sts-saml-cognito-directory-service-identity-center.md#amazon-cognito)
- You can provision a cluster of instances
- Built-in integrations for data ingestion:
  - [Amazon Data Firehose](./06-03-02-data-analytics-streaming-kinesis-data-firehose-kafka-flink.md#amazon-data-firehose),
  - [AWS IoT Core](./12-other-services.md#aws-iot-core)
  - [Amazon CloudWatch Logs](./03-01-monitoring-cloudwatch-grafana.md#logs)
- Comes with OpenSearch Dashboards (visualization, successor to Kibana) - part of the OpenSearch ecosystem
- Example usage flow:
  - Send data in form of JSON docs using ElasticSearch API or ingestion tools (e.g. Logstash, Kinesis Data Firehose)
  - OpenSearch automatically stores the original document and adds searchable reference to index.
  - Search and retrieve using Elasticsearch API.
  - Can be combined with Kibana to visualize data and build interactive dashbaords.
- 🤗 Successor to *Amazon ElasticSearch Service* and *AWS ElasticSearch* (rebranded in 2021)
  - It was based on open-source OpenSearch and Elasticsearch
