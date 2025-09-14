# AWS Lambda

## Serverless introduction

- Cloud history:
  - Data Centre => IaaS (EC2, 2006) => PaaS => Containers => Serverless
- Concept came up with FaaS (Function as a Service) = Lambda
  - Today the definition includes anything that's managed where developers don't see / provision / manage servers.
  - Serverless service examples in AWS:
    - Lambda & [Step Functions](./08-04-integrations-workflows-swf-step-functions.md#aws-step-functions)
    - DynamoDB
    - [Amazon Cognito](./02-02-security-identity-federation-sts-saml-cognito-directory-service-identity-center.md#amazon-cognito)
    - AWS API Gateway
    - S3
    - SNS
    - SQS
    - Kinesis
    - [Aurora Serverless](./06-02-03-data-databases-aurora.md#aurora-serverless)
    - ...
- Serverless application examples:
  - Users *->* Rest API *->* API Gateway *->* Lambda *->* DynamoDB with log in function from [Amazon Cognito](./02-02-security-identity-federation-sts-saml-cognito-directory-service-identity-center.md#amazon-cognito).
  - Serverless thumbnail creation
    - User uploads image to S3
    - S3 triggers lambda function to create a thumbnail
    - Lambda
      - Creates & pushes thumbnail image into S3
      - Saves metadata in DynamoDB

## Lambda

- Runs in logically separated RAM, CPU and storage.
- Can be invoked sync or async.
- Can use different programming languages: Node.js (JavaScript), Python, Java, C# (.NET CORE), Golang, C# / PowerShell, C++ and more.
- Lambda like EC2 and ECS supports hyper-threading on one or more virtual CPUs.
- **Lambda@Edge** lets you run lambda functions in edge locations.
- ***Blueprint***s are code templates for writing Lambda functions.
- You can test lambdas directly on portal by configuring & sending ***test event***s in the Console

### Lambda vs EC2

| [EC2](./05-01-01-compute-ec2-overview.md) | Lambda |
|-----|--------|
| Virtual servers in the cloud  | Virtual functions -> no servers to manage! |
| Limited by RAM and CPU | Limited by time - short executions |
| Continuously running | Run on-demand |
| Scaling means intervention to add / remove servers | Scaling is automated |

### Lambda Pricing

1. Pay per **calls**.
2. Pay per **duration** (in increment of 100ms)
   - E.g. you get 400.000 GBs of FREE compute time
     - = 400.000 seconds if function is 1 GB RAM
     - = 3.200.000 seconds if function is 128 MB RAM

### Lambda Integrations

- Almost whole AWS Stack can trigger it
- Popular examples:
  - API Gateway
  - Kinesis
  - DynamoDB
  - S3
  - AWS IoT
  - [CloudWatch Events](./03-01-monitoring-cloudwatch-grafana.md#cloudwatch-events)
  - CloudWatch Logs
  - AWS SNS
  - [Amazon Cognito](./02-02-security-identity-federation-sts-saml-cognito-directory-service-identity-center.md#amazon-cognito)
  - [Amazon SQS](./08-01-02-integrations-queues-sqs.md)

### Lambda Security

- Each Lambda gets an execution role that can be IAM configured (grant/deny access to specific resources)
- Can be deployed within a **VPC**.
  - To enable your Lambda function to access resources inside your private VPC:
    - Give subnet IDs and security group IDs
    - Lambda uses those IDs to set up ENIs.
  - AWS Lambda uses this information to set up elastic network interfaces (ENIs) that enable your function to connect securely to other resources within your private VPC.
  - 💡 In your subnet you need enough available IP / ENI's otherwise you get `EC2ThrottledException` for concurrent execution.
- Lambda can have **Security Groups**.
- Auditing and compliance through ***CloudTrail*** logging.
- Encrypts by environment variables by default
  - ❗ Sensitive information would still be visible to other users who have access to the Lambda console
  - 💡📝 Use new AWS KMS key and use it to enable encryption helpers that leverage on AWS KMS to store and encrypt the sensitive information
    - Then you input encrypted data as environment variables
    - Others users will not see the values

### Lambda Configurations

- **Timeout**: Default 3 seconds, 📝 max of 15 minutes
  - Function fails directly after timeout
- **Environment variables**
  - Can be accessed directly from the code.
- **Allocated memory** (128MB to 10 GB)
  - 💡 Increasing RAM will also improve CPU and network!
  - 📝 Scaling is automated.
- **Lambda DLQ**
  - Debugging and error handling through dead letter queues
  - Can be SNS or [SQS queue](./08-01-02-integrations-queues-sqs.md#dead-letter-queue-dlq)
- **Encryption**
  - Encryption helpers to pass secure credentials in an encrypted manner.
  - Prevents other developer who has access to console from seeing the credentials.

### Lambda Scaling

- ❗ Concurrency limits how many lambda functions can be executed simultaneously.
  - For initial burst between 500-3000 depending on region.
  - Later: 500 per minute until limit is reached.
  - Concurrency limit starts from 1000 (soft limit)
- 💡 Architectures can get complicated -> [AWS X-Ray](./12-other-services.md#aws-x-ray) allows you to debug what's happening.
  - Trace and analyse keywords.

### Lambda Limitations

- RAM: Up to 10GB
- **Deployment**
  - **Max size** 250 MB or 50 MB (zipped)
    - 💡 Overcome limit: use `/tmp` directory to load other files at startup
  - **Size of environment variables**: 4 KB
- **Execution**
  - **Memory allocation**: 128 MB - 10 GB (1 MB increments)
  - **Maximum execution time**: 15 minutes
  - **Disk capacity** in the "function container" (in `/tmp`)
    - Configurable ephemeral storage (512MB default to 10GB maximum)
  - **Concurrency limits**: 1000 (soft limit)

## Lambda Startup Performance

- Lambdas can have cold start latency
  - The delay when Lambda functions initialize from scratch
- Solutions:
  - **Lambda SnapStart**
    - Snapshot/restore mechanism (like hibernation)
    - Reduces cold start times by up to 90%
  - **Provisioned Concurrency**
    - Keep-alive mechanism (pre-warmed execution environments)
    - Eliminates cold starts completely for provisioned capacity
    - Uses on-demand instances for excess traffic beyond provisioned amount
- Comparison:

  | Feature | SnapStart | Provisioned Concurrency |
  |---------|-----------|------------------------|
  | **Runtime Support** | Some runtimes | All runtimes |
  | **Cost** | Low (caching + restoration charges) | High (pay for reserved capacity) |
  | **Cold start reduction** | ~90% (sub-second in optimal scenarios) | 100% (eliminates cold starts) |
  | **Availability** | Published versions only | Any version/alias |
  | **Setup complexity** | Simple (just enable) | More complex (capacity planning) |
  | **Best for** | Scalable applications with initialization overhead | Strict latency SLAs, predictable traffic |
  | **Limitations** | No [EFS](./05-01-06-compute-ec2-ebs-efs-instance-store.md#amazon-elastic-file-system), no ephemeral storage >512MB | Ongoing costs whether used or not |
  | **Compatibility** | Cannot combine with provisioned concurrency | Cannot combine with SnapStart |
  | **Fallback** | Uses on-demand if demand exceeds provisioned | No fallback - hard scaling limit |
  | **Detection** | Environment variable shows init type | N/A |
  | **Capacity planning** | Formula-based + 10% buffer recommended | Automatic based on demand |
