# Big Data Streaming

## AWS Kinesis

- Managed alternative to Apache Kafka
- Use-cases
  - "Real-time" big data e.g. application logs, metrics, IoT, clickstreams.
  - Stream processing frameworks (Spark, NIFI, etc...)
- High availability through data replication to 3 AZ.
- Has 4 different products:
  - **Kinesis Streams** (data stream): Ingest & process streaming data
  - **[Firehose](#amazon-data-firehose)** (delivery stream): load streams into S3, Redshift, OpenSearch Service
  - **Kinesis Analytics**: perform real-time analytics on streams using SQL
  - **Kinesis video streams**: Process & analyze streaming media in real-time.
- 💡 Common integration: Feed *Kinesis Data Streams* -> Analyze data in real time using *Kinesis Data Analytics* -> Load streams into *Kinesis Data Firehose* -> S3/Redshift for long time retention.
- **Security**
  - Control access / authorization using IAM policies
  - Encryption in flight using HTTPS endpoints
  - Encryption at rest using KMS
  - Possibility to encrypt / decrypt data client side (harder)
  - [VPC Endpoints](./04-02-networking-vpc.md#vpc-endpoints) for VPC access without Internet.
  
## Comparison

| Attribute | [Data Streams](#amazon-kinesis-data-streams) | [Firehose](#amazon-data-firehose) | [Analytics](#amazon-kinesis-data-analytics) | [Video streams](#amazon-kinesis-video-streams) |
| -------- | -------- | --------- | --------- | ------------- |
| **Description** | Blob (max 1 MB) ingestion | Convert & load blob | Query engine | Video ingestion |
| **Throughput/Limits** | Per shard and message size/amount per read/write, max 1 MB blob | None: source data (max 1MB blob) | None: source stream | stream level (5TPS/H), connection-level (1/s), bandwidth MB/s, fragment-level |
| **Latency** | Real time | Near real time (60 secs latency) | Real time | Rendering + encryption (if stored) |
| **Data retention** | 1 day (default), up to 7 days | Retries during 24-hours then fails | Can create streams based on queries | Min 1 hour, max 10 years (in S3) |
| **Scaling** | Add shards | Auto-scales | Auto-scales | Auto-scales |
| **Concepts** | *Shards*, *Producers*, *Consumers* |  *Delivery stream*, *Record*, *Destination* | *Input*, *Code*, *Application* | *Video stream*, *Fragment*, *Producer*, *Consumer*, *Chunk* |
| **Use-cases** | Streams | Convert & load streaming data into e.g. Redshift, S3, OpenSearch Service, Splunk | Real-time analytics using SQL | Machine learning & analytics |

## Amazon Kinesis Data Streams

- Also known as *KDS*
- Collect and process large stream of data records in real time.
- Low latency streaming ingest at scale like Kafka.
- Use-cases:
  - Real-time processing (results)
  - Sending processed records to dashboards to generate alerts and change pricing.
- See [Data Firehose](#amazon-data-firehose) for storing and using data later.
- Data retention:
  - 1 day by default
  - Can go up to 7 days
- **Data**
  - The message is base64-encoded blob
  - ❗ Data (before encoding) cannot exceed 1 MB
  - Ability reprocess / replay data as data is not removed after message is handled.
  - Immutable: Once data is inserted in Kinesis, it cannot be deleted
  - Batching available or per message calls
    - 💡 Use batching with `PutRecords` API to reduce costs and increase throughput
- **Security**
  - You can enable server-side encryption with an AWS KMS master key
- **Monitoring**
  - You can capture ***shard level metrics*** with CloudWatch at additional cost
    - For e.g. *`IncomingBytes`*, *`IncomingRecords`*, *`OutgoingBytes`*, *`WriteProvisionedThroughputExceeded`*, *`ReadProvisionedThroughputExceeded`*.
- **SDKs**
  - Normal consumer (CLI, SDK, etc...)
  - Kinesis Client Library (in Java, Node, Python, Ruby, .NET)
    - Uses DynamoDB to checkpoint offsets
    - KCL uses DynamoDB to track other workers and share the work amongst shards
- **Shards**
  - One stream is made of many different shards
  - The number of shards can evolve over time (re-shard / merge)
  - Records are ordered per shard
  - Consumer consume from any shard
    - ***Producers*** -> **Stream** (***Shard*** 1 / ***Shard*** 2 / ***Shard*** 3) -> **Consumers**
    - Pub/sub through streams (topic)
    - Multiple applications can consume the same stream
    - You get sequence number (checkpoint offset, should be saved) back for each data added
  - 📝 Data can trigger real-time analytics by running:
    - [AWS Lambda](./05-04-compute-lambda.md)
    - Or *Amazon Managed Service for Apache Flink*
  - **Scaling**: Add shards -> Increased throughput
  - Pricing is per shard provisioned, can have as many shard as possible
  - **Partition key**
    - Partition key gets hashed to determine the shard ID
    - Ensures ordering in a shard and same key always goes to same shard.
      - 💡 Choose a partition key that's highly distributed
        - 📝 Helps preventing **hot partition** or **hot shard**
  - Throughput
    - ❗ 1 MB/s or 1000 messages/s at write PER SHARD
    - ❗ 2 MB/s at read PER SHARD
    - `ProvisionedThroughputExceeded` if you go over the limits
    - 💡 Solution
      - Use retries with exponential back-off
      - Increase shards (scaling)
      - Ensure your partition key is a good one e.g. you don't have a hot shard

## Amazon Kinesis Video Streams

- Managed service to stream video from connected to devices for AWS
- Stores, encrypts, indexes video data, and provides access via APIs.
- ***HTTP Live Streaming (HLS)*** enables you to playback video for live and on-demand viewing
- Uses Amazon S3 as the underlying data store.
- Data at rest using KMS, IAM roles, and data in transit using TLS.
- Concepts
  - **Video stream**: AWS resource that encrypts & time-stamps & stores.
  - **Fragment**: self-contained sequence of frames
  - **Source**: e.g. video-generating device, such as a security camera, a body-worn camera
  - **Consumer**: consume and process data in Kinesis video streams e.g. EC2 / AWS AI services / 3rd party.
  - **Chunk**: Stored data, consists of the actual media fragment

## Amazon Data Firehose

- Formerly known as *Amazon Kinesis Data Firehose*, or *Kinesis Firehose*.
- Fully managed data streaming service.
- Streams data to multiple consumers including:
  - AWS services e.g.[Amazon S3](./06-01-01-data-s3-overview.md), [Amazon Redshift](./06-03-01-data-analytics-warehousing-redshift.md), Amazon OpenSearch Service
  - External services e.g. Splunk, Snowflake custom HTTP endpoint (e.g. Datadog/Dynatrace)
- Can ingest data from
  - Integrated AWS services (e.g. [Data Streams](#amazon-kinesis-data-streams))
  - Custom sources using `PUT` API
- Fully managed service, no administration, automatic scaling
- Supports many data format (pay for conversion)
- Pay for the amount of data going through Firehose
- Use-case: Store and use later (near real-time or batch delivery)
  - See [Data Streams](#amazon-kinesis-data-streams) for real-time usage.
- Concepts
  - **Delivery stream**: Create & send data into it
  - **Record**: Data of interest your data producer sends to a delivery stream
    - ❗ Max size before base-64 encoding is 1000KB
  - **Destination**:
    - [S3](./06-01-01-data-s3-overview.md)
    - [Amazon Redshift](./06-03-01-data-analytics-warehousing-redshift.md)
    - Amazon OpenSearch Service
    - Splunk

## Amazon Kinesis Data Analytics

- 📝 Perform real-time analytics on Kinesis Streams using SQL
- Fully managed with auto scaling to match source stream throughput.
- Pay for actual consumption rate
- Can create streams out of the real-time queries

## Amazon Managed Streaming For Apache Kafka (MSK)

- Also known as *Amazon MSK*
- Stream data with a fully managed, highly available Apache Kafka service.
- Helps ingesting and processing streaming data in real time.
- Helps capture data, and process using Apache Zeppelin notebooks.
- Helps build real-time, centralized, and privately acessible data buses.

### Amazon Managed Service for Apache Flink

- Use-case: Query and analyze streaming data.
- Run stream processing applications.
- Helps transforming and analyzing streaming data in real time using Apache Flink and integrate apps with other AWS services.
- Fully managed: no servers/clusters to manage, no storage/compute to set up, pay-per use.
