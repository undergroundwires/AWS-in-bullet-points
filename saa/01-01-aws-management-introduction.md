# Introduction

## AWS Global Infrastructure

- Consists of:
  - [AWS Regions](#aws-regions)
  - [Edge Locations](#edge-locations)
- Helps design architectures that are
  - highly available (if one data center fails, you can use another)
  - performant (e.g. closer to the users to reduce latency)
  - compliant (geopolitical separation)
- Global service means:
  - Multi-regional products
  - Resilient against regional failures
  - E.g. [IAM](./02-01-security-iam.md#iam), [Route 53](./04-06-01-networking-edge-route-53.md#route-53)

### AWS Regions

- AWS has regions all around the world, it is a separate geographic area. e.g. `us-east-1` `eu-west-1` `ap-northeast-1`
- **Pricing**
  - ❗ Costs of products may vary from region to region.
  - There's a [charge for data transfers](./01-02-01-aws-management-economics-pricing-pricing-calculator-data-transfer-savings-plans.md#data-transfer-charges) between regions
- Each region has at least two availability zones
  - E.g. `us-east-1a`, `us-east-1b`
  - **Availability zone**
    - One or more physical data centers.
    - They're separate from one another.
    - They're isolated deliberately to survive any unforeseen circumstances like disasters.
    - Some services supports multi-AZ (more resilient), some runs in single AZ.
  - Name of AZ can correspond to different data centers in one account than in other.
    - E.g. same data center can be `us-east-1a` for one account while `us-east-1b` for another, this is purposefully done by aws to distribute resources across AZ.
- Each AWS console are region scoped (except global services such as [IAM](./02-01-security-iam.md#iam), [Route 53](./04-06-01-networking-edge-route-53.md#route-53) and [S3](./06-01-01-data-s3-overview.md#amazon-s3-overview))
  - I.e. changes, actions are specific to a region
  - ❗ Anyhow S3 buckets inside are regional
- [See where the regions are and how many AZ's within them](https://aws.amazon.com/about-aws/global-infrastructure/)

### Edge Locations

- Smaller and more than regions
- Primarily for caching content for [CloudFront](./04-06-02-networking-edge-cloudfront.md) (CDN).
- Can have some edge services
- E.g. Netflix storing TV shows

## Basic Terminology

- **Availability**
  - Percentage uptime
    - Downtime can be due to faults or other such as planned maintenance.
  - E.g. the storage system is operational and can deliver data upon request
  - 📝High availability in AWS = Multiple AZ
  - **Resiliency**
    - Being able to adapt under stress or faults in order to avoid failure
      - Self-recover & remain functional to customers
    - Often provided by redundancies and automatic re-routing of operations within a system
  - **Durability**
    - Long-term data protection
    - E.g. the stored data does not suffer from bit rot, degradation or other corruption
  - **Redundancy**
    - The inclusion of extra components in a system so it will continue to function in case of a component failure.
    - E.g. automatic back-ups to Azure in case of AWS failure
  - **Throughput**
    - Measures of how many units of information a system can process in a given amount of time
- **Consistency**
  - Application is expected to remain stable
  - **Reliability**
    - Percentage uptime, considering the downtime due only to faults
    - Measures consistency
    - Ability of a system to recover from infrastructure or service disruptions.
    - 💡 Kill unhealthy & spin up new => Quicker than human debugging
  - **Fault-tolerance**
    - App ability to self-detect and correct all types of problems in its *environment*
    - Requires extra redundancy to work in inconsistent environments.
    - 📝 Elastic Load Balancer, Route 53 with Auto Scaling Groups
  - **Robustness**
    - App ability to self-detect and correct all types of problems from its *inputs*.
    - Can work with inconsistent data.
- **🤗Trivia**: In distributed data store, CAP Theorem means you cannot reach more than two of: Consistency & Availability & Partition tolerance (packages being dropping/delayed between nodes).
- **Billing**: You can see your bills in -> My billing dashboard -> bills

## Public vs Private Services

- Not about permissions but networking.
- **Public service:**
  - Accessed publicly via Internet
  - 💡Public access can be disabled, see [VPC Endpoints](./04-02-networking-vpc.md#vpc-endpoints)
  - E.g. [S3](./06-01-01-data-s3-overview.md#amazon-s3-overview)
- **Private service:**
  - Within [VPC](./04-02-networking-vpc.md#amazon-vpc)
  - Only accessed inside VPC, not via Internet
  - 💡 Can be configured for public Internet access

### AWS Health

- AWS service for visibility into AWS resource/service/account state
- **Health events**
  - Maintenance events that'll affect your services
  - Can set-up [CloudWatch alarms](./03-01-monitoring-cloudwatch-grafana.md#alarms).

## AWS Support

- Support types

  |   | Basic | Developer | Business | Enterprise |
  | - |:-----:|:---------:|:--------:|:----------:|
  | Cost | Free | $29/mo | $100/mo | $15,000/mo |
  | Use case | Account and billing questions only | Experimenting | Production use | Mission-critical use |
  | Tech support | *NO* | Business hour via e-mail | 24x7 via email, chat & phone | 24x7 via email, chat & phone |
  | SLA | ✕ | 12-24 hrs at local business hours | 1 hr response to urgent support cases | 15 min to critical support cases w/ priority |
  | TAM & Support Concierge |  ✕ | ✕ | ✕ | ✓ |
  | Support cases | ✕ | 1 Person, unlimited cases | Unlimited contacts / cases | Unlimited contacts / cases |
