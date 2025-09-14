# Exam readiness

## Tips

- Strategy
  - Most questions are scenario based
  - First rule out obviously wrong answers
  - Pick remaining answer that makes most sense
  - Don't overthink - if complicated, probably wrong
  - No penalty for guessing
- Key Rules
  - AWS native solution with FEWEST moving parts wins
  - If AWS has built-in functionality → use it, don't build custom
  - AWS exams favor service integrations over best practices
  - "Least effort" → count services, pick fewer
- Logistics
  - 65 questions in 130 minutes (2 min each)
  - Use flag feature for hard questions
  - Come back to flagged ones later
- Study Tips
  - Read each service's FAQ
  - FAQs cover many actual exam questions

## Axioms

- Fundamental rules/patterns for AWS services that repeat across exam scenarios
- Use these to quickly eliminate wrong answers and pick right solutions

### Storage

- **Reliable/resilient** = EBS, [EFS](./05-01-06-compute-ec2-ebs-efs-instance-store.md#amazon-elastic-file-system), S3
  - For EBS
    - Small, random = SSD
    - Large, sequential = HDD
- **Durable** = S3
- **Low latency access** = EBS
- **Reproducible** = Spot instance, S3 one zone IA

### Decoupling

- **Decoupling** = SQS, SNS, EventBridge
- **Only once** = SQS FIFO or SWF

### High Availability

- **Multi-tier architecture** = ELB, ASG, EC2
- **High availability/fault tolerant** =
  - Horizontal scaling, vertical scaling
  - ASG that span over many AZ (sent to ELB)
  - DynamoDB (HA by default)
  - [RDS](./06-02-01-data-databases-rds.md) ([multi-AZ](./06-02-01-data-databases-rds.md#multi-az) needs to be enabled)
    - Availability = Sync replication, Scalability = async

### Performance

- **Performant storage and databases** =
  - 💡GP2, IO1 for EBS
  - 💡EFS, S3
- **Caching** =
  - ElastiCache
  - [CloudFront](./04-06-02-networking-edge-cloudfront.md)
  - API Gateway
  - DAX for DynamoDB

### Scaling

- **Elasticity and scalability** = ASG, Lambda, CloudWatch for alarms
- **Multi-region** = Stack Set

### Monitoring

- **CloudWatch**
  - **Enhanced** monitoring: Metrics within hyper-visor from RDS
  - **Detailed** monitoring: Send metrics in 1m interval instead of 5.

### Security

- **Secure application tiers** = Security groups
- **Secure data** = KMS, Encryption
- **Principle of least privilege** = IAM roles > IAM users
- **Network isolation** = Private subnets + NAT Gateway
- **DDoS protection** = Shield Advanced

### Cost Optimization

- **Cost optimized storage** =
  - S3, Pay for what you use, life-cycle rules to move data to cheaper, glacier for long time.
- **Cost optimized compute** =
  - Lambda, pay for what you use
  - Reserve long time instances
  - Spot Instances for short fault tolerant jobs
- **Licensing problems** = Microsoft + Oracle e.g. BYOL=Dedicated host

### Operations

- **Operational excellence** = Serverless
- **Automation** = CloudFormation, Systems Manager
- **Disaster recovery** = RTO/RPO drive solution (backup < pilot light < warm standby < hot standby)

### Data Transfer

- **Large data migration** = Snowball family
- **Ongoing sync** = DataSync, Storage Gateway
- **Real-time data** = Kinesis, Direct Connect

## More links

- Exam:
  - [Exam page](https://aws.amazon.com/certification/certified-solutions-architect-associate/)
  - [Exam guide](https://d1.awsstatic.com/training-and-certification/docs-sa-assoc/AWS-Certified-Solutions-Architect-Associate_Exam-Guide.pdf)
  - [Official Sample Exam Questions](https://d1.awsstatic.com/training-and-certification/docs-sa-assoc/AWS-Certified-Solutions-Architect-Associate_Sample-Questions.pdf)
  - [30 minute extension for your AWS Certification exam](https://www.linkedin.com/pulse/30-minute-extension-your-aws-certification-exam-garcia-lozano)
- Read:
  - [Architecting for the Cloud - AWS Best Practices](https://d1.awsstatic.com/whitepapers/AWS_Cloud_Best_Practices.pdf)
  - [AWS Well Architected Framework](https://aws.amazon.com/architecture/well-architected/)
    - [Operational Excellence](https://d1.awsstatic.com/whitepapers/architecture/AWS-Operational-Excellence-Pillar.pdf)
    - [Security](https://d1.awsstatic.com/whitepapers/architecture/AWS-Security-Pillar.pdf)
    - [Reliability](https://d1.awsstatic.com/whitepapers/architecture/AWS-Reliability-Pillar.pdf)
    - [Performance Efficiency](https://d1.awsstatic.com/whitepapers/architecture/AWS-Performance-Efficiency-Pillar.pdf)
    - [Cost Optimization](https://d1.awsstatic.com/whitepapers/architecture/AWS-Cost-Optimization-Pillar.pdf)
    - [Sustainability](https://docs.aws.amazon.com/pdfs/wellarchitected/latest/sustainability-pillar/wellarchitected-sustainability-pillar.pdf)
  - [AWS Disaster Recovery](https://d1.awsstatic.com/asset-repository/products/CloudEndure/CloudEndure_Affordable_Enterprise-Grade_Disaster_Recovery_Using_AWS.pdf)
- Reference architectures resources for real-world
  - [AWS reference architectures](https://aws.amazon.com/architecture/)
  - [AWS Solutions](https://aws.amazon.com/solutions/)
  - [This is my architecture](https://aws.amazon.com/architecture/this-is-my-architecture/)
  - [AWS Startups Blog](https://aws.amazon.com/blogs/startups/)
  - [Hands-On Tutorials](https://aws.amazon.com/getting-started/hands-on/)
