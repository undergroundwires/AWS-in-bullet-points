# Account Management

## AWS Account

- AWS Account is container for
  - identities (users)
  - resources
- An account requires:
  - A name (e.g. `prod...`)
  - Unique e-mail address
    - Must be unique for every account
    - *Owner* usually means the e-mail owner of the registered account.
    - ❗ Not required for non-root accounts in AWS Organizations
  - Credit card information (payment)
    - Can be used for multiple AWS accounts
    - All resource usage is billed to this card

### Root user

- Every account has one root user
- Each root user is unique to the single account
- ❗ Cannot be restricted and always have full access to everything
- 💡 Other identities ([IAM](./02-01-security-iam.md#iam) users, groups, roles) can be restricted, or given full access
  - By default all resource access is denied, except explicitly allowed.
- Best-practices:
  - 💡 Always enable MFA on root account
  - 💡 Avoid using root account as much as possible, create Admin [IAM](./02-01-security-iam.md#iam) user
  - 💡 Enforce the use of complex passwords for member account root user logins
  - ❗ Never use root account for everyday activity.

### Single vs Multi-account

- Traditionally organizations used single account
- Currently, many organizations use multi-account
- Helps security by isolating user access
- Can create different accounts per team, functionality, environment etc.

## AWS Organizations
  
- Consolidate multiple AWS accounts into an organization that you create and centrally manage
- Helps to centrally manage AWS environment when scaling AWS resources
- Enables enforcing [Service Control Policy (SCP)](#service-control-policies) centrally on root or OUs.
- Allows applying policies to accounts or groups.
  - Allows sharing resources (e.g. SSO)
  - See [service control policies](#service-control-policies)
  - Allows configuring conditional access.
    - E.g. using `aws:PrincipalOrgID` and `aws:PrincipalTag` in IAM policies.
- Allows **Consolidated billing**
  - Multiple standalone accounts are combined and may reduce your overall bill
  - They can still be tracked individually and the cost data can be downloaded in a separate file.
- Allows you to create accounts programmatically with AWS Organizations APIs.
- Groups all of your AWS accounts into [Organizational Units (OUs)](#organizational-units) as part of a hierarchy

### Organizational Units

- Logical grouping of accounts (1 OU can contain multiple accounts)
- Provides hierarchical based control over groups of IAM users and roles, within multiple AWS Accounts.
- Groups accounts together to administer as a single unit.
- Simplifies the management of your accounts.
  - E.g. attach a policy-based control to an OU and all accounts within OU inherit the policy.
- You can create multiple OUs within organization or other OUs
- You can move accounts between OUs
- OU names must be unique within a parent OU or root

### Service Control Policies

- SCP has precedence over individual's permission.
- Organizational guardrails that set maximum permission boundaries across AWS accounts
- Helps compliance and prevents unauthorized actions even if users have admin access.
- Common use-cases:
  - Allow list: Block services by default and then allow certain services.
  - Deny list: Allow by default and block access to certain services.
- Best suited for restricting access across multiple accounts in an organization.

## AWS Control Tower

- 📝 Multi-account governance service that automates setup of secure, well-architected AWS environments
- Built on top of [AWS Organizations](#aws-organizations), [AWS Config](./03-02-monitoring-security-cloudtrail-aws-config-trusted-advisor-inspector-security-hub-artifact-detective-audit-manager.md#aws-config), [CloudTrail](./03-02-monitoring-security-cloudtrail-aws-config-trusted-advisor-inspector-security-hub-artifact-detective-audit-manager.md#cloudtrail)
- **Core Components:**
  - **Landing Zone**: Pre-configured secure multi-account environment
  - **Guardrails**: Governance rules (preventive via SCPs, detective via Config)
  - **Account Factory**: Automated account provisioning with standardized baselines
  - **Dashboard**: Centralized compliance visibility
- **Key features:**
  - Centralized logging ([CloudTrail](./03-02-monitoring-security-cloudtrail-aws-config-trusted-advisor-inspector-security-hub-artifact-detective-audit-manager.md#cloudtrail) across all accounts)
  - Least privilege access (pre-configured [IAM](./02-01-security-iam.md#iam) roles and SCPs)
  - Security monitoring (dedicated security/audit account)
  - Standardized account baselines and change management workflows
  - Drifts (changes in compliance to baseline) can be viewed and notified (using [Amazon SNS](./08-01-02-event-broadcasting-sns-eventbridge.md#amazon-sns-simple-notification-service))
- **Benefits:**
  - Reduces complexity: Automates multi-account setup work
  - Enforces best practices: Implements Well-Architected principles automatically
  - Accelerated deployment: New accounts ready in minutes vs hours/days
- 💡 Use for:
  - Enterprise (10+ accounts)
  - Compliance requirements
  - Standardized provisioning
