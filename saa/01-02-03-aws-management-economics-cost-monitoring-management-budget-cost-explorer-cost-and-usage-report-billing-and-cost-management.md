
# AWS Cost Monitoring & Management

## AWS Budget

- Budget tracking & notifications
- Helps track costs, usage, and coverage with custom budgets.
- Supports creating custom actions
  - to run automatically or manually
  - to e.g. prevent overages, inefficient resource use, or lack of coverage.

## AWS Cost Explorer

- Cost visualization & analysis
- Interface to visualize, understand, and manage costs.
- Helps analyze & optimize AWS spending
- Shows data for up to the last 13 months.
- Provides Cost Explorer API for programmatic access.
- Provides rightsizing recommendations based on your [Amazon EC2](./05-01-01-compute-ec2-overview.md#ec2---elastic-compute-cloud) usage

## AWS Cost and Usage Report (CUR)

- Detailed billing data
- 📝 Comprehensive set of AWS cost and usage data available
- 💡 Use when you need more detailed analysis than [Cost Explorer](#aws-cost-explorer) provides
- Delivers detailed billing data to S3 bucket in CSV/Parquet formats
- Contains line-item usage data with additional metadata (resource tags, pricing info, etc.)
- **Key Features**:
  - Hourly, daily, or monthly granularity
  - Can include resource IDs for detailed cost allocation
  - Supports data integration with QuickSight, Redshift, Athena
  - Version 2.0 supports Parquet format and Athena integration
- 💡 Good for for enterprise chargeback and detailed cost allocation
- ❗ Raw data requires processing - unlike Cost Explorer's ready-made visualizations

## AWS Billing and Cost Management

- Comprehensive billing suite
- Suite of features for billing setup, invoice retrieval, cost analysis, organization, and optimization
- **Console Home** provides holistic view of AWS cloud finances with tailored insights and recommendations
- Features:
  - Bills page: Download invoices and view detailed monthly billing data
  - Payment profiles: Set up multiple payment methods for different parts of organization
  - Cost categories: Map costs to teams, applications, or environments
  - Cost allocation tags: Use resource tags to organize and view costs
  - Purchase orders: Create and manage POs for compliance with procurement processes
  - Cost Anomaly Detection: Automated alerts when AWS detects unusual cost patterns
- Supports consolidated billing across multiple AWS accounts via [AWS Organizations](./01-03-aws-management-account-organizations-control-tower.md#aws-organizations)
- Integrates with [IAM](./02-01-security-iam.md#iam) for access control to billing information
