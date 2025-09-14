# Security Monitoring

## CloudTrail

- Tracks API events allowing you to see who accessed what resources and when.
- CloudTrail reports on who made the change, when, and from which location.
- Per AWS account & per region
  - 💡 Should be enabled in all regions with a CloudFormation stack.
  - All accounts / regions can log into same S3 bucket in an account / region.
  - In a region when you apply the trail to all regions, CloudTrail creates a new trail in all other regions.
- Enabled by default
  - Provides event history for the last 90 days
  - Records management events by default (data events require configuration)
- **Encryption**
  - A single KMS key can be used to encrypt log files for trails applied to all regions.
  - CloudTrail log files are by default encrypted using S3 Server Side Encryption (SSE)
  - You can also enable encryption SSE KMS for additional security
- Get a history of events / API calls made within your AWS Account by Console, SDK, CLI, AWS Services
- Can push to S3 (encrypted by default), CloudWatch Logs and SNS.
- Has 90 days of retention
- 📝 If a resource is deleted in AWS, look into CloudTrail first!

## AWS Config

- Assess, audit and evaluate configurations of AWS resources
  - Resources include [RDS](./06-02-01-data-databases-rds.md), subnets, DB snapshots, security groups, and event subscriptions.
- Detailed view of the configuration of AWS resources in your AWS account
  - How resources are releated to each other
  - How they were configured in the past
  - How the configurations and relationships change over time
- 📝 Based on evaluation, can trigger remediation actions:
  - E.g. using AWS Lambda to enforce configuration changes.
- Reports on what has changed
  - You can e.g. look back and see what instances were in default VPC last week.
- Per AWS account & per region
  - 💡 Should be enabled in all regions with a CloudFormation stack.
  - Data aggregation in AWS Config allows you to aggregate AWS Config data from multiple accounts and regions into a single account.
- Tracks resource state.
- AWS Config is around compliance, [Trusted Advisor](#aws-trusted-advisor) is more around recommendations but they check same things for security.
- Integrates with SNS to receive notifications.

## AWS Trusted Advisor

- High level AWS account assessment
- Helps to know whether our account is sticking to AWS' best practices.
- Serverless global service
- **Pricing**: Most checks require different support plans (Developer plan includes only 7 core checks).
- Analyzes your AWS accounts and provides AWS-defined recommendations based on core checks:
  - **Cost Optimization**: e.g. Low utilization Amazon EC2 instances, Idle Load Balancers, unassociated Elastic IP address...
  - **Performance**: e.g. High Utilization Amazon EC2 Instances, Large Number of Rules in an EC2 security group
  - **Security**: e.g. "specific ports unrestricted", "Amazon EBS public snapshots"
  - **Fault Tolerance**: e.g. "VPN Tunnel Redundancy", "Age of EBS snapshots", "EC2 AZ Balance".
  - **Service Limits**
    - 📝 **Free** to check if you're getting close to limits
    - E.g. "EC2 On-Demand instances", "CloudFormation stacks"
- Can enable weekly email notification from the console

## Amazon Inspector

- Automated vulnerability assessment service
- Flow:
  1. Automatically scans software vulnerabilities and unintended network exposure
  2. Compares state to known security standards, vulnerabilities and best-practices
- Integrates with:
  - EC2 instances
    - Scans running EC2 instances for software vulnerabilities and unintended network exposure
    - ❗ Requires agent software on EC2 instances to check for unintended network accessibility
  - Container images
    - Scans container images stored in Amazon Elastic Container Registry (ECR) for vulnerabilities
  - Lambda functions
    - Scans Lambda function code and package dependencies for vulnerabilities
- Multi-account support with [AWS organizations](./01-03-aws-management-account-organizations-control-tower.md#aws-organizations)

## Amazon GuardDuty

- Threat detection service that monitors AWS accounts/workloads for malicious activity.
- Helps protect AWS accounts with intelligent threat detection
- Continously monitors accounts for malicious activity
  - Analyzes VPC Flow Logs, CloudTrail, and DNS logs using ML to identify risks
  - E.g. compromised instances or data exfiltration.
- Delivers security findings for visibility and remediation
- Provides automated security alerts.

## AWS Security Hub

- Shows "How compliant are WE?"
- Centralized security dashboard that aggregates findings from multiple AWS security services
- Acts as single pane of glass for security across AWS accounts
- Core purpose: Collects and standardizes security findings from GuardDuty, Inspector, Config, Macie, etc.
- Converts all findings into standard AWS Security Finding Format (ASFF)
- Multi-account support through [AWS Organizations](./01-03-aws-management-account-organizations-control-tower.md#aws-organizations)
- Key features:
  - Compliance dashboards for standards like CIS, PCI DSS, AWS Foundational Security Standard
  - Custom insights to filter and group findings
  - Automated remediation through EventBridge integration
  - Security scores and trend analysis
- 📝 Security Hub aggregates findings - it doesn't generate them
- Per region service but supports cross-region aggregation
- Integrates with 3rd party security tools through partner integrations

## AWS Artifact

- Shows "How compliant is AWS?"
- 📝 Central repository for AWS compliance documentation and security reports
- Provides on-demand access to formal compliance reports and certifications
- Self-service portal - no need to contact AWS support for standard compliance reports
- E.g. SOC 1, SOC 2, SOC 3 reports, ISO certifications, Payment Card Industry (PCI) DSS
- Requires AWS account, some reports (like SOC 1 and 2) require NDA agreements
- **Use cases:**
  - 💡 Audit teams assessing whether AWS services meet regulatory standards
  - 💡 Compliance officers needing formal documentation for certifications
  - 💡 Legal teams requiring contractual agreements like BAAs

## Amazon Detective

- Security investigation service that analyzes and visualizes security data to investigate findings
- Uses ML to build interactive graph visualizations from VPC Flow Logs, DNS logs, and CloudTrail
- While [GuardDuty](#amazon-guardduty) detects threats, Detective helps investigate and analyze the root cause of security issues
- Automatically collects data from AWS resources to create a unified view of user and resource interactions
- Provides pre-built data aggregations and visualizations to speed up investigations

## AWS Audit Manager

- Automated compliance audit preparation service that helps collect evidence for audits
- Continuously collects and organizes evidence from AWS services to support compliance audits
  - Automates evidence collection from CloudTrail, Config, Security Hub, and other AWS services
- Provides pre-built frameworks for standards like SOC 2, ISO 27001, HIPAA, GDPR
- Creates assessment reports that can be shared with auditors to demonstrate compliance
- Key distinctions:
  - While [Config](#aws-config) tracks configuration compliance, Audit Manager prepares audit-ready evidence packages for external auditors
  - While [Security Hub](#aws-security-hub) provides real-time security monitoring and compliance dashboards, Audit Manager focuses on formal audit preparation and evidence collection
