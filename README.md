# AWS in bullet points

[![Quality checks status](https://github.com/undergroundwires/AWS-in-bullet-points/workflows/Quality%20checks/badge.svg)](https://github.com/undergroundwires/AWS-in-bullet-points/actions)
[![GitHub sponsors](https://undergroundwires.dev/img/badges/donate/flat.svg)](https://undergroundwires.dev/donate/?project=AWS-in-bullet-points)

- This repo contains study notes for different AWS exams.
- The notes are comprehensive and written with goal of covering all exam areas.
- Currently maintained for **SAA-C03** (I passed with 956/1000 with these notes).
- Good luck & enjoy studying! ☕

## Symbols

- There are some symbols used throughout the documentation:

  | Symbol | Description |
  |:------:|-------------|
  | 💡 | Best practice or an important solution |
  | ❗ | An important limitation, challenge or an exception |
  | 📝 | Common exam area |
  | 🤗 | Fact / trivia (most likely unrelated to the exam) |

## Content

### AWS Certified Solutions Architect - Associate

1. AWS Management
   1. [Introduction](./saa/01-01-aws-management-introduction.md)
   2. AWS Economics
      1. [Pricing (Pricing Calculator, Data Transfer, Savings Plans)](./saa/01-02-01-aws-management-economics-pricing-pricing-calculator-data-transfer-savings-plans.md)
      2. [Cost Optimizations (Compute Explorer, Instance Scheduler)](./saa/01-02-02-aws-management-economics-cost-optimizations-compute-optimizer-instance-scheduler.md)
      3. [Cost Monitoring & Management (Budget, Cost Explorer, Cost and Usage Report, Billing and Cost Management)](./saa/01-02-03-aws-management-economics-cost-monitoring-management-budget-cost-explorer-cost-and-usage-report-billing-and-cost-management.md)
   3. [Account Management (Account, Organizations, Control Tower)](./saa/01-03-aws-management-account-organizations-control-tower.md)
   4. [Resource Management (Resource Groups, Resource Access Manager, License Manager, Service Catalog)](./saa/01-04-aws-management-resource-groups-resource-access-manager-ram-license-manager-service-catalog.md)
   5. [Interacting with services (Console, CLI, SDKs)](./saa/01-05-aws-management-interacting-with-services-console-cli-sdks.md)
2. Security
   1. [IAM](./saa/02-01-security-iam.md)
   2. [Identity federation (STS, SAML, Cognito, SSO, Directory Service)](./saa/02-02-security-identity-federation-sts-saml-cognito-directory-service-identity-center.md)
   3. [Encryption in AWS (SSM, KMS, Secrets Manager)](./saa/02-03-security-encryption-ssm-kms-secrets-manager-certificate-manager.md)
3. Monitoring
   1. [CloudWatch, Managed Grafana](./saa/03-01-monitoring-cloudwatch-grafana.md)
   2. [Security Monitoring (CloudTrail, AWS Config, Trusted Advisor, Inspector, Artifact, Detective, Audit Manager)](./saa/03-02-monitoring-security-cloudtrail-aws-config-trusted-advisor-inspector-security-hub-artifact-detective-audit-manager.md)
4. Networking
   1. [IP Addresses (Elastic IP)](./saa/04-01-networking-ip-addresses-elastic-ip.md)
   2. [VPC](./saa/04-02-networking-vpc.md)
   3. [Subnets](./saa/04-03-networking-vpc-subnets.md)
   4. [Network Security](./saa/04-04-networking-security.md)
   5. [Hybrid connections (Site to site VPN, Client VPN, Direct Connect, Outposts)](./saa/04-05-networking-hybrid-connections-site-to-site-vpn-client-vpn-direct-connect-outposts.md)
   6. Edge
      - [Route 53](./saa/04-06-01-networking-edge-route-53.md)
      - [CloudFront](./saa/04-06-02-networking-edge-cloudfront.md)
      - [Optimization (Global Accelerator, AWS Wavelength)](./saa/04-06-03-networking-edge-optimization-aws-global-accelerator-wavelength.md#aws-global-accelerator)
5. Compute
   1. EC2
      1. [EC2 Overview](./saa/05-01-01-compute-ec2-overview.md)
      2. [EC2 Provisioning (Launch Types, Capacity Management, Tenancy, User Data, Instance Types, Fleet)](./saa/05-01-02-compute-ec2-provisioning-instance-types-launch-types-capacity-tenancy-user-data-fleet.md)
      3. [EC2 Networking](./saa/05-01-03-compute-ec2-networking.md)
      4. [EC2 Administration](./saa/05-01-04-compute-ec2-administration.md)
      5. [EC2 AMIs](./saa/05-01-05-compute-ec2-amis.md)
      6. [EC2 Storage (EBS, EFS, Instance Store)](./saa/05-01-06-compute-ec2-ebs-efs-instance-store.md)
   2. [Elastic Beanstalk](./saa/05-02-compute-elastic-beanstalk.md)
   3. [Containers (ECS, EKS, Managed Service for Prometheus)](./saa/05-03-compute-containers-ecs-eks.md)
   4. [Lambda](./saa/05-04-compute-lambda.md)
6. Data Services
   1. S3
      1. [S3 Overview](./saa/06-01-01-data-s3-overview.md)
      2. [S3 Security (Macie)](./saa/06-01-02-data-s3-security-macie.md)
      3. [S3 Integrated Services (Snowball, Athena, Storage Gateway, Object Lambda, Transfer)](./saa/06-01-03-data-s3-integrated-services-snowball-athena-storage-gateway-object-lambda-transfer.md)
   2. Databases
      1. [Overview](./saa/06-02-01-data-databases-overview.md)
      2. [Amazon RDS](./saa/06-02-01-data-databases-rds.md)
      3. [Amazon Aurora](./saa/06-02-03-data-databases-aurora.md)
      4. [DynamoDB](./saa/06-02-04-data-databases-dynamodb.md)
      5. [ElastiCache (Redis & Memcached)](./saa/06-02-05-data-databases-elasticache-redis-memcached.md)
      6. [Other databases (DocumentDB, Neptune)](./saa/06-02-06-data-databases-other-databases-documentdb-elasticsearch-neptune-timestream.md)
   3. Analytics
      1. [Data Warehousing (Redshift)](./saa/06-03-01-data-analytics-warehousing-redshift.md)
      2. [Data Streaming (Kinesis, Data Firehose, Managed Kafka/Flink)](./saa/06-03-02-data-analytics-streaming-kinesis-data-firehose-kafka-flink.md)
      3. [Data Querying (QuickSight, OpenSearch, RedShift Spectrum, Athena)](./saa/06-03-03-data-analytics-querying-quicksight-opensearch-redshift-spectrum-athena.md)
      4. [Data Processing (EMR, Glue, Batch, LakeFormation)](./saa/06-03-04-data-analytics-processing-emr-glue-batch-lakeformation.md)
      5. [Data Orchestration (Data Pipeline, AppFlow)](./saa/06-03-05-data-analytics-orchestration-data-pipeline-appflow.md)
7. Resiliency
   1. [Resiliency and High Availability (Health Dashboard)](./saa/07-01-resiliency-resiliency-and-high-availability-health-dashboard.md)
   2. [Load Balancers (ELB, CLB, NLB, ALB, GWLB)](./saa/07-02-resiliency-load-balancers-elb-clb-nlb-alb-gwlb.md)
   3. [Scalability (Auto Scaling, ASG)](./saa/07-03-resiliency-scalability-auto-scaling-asg.md)
   4. [Disaster Recovery (AWS Backup)](./saa/07-04-resiliency-disaster-recovery-aws-backup.md)
   5. [Infrastructure as Code (CloudFormation, CDK, SAM)](./saa/07-05-resiliency-infrastructure-as-code-cloudformation-cdk-sam.md)
   6. [Deployment Automation (Systems Manager, OpsWorks, Proton)](./saa/07-06-resiliency-deployment-automation-systems-manager-opsworks-proton.md)
8. Integrations
   1. Queues
      1. [Queues - Overview](./saa/08-01-01-integrations-queues-overview.md)
      2. [SQS](./saa/08-01-02-integrations-queues-sqs.md)
      3. [Amazon MQ](./saa/08-01-04-integrations-queues-amazon-mq.md)
   2. [Event Broadcasting (SNS, EventBridge)](./saa/08-01-02-event-broadcasting-sns-eventbridge.md)
   3. [API Integrations (API Gateway, AppSync)](./saa/08-03-integrations-api-gateway-appsync.md)
   4. [Workflows (SWF & Step Functions)](./saa/08-04-integrations-workflows-swf-step-functions.md)
9. [Migrations (Migration Hub, Migration Evaluator, Application Discovery Service, Database Migration Service, Application Migration Service, App2Container, DataSync)](./saa/09-migrations-discovery-migration-hub-migration-evaluator-application-discovery-service-database-migration-service-application-migration-service-app2container-datasync.md)
10. [Well-Architected Framework (Well-Architected Tool)](./saa/10-well-architected-framework-tool.md)
11. [AI Services](./saa/11-ai-services.md)
12. [Other services](./saa/12-other-services.md)
13. [Exam Readiness](./saa/13-exam-readiness.md)

## Support

- ⭐️ Simplest way to say thanks is just to give it a star 🤩
- ❤️ To show more support:
  - 👏🏿 [sponsor me](https://github.com/sponsors/undergroundwires)
  - 🎈 [buy me a coffee](https://undergroundwires.dev/donate/)
- ✨ Contributions of any kind are welcome!
