# IAM & Organization

## MFA

- Stands for *multi-factor authentication*
- Factors
  - Pieces of evidence which prove identity
  - Can be knowledge (you know), possession (you have), inherent (you are), location (network/physical)
- More factors = more security
- Username + Password single factor
- 💡 Always activate MFA
- 💡 If your MFA device is lost/damaged/not working:
  - You can sign in using alternative methods of authentication.
    - E.g. sign in by verifying your identity using the email and phone that are registered with your account.

## IAM

- Centralized control of your AWS account.
  - Each API call to AWS is authorized using IAM.
- Shared Access to your AWS account
- Global
  - Does not apply to regions.
  - Any data is always secure across all AWS regions.
  - You use the same IAM configuration no matter which region you use.
- Handles Identity Federation:
  - Federating users of Mobile/Web app using [Amazon Cognito](./02-02-security-identity-federation-sts-saml-cognito-directory-service-identity-center.md#amazon-cognito)
    - Using [Amazon Cognito](./02-02-security-identity-federation-sts-saml-cognito-directory-service-identity-center.md#amazon-cognito) will create identities (with the help of Mobile SDK) for users and sync across devices.
  - Federating users with **OpenID Connect (OIDC)** or public identity service provider
    - Using third-party services like Facebook, Google or any identity providers (IdP) compatible with OIDC can federate users.
      - 💡 Amazon recommends against it, use [Amazon Cognito](./02-02-security-identity-federation-sts-saml-cognito-directory-service-identity-center.md#amazon-cognito) or Web Identity Federation only for advanced scenarios.
  - Federating users with **SAML 2.0**
    - If companies managing users with identity store like Microsoft Active Directory and Active Directory Federation Service can federate users with single-sign on (SSO) to AWS Management Console. Company can create trust between organization as an identity provider (IdP) and AWS as the service provider if it is compatible with SAML 2.0
  - Federating users by creating a **custom identity brokers**
    - If identity store is not compatible with SAML 2.0 then custom identity broker application can be built.
- Supports [MFA](#mfa)
- Handles:
  - Authentication: Prove you are who you claim to be
  - Authorization: Allow or deny access to resources
- Provide temporary access for users/devices and services where necessary
- Supports password policies:
  - Enforce strong password
  - Password rotation policy, e.g. expire after 90 days.
- Credential report: See which users are using what permissions.
- Integrates with many different AWS services
- Supports PCI DSS Compliance
- **IAM Certificate store**: Legacy way to upload certificates, use AWS Certificate Manager instead.
- **IAM Query API** to make direct calls to the IAM web service.
- **Pricing:** For free, at no cost.

### IAM Entities

![IAM entities](./img/iam/iam-entities.png)

#### IAM Principals

- **Principal** is an entity in AWS that can perform actions and access resources.

##### IAM Users

- **Programmatic Access: ID + secret key**
  - Mostly for machines.
  - 💡 Best practice for services is to assume roles
  - Assigned an Access Key ID and Secret Access Key when first created.
  - Secrets are only shown once after created, then needs to be regenerated to reveal/download again.
  - ❗ IDs are permanent you need to manually rotate (make active/disable, delete or create new)
- **Console Access: Password**
  - 💡 Ensure no one shares the same users by creating different users per person.
  - By default, newly created IAM users has no permissions.

##### IAM Roles

- 💡 Use cases:
  - Create roles and can then assign them to AWS resources
  - Give cross-account access to users.
  - Instead of being uniquely associated with one person, a role is intended to be assumable by anyone who needs it.
    - 💡 However for unique access of one you can delegate a role through:
      - In account add *trust policy* that specifies which trusted account members are allowed to assume the role.
      - The *trust policy* specifies which trusted account members (*principal*s) are allowed to assume the role.
- Credentials are rotated automatically -> Only temporary access.
  - Uses STS with expirations to generate secret id + secret key.
- You can only assume one role a time
- Can attach two types of policies:
  - Trust policy
    - Defines *who* can assume the role
  - Permission policy
    - Defines *what* the role can do once assumed (AWS actions + resources)

#### IAM Groups

- A collection of [IAM users](#iam-users) under one set of permissions (policies)
- Not a [principal](#iam-principals)
  - Cannot be identified as principal in an IAM policy
- 💡 Use groups to assign permissions instead of individual users
  - Users in group inherit the permissions from groups.
  - E.g. HR, finance, development team

#### IAM Policies

- Defines what resources can be accessed
- Identity-based policies:
  - Attached to user/role/group.
  - What can this user/role do?
- Resource-based policies
  - To supported AWS resources
  - Attached to resource, e.g. S3/SQS/VPC Endpoints etc..
  - Who can access this resource?
- When there's overlap => Deny overrides allow.
- To define resources ARN's are used (Amazon Resource Names) to uniquely identify AWS resources
- 💡 Principle of Least Privilege: Always adhere to it when creating IAM user, restrict access with only resources/actions to do the job.
- Can be attached to [IAM groups](#iam-groups), [IAM users](#iam-users) and [IAM roles](#iam-roles)

## IAM Authentication

### Password

- Optional for IAM users
- Long term credentials
  - Don't change automatically or regularly.
- Each IAM user has 1 username and 1 password

### IAM Access Keys

- IAM user can have 0 to 2 access keys
  - Ability to have two is useful when rotating access keys
  - Rotating access keys means deleting one and creating new ones.
- IAM roles don't use access keys.
- Access keys can be created, deleted, made active/inactive.
- Consists of:
  - **Access Key ID**
    - Similar to user name
    - E.g. `AKIAIOSFODNN7EXAMPLE`
  - **Secret access key**
    - Similar to password
    - E.g. `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY`

### IAM Service Authentication

- Most of AWS services authenticates a user / role.
- User / role (IAM) credentials must be sent in header
  - It's called Sig v4 **AWS Signature Version 4 Signing Process**
  - You sign all requests with signed using an access key (derived from access key ID + secret access key) in header
- All requests to AWS API's are signed automatically by SDK/CLI.
