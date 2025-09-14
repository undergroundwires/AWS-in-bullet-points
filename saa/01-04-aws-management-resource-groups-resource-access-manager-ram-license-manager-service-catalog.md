# Resource Management

## AWS Resource Groups

- Can be based on tags or [CloudFormation](./07-05-resiliency-infrastructure-as-code-cloudformation-cdk-sam.md#cloudformation) stacks
- Good for
  - Bulk options e.g. updates/security patches, upgrading applications, opening/closing ports
  - [IAM](./02-01-security-iam.md#iam) permissions can be assigned to RG.

## AWS Resource Access Manager (RAM)

- 📝 Securely share AWS resources across accounts within [organization](./01-03-aws-management-account-organizations-control-tower.md#aws-organizations) or with external accounts
- Key shareable resources: [VPC](./04-02-networking-vpc.md#amazon-vpc) subnets, Transit Gateway attachments, [Route 53](./04-06-01-networking-edge-route-53.md#route-53) Resolver rules, License Manager configurations
- Invitations:
  - Within [AWS Organizations](./01-03-aws-management-account-organizations-control-tower.md#aws-organizations): Automatic sharing, no invitations required
  - External accounts: Requires invitation and acceptance
- ❗ Limitations: Cannot share default VPC/subnets, resource owner retains control
- **Costs**
  - No additional charges for RAM itself - only pay for underlying shared resources
  - Resource owner pays for resources, consumers pay for usage (e.g. data transfer)

## AWS License Manager

- Centrally manage software licenses across AWS accounts and on-premises environments
- Helps track, manage, and control license usage to avoid compliance violations and reduce costs

## AWS Service Catalog

- Create and manage catalogs of IT services approved for use on AWS
- Enables organizations to achieve consistent governance while enabling users to quickly deploy approved resources
