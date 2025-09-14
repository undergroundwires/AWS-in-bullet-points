# AWS Cost Optimizations

## AWS Compute Optimizer

- Delivers ML-powered recommendations to optimize AWS resources for cost and performance
- Analyzes [EC2](./05-01-01-compute-ec2-overview.md#ec2---elastic-compute-cloud), EBS, [Lambda](./05-04-compute-lambda.md#lambda), [RDS](./06-02-01-data-databases-rds.md#amazon-relational-database-service-amazon-rds), ECS Fargate, and Auto Scaling groups
- Uses [CloudWatch](./03-01-monitoring-cloudwatch-grafana.md#cloudwatch) metrics and resource configuration data
- Can reduce costs by up to 25% through rightsizing
- Provides more comprehensive recommendations than Cost Explorer
- Integrates with Cost Optimization Hub for savings visibility
- 💡 Cost Explorer shows subset of these recommendations with billing context

## Instance Scheduler

- 📝 Automates starting and stopping of AWS resources on defined schedules.
- Supports [EC2](./05-01-01-compute-ec2-overview.md#ec2---elastic-compute-cloud), Auto Scaling Groups, and [RDS](./06-02-01-data-databases-rds.md#amazon-relational-database-service-amazon-rds) instances.
- Uses resource tags and [AWS Lambda](./05-04-compute-lambda.md#lambda) to manage instances across multiple AWS Regions
- Supports cross-account instance scheduling and automated tagging
