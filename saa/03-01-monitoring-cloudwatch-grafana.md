# CloudWatch

- Collects and manages operational data (monitoring/performance)
- Can monitor Compute (EC2, ASG, ELB, Route53 health checks..), Storage & Content Delivery (EBS, Storage)...

## Metrics

- Collection of related data points in time ordered structure
  - E.g. CPU utilization, network utilization, disk reads/writes, status checks.
- **Namespaces**
  - Container for metric data.
  - E.g. `AWS/EC2` for EC2, can also be custom.
- Retention: 14 days
  - **Extended retention** offers up to 15 months.
- Metrics have
  - timestamps (e.g. `2025-08-24T15:30:30Z`)
  - metric value (e.g. `90.3` for CPU)
  - dimension (e.g. instance ID and instance type for EC2 granularity)
- **Dimension**
  - Attribute of a metric (instance id, environment, etc..)
  - Up to 10 dimensions per metric
- **EC2 Detailed Monitoring**
  - 📝 EC2 instance metrics have basic metrics for "every 5 minutes"
    - With detailed monitoring (for a cost), you get data "every 1 minute"
  - 💡 Use detailed monitoring if you want to more prompt scale your ASG
  - ❗ EC2 memory usage is by default not pushed
    - 💡 Must be pushed from inside the instance as a custom metric with e.g. CloudWatch agent.
- **Custom Metrics**
  - Possibility to define and send your own custom metrics to CloudWatch
    - E.g. for RAM utilization, disk storage usage.
  - Must be pushed from inside the instance as a custom metric by installing agent
    - E.g. CloudWatch agent
  - Ability to use dimensions (attributes) to segment metrics
    - E.g. `Instance.id`, `Environment.name`
  - Metric resolution
    - Standard: 1 minute
    - High resolution: up to 1 second (`StorageResolution` API parameter)
      - Higher cost
  - API call `PutMetricData`
  - 💡 Use exponential back off in case of throttle errors
- ❗ CloudWatch itself does not have a native export feature that will send data periodically to S3.
- **Metric Filters**
  - Define the terms and patterns to look for in log data as it is sent to CloudWatch Logs
  - Filters only publish the metric data points for events that happen after the filter was created.
    - Filters do not retroactively filter data.
  - E.g. count log events, count occurances of a term, count HTTP 404 codes, count HTTP 4xx codes

## Dashboards

- Can create CloudWatch dashboard of metrics
- In console you can monitor & access dashboards
- Can be ***regional*** or ***global*** (e.g. include graphs from different regions)
- You can change the time zone & time range of the dashboards
- You can setup automatic refresh (10s, 1m, 2m, 5m, 15m)
- Pricing
  - 3 dashboards (up to 50 metrics) for free
  - 3$ per dashboard per month afterwards
- Types are:
  - *Line*: compare metrics over time
  - *Stacked area*: Compare the total over time
  - *Number*: instantly see the latest value for a metric
  - *Text*: Free text with markdown formatting
  - *Query results*: Explore results from Logs Insights
