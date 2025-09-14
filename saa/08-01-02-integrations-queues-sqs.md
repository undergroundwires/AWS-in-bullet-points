# Amazon SQS (Simple Queue Service)

- Fully managed queue service
- 🤗 Oldest offering (over 10 years)
- Message retention period - between 1 minute to 14 days - default: 4 days
- No limit to how many messages can be in the queue
  - ❗ You need to process those within the time limit
- Flow
  - One or many ***Producers*** send ***messages*** to ***SQS Queue***
  - One or many ***Consumers*** poll ***messages*** from ***SQS Queue***
    - Many consumers does not mean fanout
    - Multiple Consumers:
      - Multiple consumers can read from the same queue
      - Each message is typically processed by only ONE consumer (after visibility timeout)
      - This is a "competing consumers" pattern where messages are distributed among consumers
      - Good for scaling processing of the same type of work
      - Example: 3 Lambda functions processing orders from one queue to increase throughput
    - Fanout:
      - One message is sent to multiple consumers simultaneously
      - Each consumer gets a copy of the SAME message
      - Example: One order event goes to billing queue, shipping queue, and analytics queue
      - ❗ SQS does not support fanout, see [SQS vs SNS](#sqs-vs-sns)
- **Scaling**
  - Scales automatically to 10,000 messages per second.
  - Horizontal scaling in terms of number of consumers.
- **Security**
  - You can use Server-Side Encryption using KMS.
- **Testing**
  - Send messages & simulate a consumer with polling on the console.
- Comparisons:
  - [SQS vs Amazon EventBridge](./08-01-02-event-broadcasting-sns-eventbridge.md#sqs-vs-amazon-eventbridge)
  - [SQS vs SNS](#sqs-vs-sns)

## Producing messages

- ❗ Maximum message size - between 1 and 256 KB - default: 256 KB
- *(Optional)* Add message attributes (key & value metadata)
- *(Optional)* Provide delay delivery
  - **Delay Queue**
    - Delay a message up to 15 minutes e.g. consumers don't see it immediately.
    - ***At queue level:*** Can set a default (default is 0 seconds -> visible right away)
    - ***At message level:*** Can override the default using the `DelaySeconds` parameter.
- Get back:
  - Message identifier
  - MD5 hash of the body

## Consuming messages

- Polls SQS for messages
  - ❗ Up to 10 messages at a time
- Consumer needs to ***delete messages*** from queue to prevent the message from being received.
  - Use API `DeleteMessage` using the message ID & receipt handle after successful processing.
- Receive message wait time - between 0 and 20 seconds - default: 0 seconds

### Visibility Timeout

- When a consumer polls a message from a queue, the message is "invisible" to other consumers for a defined period, the **Visibility Timeout**
- 📝 Messages remain in queue while being processed i.e. they're not returned to subsequent receive requests for duration of ***visibility timeout***
- Set between 0 seconds and 12 hours (default: 30 seconds)
  - If too high (e.g. 15 min): If consumer fails you must wait a long time before processing again.
  - If too low (e.g. 30 seconds): Message can be processed more than once before one consumer is done.

### Dead Letter Queue (DLQ)

- If consumer cannot handle the message, the message goes back to the queue after visibility timeout.
- If it happens many times then the message can be put into ***dead letter queue***.
  - Threshold of how many times is called **redrive policy**.
- DLQ is another SQS queue.
  - Create a DLQ first then designate it dead letter queue
  - 💡 Make sure to process the messages in the DLQ before they expire

### Pulling types

- **Short Polling**: Queue responds right away to *"Do you have messages?"*
- **Long Polling**
  - Eliminate empty responses by allowing SQS to wait until a message is available in a queue before sending a response.
    - The response to the `ReceiveMessage` request contains at least one of the available messages or unless the connection times out.
      - ❗ Up to the maximum number of messages specified in the `ReceiveMessage` action.
  - Recommended as it decreases the number of API calls
    - Reduces latency and increases efficiency of your application.
  - The wait time can be between 1 sec to 20 sec (20 sec preferable)
  - Can be enabled at the queue level or at the message level using `WaitTimeSeconds` API.

## Queue types

- **Standard Queue**
  - At least once delivery is guaranteed
    - 📝Can have ***duplicate messages*** occasionally
  - By design as it's very high throughput, AWS calls it unlimited throughput.
  - Best effort ordering: 📝Can have ***out of order messages***
  - Usually integrated with S3, [CloudWatch Events](./03-01-monitoring-cloudwatch-grafana.md#cloudwatch-events), SNS, Auto Scaling Hooks, IoT rule, lambda DLQs..
  - ❗ Limitation of 256KB per message sent
- **FIFO Queues**
  - Extends standard queue: have all capabilities.
  - Lower throughput: ❗ Up to 300 messages per seconds (for receive, delete & send)
    - If you batch messages to 10 then it'll be 3000 per second
    - Soft limit, you can request for higher limits
  - Ensures ***ordering*** and ***exactly-once processing***.
  - ❗ No per message delay (only per queue delay)
  - Ability to do content based **de-duplication**
    - Avoid duplicate messages during the deduplication interval
    - Can be
      - **Content based**: Uses hash of the message
      - **Deduplication ID**: 5-minute interval de-duplication using "Deduplication ID"
        - Messages with same deduplication ID are accepted but only one is delivered during 5-minute.
  - Name of the queue must end with `.fifo`
  - **Message Groups**
    - Possibility to group messages for FIFO ordering using `Message GroupID` tag on message.
    - Only one worker can be assigned per message group, so that messages are processed in order.
- Differences

  | Queue type | Delivery type | Ordering | Delay | Deduplication | Latency | AWS integrations |
  | ---------- | ------------- | -------- | ---- | -------------- | ------- | --------- |
  | **Standard** | At least once | Best effort | Per message and queue | NaN | <10 ms | Many |
  | **FIFO** | Exactly-once | FIFO, can also use custom `GroupID` tag to ensure one worker per group in order  | Only per queue | Content (hash) or custom ID based | <100 ms | Less, missing e.g. S3/CloudWatch/SNS/Lambda DLQ... |

## SQS vs SNS

- Choose SNS for fan out
  - E.g. someone uploads picture and listeners need to take different actions: send thank you email, crop thumbnail & save in S3.
- Choose SQS for single producer => single listener:
  - E.g. application to application communication.
- **Fanout**:
  - Also written as *fan out* or *fan-out*
  - Messaging pattern where a single message or event is distributed to multiple independent destinations or consumers simultaneously
  - SNS to SQS to listeners = best of both worlds.
