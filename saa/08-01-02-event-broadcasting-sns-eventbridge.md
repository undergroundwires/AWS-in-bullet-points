# Event Broadcasting

## Amazon SNS (Simple Notification Service)

- Fully managed Pub/Sub service for A2A, A2P messaging
  - A2A: High-throughput, push-based, many-to-many messaging
    - E.g. Amazon SQS, Amazon Kinesis Data Firehose, AWS Lambda, custom HTTPs endpoints
  - A2P: Send messages to customers
    - E.g. SMS texts, push notifications, e-mail
- Event producer only sends message to one topic
- As many event receivers (subscriptions) as we want to listen
- Different from SQS where you have direct integration between listeners & senders.
- **Retention**: ❗ You have to process the data right away or it's lost
- Flow
- ***Event producer*** publish messages to a ***SNS topic***
  - ❗ Up to 100,000 topics (soft limit)
- ***Subscriptions*** listen a specific ***SNS topic***
  - ❗ Up to 10,000,000 subscriptions per topic (soft limit)
  - Each subscriber get all the messages but can filter.
- **Subscribers** can be
  - SQS
  - HTTP/HTTPS (with delivery retries)
  - Lambda
  - Emails, SMS messages, Mobile Notifications
- High availability with managed replication to 3 AZ.
- Pay as you go pricing
- **Integrations**: SNS integrates with a lot of Amazon Services
- Some services can send data directly to SNS for notifications
- E.g. CloudWatch for alarms, Auto Scaling Groups notifications, Amazon S3 on bucket events [CloudFormation](./07-05-resiliency-infrastructure-as-code-cloudformation-cdk-sam.md#cloudformation) (upon state changes => failed to build etc.) ...
- Publish types
  - **Topic Publish** (within your AWS server - using the SDK)
    1. Create a topic
    2. Create a subscription (or many)
    3. Publish to the topic
  - **Direct Publish** (for mobile apps SDK)
    1. Create a platform application
    2. Create a platform endpoint
    3. Publish to the platform endpoint
       - Works with Google GCM, Apple APNS, Amazon ADM
- 📝**Fan out (SNS -> SQS)**
  - Push once in SNS, receive in many SQS
    - E.g. *Buying Service -> SNS Topic ->
      1. SQS Queue 1 (-> Fraud service)
      2. SQS Queue 2 (-> Shipping service)
  - Good because:
    - 💡 No data loss (in SNS you need to handle message right away)
    - Ability to add receivers later
    - SQS allows for delayed processing and retries of work
    - May have many workers on one queue and one worker on the other queue
    - In only SQS a message is not guaranteed to be delivered to all consumers

## Amazon EventBridge

- Serverless event bus/router for application components
- Helps building event-driven applications
- Routes events between AWS services, SaaS applications, and custom applications
- Allows sending events from one account to other
  - By deploying eventbridge on both accounts to establish communication
- Evolution of [CloudWatch Events](./03-01-monitoring-cloudwatch-grafana.md#cloudwatch-events) with enhanced capabilities
  - [CloudWatch Events](./03-01-monitoring-cloudwatch-grafana.md#cloudwatch-events) still exists
- Broadcasts events to all targets:

  ```txt
    Event Source → EventBridge → Immediately routes to all targets
                                  ↘ Lambda
                                  ↘ SNS  
                                  ↘ SQS
  ```

- **Event sources:**
  - AWS services (EC2 state changes, S3 uploads, etc.)
  - SaaS applications (Shopify, DataDog, Zendesk, etc.)
  - Custom applications
- **Event targets:**
  - Lambda functions, SQS queues, SNS topics, Kinesis streams
  - [Step Functions](./08-04-integrations-workflows-swf-step-functions.md#aws-step-functions), ECS tasks, Systems Manager
  - Third-party destinations
- **Key features:**
  - Event transformation before delivery to targets
  - Dead letter queues for failed deliveries
    - ❗ Otherwise EventBridge is fire and forget
  - Event replay for testing and recovery
- **Use cases:**
  - Decouple microservices architecture
  - Real-time data processing pipelines
  - Application integration with SaaS platforms
  - Automated responses to infrastructure changes
  - Event-driven serverless workflows
- **Execution:**
  - Events processed in real-time (near real-time)
  - Pay-per-event pricing model
  - Automatic scaling and high availability

## SQS vs Amazon EventBridge

| Aspect | SQS (Simple Queue Service) | Amazon EventBridge |
|--------|----------------------------|---------------------|
| **Type** | Message Queue | Event Bus/Router |
| **Purpose** | Store and deliver messages | Route events between services |
| **Delivery Pattern** | One-to-One (point-to-point) | One-to-Many (pub-sub) |
| **Message Storage** | Yes - messages persist until consumed | No - events routed immediately |
| **Processing Model** | Pull-based (polling) | Push-based (real-time routing) |
| **Message Ordering** | FIFO available (with FIFO queues) | No ordering guarantees |
| **Retry Logic** | Built-in retry with DLQ | Target-specific retry with DLQ |
| **Event Sources** | Applications send messages explicitly | AWS services, SaaS apps, custom apps |
| **Event Filtering** | None (consumer processes all messages) | Advanced pattern matching and filtering |
| **Targets/Consumers** | One consumer per message | Multiple targets per event |
| **Use Case** | Decouple components, async processing | Event-driven architecture, integration |
| **Scalability** | Handle traffic bursts via buffering | Auto-scaling event routing |
| **Cost Model** | Per request + storage time | Per event published |
| **Message Transformation** | None | Built-in event transformation |
| **Schema Registry** | No | Yes - discover event structures |
| **SaaS Integration** | No built-in integrations | 100+ SaaS partner integrations |
| **Dead Letter Queue** | Yes | Yes (per target) |
| **Maximum Retention** | 14 days | No retention (immediate routing) |

## SNS vs EventBridge

- EventBridge = Modern event routing platform
- SNS = Notification delivery service
- For simple human notification, choose SNS, e.g.:
  - "Send SMS when server goes down"
  - "Email alerts for billing thresholds"
  - "Push notifications to mobile app"
- For event-driven architecture choose EventBridge, e.g:
  - Route S3 events to multiple Lambda functions
  - Integrate Salesforce with AWS services
  - Coordinate microservices in serverless architecture
- Can work together (a common pattern): `AWS Service → EventBridge (route to services) → SNS (notify humans) → SMS/Email/Push`

## SNS vs EventBridge vs Kinesis Data Streams

- SNS = Simple broadcasting
- EventBridge = Smart broadcasting with filtering
- Kinesis = High-volume data broadcasting

| Service | Broadcasting Type | Best For |
|---------|------------------|----------|
| [SNS](./08-01-02-event-broadcasting-sns-eventbridge.md#amazon-sns-simple-notification-service) | Simple pub-sub | Notifications, alerts, basic fan-out |
| [EventBridge](./08-01-02-event-broadcasting-sns-eventbridge.md#amazon-eventbridge) | Filtered event routing | Event-driven architecture, complex routing |
| [Kinesis Data Streams](./06-03-02-data-analytics-streaming-kinesis-data-firehose-kafka-flink.md#amazon-kinesis-data-streams) | Real-time data streaming | High-throughput data, multiple consumers |