- ***Sharing dashboards:***
  - Can be shared using IAM permissions using e.g.:
    - `cloudwatch:GetDashboard` for specific dashboard
    - `cloudwatch:GetMetricData` for metrics
    - and other permissions resource specific information
  - For external users
    - Can provide third-party SSO to access (requires [Amazon Cognito](./02-02-security-identity-federation-sts-saml-cognito-directory-service-identity-center.md#amazon-cognito) integration)
    - Directly to their e-mail address
      - Up to 5 people
      - Each will provide passwords
  - ❗All who can access the dashboard can query all CloudWatch metrics and the names/tags of all EC2 instances in the account

## Logs

- Applications can send logs to CloudWatch using the SDK
- CloudWatch Logs **metric filters** can evaluate CloudTrail logs for specific terms, phrases or values.
  - I.e. values are not always from CloudWatch Metrics, but can be generated from Logs e.g. HTTP errors.
- CloudWatch can collect log from *Elastic Beanstalk*, *ECS*, *AWS Lambda*, *VPC Flow Logs*, *API Gateway*, *CloudTrail*, *CloudWatch log agents*, *Route53* and more.
  - CloudWatch Log Agents
    - Install on EC2 machines `sudo yum install -y awslogs`
      - Ensure EC2 has IAM permissions to write to CloudWatch
    - Configure `/etc/awslogs/awslogs.conf` for logs (errors etc.)
    - Configure `/etc/awslogs/awscli.conf` for region
    - Start service with `service awslogsd start`
- CloudWatch logs can go to:
  - Batch exporter to [S3](./06-01-01-data-s3-overview.md) for archival
  - 📝 Stream to [OpenSearch](./06-03-03-data-analytics-querying-quicksight-opensearch-redshift-spectrum-athena.md#amazon-opensearch-service) cluster for further analytics
  - Stream to Lambda
- You need to store logs in 2 things:
  - ***Log groups***: arbitrary name, usually representing an application
  - ***Log stream***: instances within application / log files / containers
- Can define ***log expiration policies*** (never expire, 30 days, etc..)
  - You pay for data retention in CloudWatch
- Using the AWS CLI we can tail CloudWatch logs
  - To see e.g. how application is behaving in real time
- **Security**
  - ❗ To send logs to CloudWatch, make sure IAM permissions are correct!
  - Encryption of logs using KMS at the Group Level
- CloudWatch Logs can use ***filter expressions***
  - E.g. find a specific IP inside of a log
  - ***Metric filter***s can be used to trigger alarms
    - E.g. if specific IP appears you can trigger an alarm
- **CloudWatch Logs Insights**
  - Log analytics service for CloudWatch
  - Can be used to query logs and add queries into CloudWatch Dashboards
  - Pay for the queries you run
- **Subscription Filters**
  - Provide real-time access to CloudWatch Logs
  - Can deliver data to
    - [Amazon Kinesis stream](./06-03-02-data-analytics-streaming-kinesis-data-firehose-kafka-flink.md#amazon-kinesis-data-streams)
    - [Amazon Data Firehose stream](./06-03-02-data-analytics-streaming-kinesis-data-firehose-kafka-flink.md#amazon-data-firehose)
    - [AWS Lambda](./05-04-compute-lambda.md)
    - [Amazon OpenSearch Service](./06-03-03-data-analytics-querying-quicksight-opensearch-redshift-spectrum-athena.md#amazon-opensearch-service)
  - Allows custom processing, analysis, or loading to other systems

## Alarms

- Used to trigger notifications for any metrics
- You can set up ***billing alarms*** to be triggered after the account charges go over a certain threshold.
- Alarms invokes **action**s such as:
  - **EC2 Actions**: e.g. restart EC2.
  - **SNS Notifications**: email, SMS, etc.
  - **Auto Scaling**: triggers Auto Scaling policies.
- Various options (sampling, %, max, min, etc...)
- Alarm states e.g.:
  - `OK`
  - `INSUFFICIENT_DATA`
  - `ALARM` (being triggered)
- Period:
  - Length of time in seconds to evaluate the metric
  - 📝 High resolution custom metrics
    - Decreases as metrics age: 1 sec (for 3 hours), then 1 minute (for 15 days), 5 minute (for 63 days), 1 hour for 15 months.
  - E.g. `NetworkOut < 2,000,000` for 1 data points (EC2) within 5 minutes
- Data points
  - Represents the values of that variable over time
  - E.g. if period is 5 minutes and data points is three then the alarm will trigger after 15 minutes of being condition met

## CloudWatch Events

- Creates a JSON document with change information.
- Can trigger to Lambda functions, SQS/SNS/Kinesis Messages
- **Event Rule**
  - Types
    - ***Schedule***: Notifications that'll be triggered on demand
    - ***Event Pattern***: React to service doing something e.g. CodePipeline state changes.
  - Targets: e.g. lambda function, EC2 `StopInstances` API call, SNS, SQS, ECS Task, Event bus in another AWS account...
- [EventBridge](./08-01-02-event-broadcasting-sns-eventbridge.md#amazon-eventbridge) vs CloudWatch Events
  - [EventBridge](./08-01-02-event-broadcasting-sns-eventbridge.md#amazon-eventbridge) is evolution of CloudWatch.
    - Extends it capabilities with third-party integrations, custom buses, schema registry.
  - CloudWatch Events: "When my EC2 instance stops, trigger a Lambda"
  - [EventBridge](./08-01-02-event-broadcasting-sns-eventbridge.md#amazon-eventbridge): "Route events between Salesforce, my app, and AWS services"

## Amazon Managed Grafana  

- Managed service for Grafana dashboards and visualizations
- Connects to multiple data sources (CloudWatch, Prometheus, X-Ray, etc.)
- Advanced visualization capabilities beyond CloudWatch dashboards
- Team collaboration and user management features
- 💡 Use cases: Complex dashboards, multi-source data visualization
