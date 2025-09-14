# Well-Architected Framework

## 6 pillars of Well-Architected Framework

- Not something to balance, or trade-offs, they're a synergy
  - Help you track your performance and evolve your architecture.

### Operational Excellence

- **Description**: Run and monitor systems & continually improve supporting processes and procedures
- **Design principles**
  - **Perform operations as code**: Infrastructure as Code
  - **Annotate documentation**: E.g. auto-document after every build
  - **Make frequent, small, reversible changes**
  - **Refine operations procedures frequently**: Ensure team members are familiar with it.
  - **Anticipate failure**: Learn from all operational failures
- **Example AWS Services**
  - **Selection**: Auto Scaling, Lambda, EBS, S3, [RDS](./06-02-01-data-databases-rds.md)
  - **Review**:
    - [AWS CloudFormation](./07-05-resiliency-infrastructure-as-code-cloudformation-cdk-sam.md#cloudformation)
    - [AWS News Blog](https://aws.amazon.com/blogs/aws/)
  - **Monitoring**: CloudWatch, Lambda
  - **Tradeoffs**:
    - [Amazon RDS (vs Aurora)](./06-02-01-data-databases-overview.md#aurora-vs-rds)
    - [ElastiCache](./06-02-05-data-databases-elasticache-redis-memcached.md#elasticache) (read performance but stale)
    - [Snowball](./06-01-03-data-s3-integrated-services-snowball-athena-storage-gateway-object-lambda-transfer.md#aws-snowball-edge) (a lot of data but takes a week)
    - [CloudFront](./04-06-02-networking-edge-cloudfront.md) (cache but stale)

### Security

- **Description**: Protect information, systems, and assets through risk assessments and mitigation strategies
- **Design principles**
  - **Implement a strong identity foundation**: Least privilege, IAM, centralize privilege management, eliminate reliance on long-term credentials
  - **Enable traceability**: Automatically respond to logs and metrics.
  - **Apply security at all layers**: Every instance, OS, and app, e.g. VPC, subnet, LB.
  - **Automate security best practices**: Encryption, tokenization, and access control.
  - **Protect data in transit and at rest**
  - **Keep people away from data**: Reduce the need for direct access or manual data processing
  - **Prepare for security**: Run incident response simulations and use tools with automation to increase your speed for detection, investigation, and recovery
- **Example AWS Services**
- **Identity and access management**:
  - IAM
  - AWS STS
  - MFA token
  - [AWS Organizations](./01-03-aws-management-account-organizations-control-tower.md#aws-organizations)
- **Detective controls**: AWS Config, AWS CloudTrail, Amazon CloudWatch
- **Infrastructure Protection**
  - [Amazon CloudFront](./04-06-02-networking-edge-cloudfront.md): Defense against DDoS attack
  - [Amazon VPC](./04-02-networking-vpc.md)
  - [AWS Shield](./04-04-networking-security.md#aws-shield): DDoS protection of AWS account
  - [AWS WAF](./04-04-networking-security.md#aws-waf): Web Application Firewall
  - [Amazon Inspector](./03-02-monitoring-security-cloudtrail-aws-config-trusted-advisor-inspector-security-hub-artifact-detective-audit-manager.md#amazon-inspector) for security of EC2 instance
- **Data protection**: KMS, S3 (SSE, SSE-KMS, SSE-C, bucket policies etc.)
  - And other managed services such as Elastic Load Balancing (e.g. only HTTPS), Amazon EBS & [RDS](./06-02-01-data-databases-rds.md) (SSL Capability)
- **Incident Response**
  - **[IAM](./02-01-security-iam.md)**: Delete account or give it zero privilege
  - **[CloudFormation](./07-05-resiliency-infrastructure-as-code-cloudformation-cdk-sam.md#cloudformation)**: If someone deletes entire structure, how to get back?
  - [Amazon CloudWatch Events](./03-01-monitoring-cloudwatch-grafana.md#cloudwatch-events)

### Reliability

- **Description**:
  - Ability of a system to recover from infrastructure or service disruptions
  - Dynamically acquire computing resources to meet demand
  - Mitigate disruptions such as misconfigurations or transient network issues.
- **Design principles**
  - **Test recovery procedures**: Use automation to simulate different failures or to recreate scenarios that led to failures before
  - **Automatically recover from failure**: Anticipate and remediate failures before they occur.
  - **Scale horizontally to increase aggregate system availability**: Distribute requests across multiple, smaller resources to ensure that they don't share a common point of failure
  - **Stop guessing capacity**: Maintain the optimal level to satisfy demand without over or under provisioning, use auto-scaling.
  - **Manage change in automation**: Use automation to make changes to infrastructure
- **Example AWS Services**
  - **Foundations**
    - **IAM**: No one has too many rights to damage
    - Amazon VPC
    - **Service limits**: monitor limits so you don't get disruption.
    - **AWS Trusted Advisor**: e.g. look at service limits
  - **Change Management**: AWS Auto Scaling, Amazon CloudWatch, AWS CloudTrail, AWS Config
  - **Failure Management**: Backups, [AWS CloudFormation](./07-05-resiliency-infrastructure-as-code-cloudformation-cdk-sam.md#cloudformation) (recreate whole infrastructure at once), Amazon S3, Amazon S3 Glacier, [Amazon Route 53](./04-06-01-networking-edge-route-53.md) (e.g. if application fails you redirect to another application)

### Performance

- **Description**: Adopt practices to provide best performance
- **Design principles**
  - Democratize advanced technologies *(track new services)*
  - Go global in minutes *(easy multi-region deployment)*
  - Use serverless architectures *(avoid burden of managing servers)*
  - Experiment more often *(easy to carry out comparative testing)*
  - Mechanical sympathy *(be aware of all AWS services)*
- **Example AWS Services**
  - **Prepare**:
    - [AWS CloudFormation](./07-05-resiliency-infrastructure-as-code-cloudformation-cdk-sam.md#cloudformation)
    - AWS Config
  - **Operate**:
    - [AWS CloudFormation](./07-05-resiliency-infrastructure-as-code-cloudformation-cdk-sam.md#cloudformation)
    - AWS Config
    - AWS CloudTrail
    - Amazon CloudWatch
    - [AWS X-Ray](./12-other-services.md#aws-x-ray)
  - **Evolve**:
    - [AWS CloudFormation](./07-05-resiliency-infrastructure-as-code-cloudformation-cdk-sam.md#cloudformation)
    - CI/CD: CodeBuild, CodeCommit, CodeDeploy, CodePipeline

### Cost Optimization

- **Description**: Can run systems to deliver business value at the lowest price point.
- **Design Principles**
  - **Adopt a consumption mode**: Pay only for what you use
  - **Measure overall efficiency**: Use CloudWatch
  - **Analyze and attribute expenditure**: Use tags, Accurate identification of system usage and costs -> helps measure return on investment (ROI)
  - **Use managed and application level services to reduce cost of ownership**: As managed services operate at cloud scale, they can offer a lower cost per transaction or service
- **Example AWS Services**
  - **Expenditure awareness**: AWS Budgets, AWS Cost and Usage Report, AWS Cost Explorer, Reserved Instance Reporting
  - **Cost-effective resources**: Spot instance, Reserved instance, Amazon S3 Glacier
  - **Matching supply and demand**: AWS Auto Scaling, AWS Lambda
  - **Optimizing over time**: AWS Trusted Advisor, AWS Cost and Usage Report, [AWS News Blog](https://aws.amazon.com/blogs/aws/)

### Sustainability

- **Description**: Minimize environmental impacts of running cloud workloads
- **Design principles**
  - **Understand your impact**: Measure the environmental impact of cloud workload and establish KPIs
  - **Establish sustainability goals**: Set long-term sustainability goals for workloads
  - **Maximize utilization**: Right-size workloads and implement efficient design patterns
  - **Anticipate and adopt new, more efficient hardware and software offerings**
  - **Use managed services**: Leverage AWS managed services to reduce your environmental impact
  - **Reduce the downstream impact of your cloud workloads**
- **Example AWS Services**
  - **Sustainability in the cloud**: EC2 Auto Scaling, Lambda (serverless), EBS (right-sizing), S3 Intelligent Tiering
  - **Sustainability of the cloud**: AWS uses renewable energy, efficient cooling, server utilization optimization

## AWS Well-Architected Tool

- 💡 Free service that helps you review your architectures against the [6 pillars of Well-Architected Framework](#6-pillars-of-well-architected-framework)
- Provides a consistent process to evaluate and improve your cloud architectures
- **Key Features:**
  - **Architecture reviews**: Comprehensive assessment across all [6 pillars](#6-pillars-of-well-architected-framework)
  - **Best practice guidance**: Actionable recommendations based on AWS expertise
  - **Risk identification**: Highlights high and medium risk issues
  - **Improvement tracking**: Monitor progress over time with milestones
- **How it works:**
  1. **Define a workload**
     - Name your workload and provide description
     - Select the industry type and AWS Regions
     - Identify the architecture components
  2. **Conduct a review**
     - Answer questions for each pillar: [Operational Excellence](#operational-excellence), [Security](#security), [Reliability](#reliability), [Performance](#performance), [Cost Optimization](#cost-optimization), [Sustainability](#sustainability)
     - Questions assess your architecture against best practices
     - Example questions:
       - **Operational Excellence**: "How do you determine what your priorities are?"
       - **Security**: "How do you securely operate your workload?"
       - **Reliability**: "How do you design your workload service architecture?"
     - Provide notes and evidence for your answers
  3. **Review results and take action**
     - View risk summary across all pillars
     - Get prioritized improvement recommendations
     - Access detailed guidance and implementation resources
     - Track progress with action plans
- **Additional capabilities:**
  - **Custom lenses**: Industry-specific or specialized assessments (e.g., SaaS lens, IoT lens)
  - **Workload sharing**: Collaborate with team members and AWS partners
  - **Milestone creation**: Save point-in-time snapshots of your review
  - **Report generation**: Export findings and improvement plans
  - **Integration**: Works with AWS Trusted Advisor and other AWS optimization tools
- **Access**: Available through [AWS Management Console](https://console.aws.amazon.com/wellarchitected) at no additional cost
- 📝 **Common exam scenarios**:
  - Identifying architectural improvements before issues arise
  - Preparing for architecture reviews and audits
  - Establishing baseline measurements for architectural quality
  - Continuous improvement of cloud workloads
