# Interacting with services

- All services can be called using APIs.
- There other services/tools that calls these APIs such as:
  - [AWS Management Console](#aws-management-console) (web)
  - [AWS SDKs](#aws-sdks)
  - [AWS CLI](#aws-cli)

## AWS Management Console

- Web-based interface to access and manage AWS services through a browser
- 💡 Provides a user-friendly graphical interface for users who prefer visual management over command-line tools, though [CLI](./01-05-aws-management-interacting-with-services-console-cli-sdks.md#aws-cli) and [SDKs](./01-05-aws-management-interacting-with-services-console-cli-sdks.md#aws-sdks) are preferred for automation and scripting

## AWS CLI

- Interact with AWS services ([S3](./06-01-01-data-s3-overview.md#amazon-s3-overview), [DynamoDB](./06-02-04-data-databases-dynamodb.md#dynamodb) etc.).
- Can be installed on Linux/Windows/Mac OS
- Python based using *botocore* / *boto3* (AWS SDK for python)
- Requires access credentials
  1. Can be found at *[IAM](./02-01-security-iam.md#iam) -> Users -> Security credentials*
  2. Create your [access key](./02-01-security-iam.md#iam-access-keys) (can download as CSV)
     - ❗ You cannot restore them, they're generated & shown once
     - To revoke it: disable or delete your key.
  3. Run `aws configure` & paste *Access Key ID* and *Secret Access Key* when prompted
     - You can optionally set default region & output format
       - Otherwise the region is `us-east-1` by default
     - You can run `aws configure` to check if they're saved & modify them.
  4. Credentials are saved in `~/.aws` in Mac OS & Linux
     - In `config` file you have region and output settings.
     - In `credentials` file your access key ID and secret key are saved
  5. You can run `help` to get more information about command e.g. `aws s3 ls help`
- **AWS CLI on EC2**
  - ❗ Bad & insecure way:
    - Run `aws configure` and save your credentials
    - Your personal credentials are personal and should only be on your personal computer (not on EC2)
    - Risks:
      - If EC2 is compromised -> Your account will also be compromised
      - If EC2 is shared -> other people may perform actions impersonating you
  - 💡 Better way: AWS [IAM](./02-01-security-iam.md#iam) Roles
    - [IAM](./02-01-security-iam.md#iam) Roles can be attached to [EC2](./05-01-01-compute-ec2-overview.md#ec2---elastic-compute-cloud) instances.
      - ❗ One [EC2](./05-01-01-compute-ec2-overview.md#ec2---elastic-compute-cloud) instance can have one [IAM](./02-01-security-iam.md#iam) role at a time
        - But same role can be attached to many other [EC2](./05-01-01-compute-ec2-overview.md#ec2---elastic-compute-cloud) instances
      - 📝 Also, Roles are not directly attached to [EC2](./05-01-01-compute-ec2-overview.md#ec2---elastic-compute-cloud) instance, but are attached to instance profile. Instance profile is a container, where roles are attached.
    - [IAM](./02-01-security-iam.md#iam) Roles can come with a policy authorizing exactly what the [EC2](./05-01-01-compute-ec2-overview.md#ec2---elastic-compute-cloud) instance should be able to do e.g. AdminAccess or List buckets in [S3](./06-01-01-data-s3-overview.md#amazon-s3-overview).
    - [EC2](./05-01-01-compute-ec2-overview.md#ec2---elastic-compute-cloud) instances can use these profiles automatically without any additional configurations
  - 📝If [EC2](./05-01-01-compute-ec2-overview.md#ec2---elastic-compute-cloud) needs access to other AWS resources, never put credentials in it, always use [IAM](./02-01-security-iam.md#iam) roles
  - Flow:
    - Create [IAM](./02-01-security-iam.md#iam) role for the resource [EC2](./05-01-01-compute-ec2-overview.md#ec2---elastic-compute-cloud) will access
    - [EC2](./05-01-01-compute-ec2-overview.md#ec2---elastic-compute-cloud) console -> Select instance -> Actions -> Security -> Modify [IAM](./02-01-security-iam.md#iam) role
- **S3 CLI**
  - `aws s3 ls`: Lists all buckets available
  - `aws s3 ls s3://bucketname`: Lists contents of the bucket
  - `aws s3 cp s3://bucketname/pic.jpg pic.jpg`: Copy a local file, or S3 objects to your computer or from your computer.
  - `aws s3 mb s3://bucketname`: create a new bucket
  - `aws s3 rb s3://bucketname`: delete existing bucket
  - `aws configure --profile <profile-name>`: Allows you to name a profile and work with multiple profiles. Future commands should have `--profile <profile-name>` parameter to use the profile.
  - Use appropriate region to the end of the command in all the above, remember S3 buckets are regional.

## AWS SDKs

- Official SDKs are: Java, .NET, Node.js, PHP, Python (named boto3/botocore), Go, Ruby, C++ ...
- AWS CLI uses the Python SDK
- Recommended to use the **default credential provider chain**
- It works seamlessly with:
  - AWS credentials at `~/.aws/credentials`
  - Use only on your computers or on premise
    - Even better to create a user specifically for that one on-premise server
  - Instance Profile Credentials using [IAM](./02-01-security-iam.md#iam) Roles
  - Use 100% within AWS services e.g. [EC2](./05-01-01-compute-ec2-overview.md#ec2---elastic-compute-cloud) machines.
  - You can use environment variables (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`)
    - ❗ Less recommended
- **Exponential Backoff**
  - E.g. request 1 fails -> wait 2 sec -> request 2 fails -> wait 4 sec -> request 3 fails -> wait 8 sec ...
  - Any API that fails because of too many calls needs to be retried with Exponential Backoff
    - Applies also to rate limited API
  - Retry mechanism included in SDK API calls
  - Ensures you don't overload API

## Penetration testing

- AWS allows penetration testing on your own AWS resources without prior approval
- Common permitted services include:
  - [EC2](./05-01-01-compute-ec2-overview.md#ec2---elastic-compute-cloud) instances, NAT Gateways, [ELB](./07-02-resiliency-load-balancers-elb-clb-nlb-alb-gwlb.md#elastic-load-balancer-elb)
  - [RDS](./06-02-01-data-databases-rds.md#amazon-relational-database-service-amazon-rds)
  - [CloudFront](./04-06-02-networking-edge-cloudfront.md)
  - [Aurora](./06-02-03-data-databases-aurora.md#amazon-aurora)
  - [API Gateway](./08-03-integrations-api-gateway-appsync.md#api-gateway)
  - [Lambda](./05-04-compute-lambda.md#lambda) functions
  - Elastic Beanstalk
- 💡 Always check current AWS Customer Support Policy for Penetration Testing for latest guidelines
- ❗ Prohibited activities include: DDoS, DoS, DNS zone walking, port/protocol/request flooding
- Some activities still require approval - check AWS documentation for current requirements.
