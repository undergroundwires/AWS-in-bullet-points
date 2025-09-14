# Encryption in AWS (SSM, KMS, Secrets Manager, Certificate Manager)

- ![Encryption in AWS](img/encryption/encryption-in-aws.png)

## Encryption Concepts

- Adds tunnel around the data, so no one can read it.
- Encryption has impact on performance sometimes high sometimes minimal
- Concepts:
  - *Plaintext:* Unencrypted, raw data
  - *Algorithm:* Takes encryption key and generates encrypted data
  - *Key:* Password used with algorithm to generate ciphertext
  - *Ciphertext:* Encrypted data.

### Encryption Key Management

- Two options:
  - **Hardware security module (HSM)**
    - Designed and certified to
      - be tamper-evident,
      - be intrusion-resistant,
      - provide the highest level of physical security, no hardware sharing.
    - AWS service: [AWS CloudHSM](#aws-cloudhsm)
  - **Key management services (KMS)**
    - Also known as a cryptographic key management service (CKMS)
    - Enables clients to manage encryption keys without concerns about HSM appliance selection or provisioning
    - Usually come with more scalability, availability, native integration with other services such as databases, Bring Your Own Key (BYOK) that allows you use external HSM for master keys.
    - AWS service: [AWS KMS](#aws-key-management-service)

### Encryption Types

- **Encryption in-transit**
  - Data is encrypted before sending and decrypted after receiving
  - TLS certificates help with encryption (HTTPs = SSL enabled)
  - Protects when more parties are involved
    - I.e. No one inspecting the traffic can see data
  - ***Flow***
    - You send data TLS encrypted with key and over HTTPS
    - Server (HTTPS website) decrypts data with SSL key.
- **Encryption at-rest**
  - Protects usually against one party
  - **Server side encryption at rest**
    - Encryption/decryption is handled by server-side
      - Data is encrypted after being received by the server
      - Data is decrypted before being sent
    - Used by many AWS services
    - It's stored in an encrypted form thanks to a key (usually a ***data key***)
    - The encryption / decryption keys must be managed somewhere and the server must have access to it.
      - Usually stored in KMS (Key Management Service)
    - ***Flow***
      - Object is being sent using HTTP(s)
        - Server (e.g. EBS AWS service) encrypts object with data key & saves it
      - Object is requested using HTTP(s)
        - Server decrypts object with data key & sends it
    - ❗ Requires migration (through Snapshot Backup) to enable/disable in EBS Volumes, [RDS databases](./06-02-01-data-databases-rds.md), ElastiCache, [EFS](./05-01-06-compute-ec2-ebs-efs-instance-store.md#amazon-elastic-file-system)
    - In S3, it's in-place e.g. you encrypt & decrypt with parameters.
  - **Client side encryption**
    - Data is encrypted by the client and never decrypted by the server
    - Data will be decrypted by a receiving client
    - The server should never be able to decrypt the data
    - 📝 Can leverage ***Envelope Encryption***
      - You encrypt encryption key with **Envelope Key** in KMS.
      - You manage encryption key, AWS manages Envelope Key.
    - ***Flow***:
      - Client encrypts object with *Client side data key* & sends
      - Object is stored in encrypted form in AWS
      - Client retrieves encrypted object and decrypts with *Client side data key*

## AWS Services

- AWS credentials - Use [AWS IAM](./02-01-security-iam.md) instead.
- Encryption keys - Use [AWS Key Management Service](#aws-key-management-service).
- Private keys and certificates - Use [AWS Certificate Manager](#aws-certificate-manager).
- Automatically rotating keys - Use [AWS Secrets Manager](#aws-secrets-manager)

### AWS CloudHSM

- Cloud-based hardware security module (HSM)
- Single-tenant access to HSM, hardware may be shared.
- Enables you to easily generate and use your own encryption keys on the AWS Cloud.
- Primarily intended to support customer-managed applications that are specifically designed to use HSMs
- CloudHSM cluster contains at least two HSMs => minimum $1,000 per month.
- Can have automatic back-ups
- If you lose the keys, there's no way to recover

### AWS Key Management Service

- Also known as *AWS KMS*
- Fully managed service for generating, distributing and managing cryptographic keys.
- Help you generate, store, audit, control cryptographic keys.
- KMS uses HSM (hardware security modules) to protect and validate your keys.
  - KMS manages the software for encryption
  - But on CloudHSM, AWS provisions the hardware but user manages the software for encryption.
  - KMS is Level-2 FIPS compliant, CloudHSM is Level-3 FIPS compliant.
- **Key Management**
  - Use AWS KMS key policies to control access to the AWS KMS keys.
  - Maintenance of: *key policies*, *IAM policies*, *enabling/disabling them*, *rotating*, *adding tags*, *creating aliases*, *scheduling for deletion*.
    - **Rotation policies** e.g. key must be changed ever year
- **Multi-Region keys**
  - 📝 Shares same key ID and can be used to encrypt/decrypt data in multiple regions.
  - Can be used with AWS services that integrates with KMS, or using AWS SDKs.
  - You can convert an existing single-Region key to a multi-Region key
- **Monitoring**:
  - Audit key usage using CloudTrail
- **Integrations:**
  - Some of integrated services:
    - [EBS](./05-01-06-compute-ec2-ebs-efs-instance-store.md#ebs-elastic-block-store)
    - [S3](./06-01-01-data-s3-overview.md) ([SSE-KMS](./06-01-02-data-s3-security-macie.md#s3-encryption))
    - [Redshift](./06-03-01-data-analytics-warehousing-redshift.md)
    - [RDS](./06-02-01-data-databases-rds.md)
    - [IAM](./02-01-security-iam.md) for authorization
      - 💡 Can add external accounts that can use the key.
  - 💡 Managed KMS key per service is created for AWS services.
    - E.g. one for CodeCommit, one for [RDS](./06-02-01-data-databases-rds.md), one for S3, one for Lambda...
- Has API's in CLI /SDK
- **KMS Keys**

  | Type | Can view | Can manage | Account-specific | Price |
  | ---- | ---- | ------ | ------------------- | -- |
  | Customer managed KMS | ✔️ | ✔️ | ✔️ |  Monthly fee key + usage |
  | AWS managed KMS | ✔️ | ❌ | ️✔️ | No monthly fee, just usage |
  | AWS owned KMS | ❌ | ❌ | ❌ | Free |

  - ❗ The value in KMS is that the *KMS Key* used to encrypt data can never be retrieved by the user.
  - The KMS Key can be rotated for extra security.
  - 🤗 Formerly known *Customer Master Keys - CMKs* [REBRANDED 2021]
- Encrypted secrets can be stored in the code / environment variables
  - E.g. **encryption helpers** in lambda environment variables.
  - No one can decrypt without having access through IAM
- ❗ KMS can only encrypt up to 4KB of data per call
  - 📝 If data > 4 KB, use envelope encryption
  - Envelope encryption = Encrypt your key (data key) using master key.
- To give access to KMS to someone
  - Make sure the Key Policy allows the user
  - Make sure the IAM Policy allows the API calls
- ***Flow***
  - ***Encrypt API***: *KMS* checks IAM permissions & performs encryption & sends encrypted secret
  - ***Decrypt API***: *KMS* check IAM permissions & performs decryption & sends decrypted secrets (in plain-text)
- **Custom Key Store**
  - Feature that allows managing KMS keys independently of AWS KMS
  - Can be:
    - AWS CloudHSM key stores
      - Keys are stored in AWS CloudHSM clusters
    - External Key Stores (XKS)
      - Keys can be stored in external key management systems outside of AWS

### AWS Systems Manager Parameter Store

- Secure storage for configuration and secrets
- (*Optional*) Seamless Encryption using KMS
  - Requires KMS policy to be activated
- Serverless, scalable, durable, easy SDK, free
- Version tracking of configurations / secrets
- Configuration management using path & IAM
  - 💡 Best practice to give IAM role for reading only the needed parameter
  - For plain text configurations it's enough to give SSM read permission
    - For decrypted parameters, you also need to give IAM permission to KMS.
- Notification with [CloudWatch Events](./03-01-monitoring-cloudwatch-grafana.md#cloudwatch-events)
- Integration with [CloudFormation](./07-05-resiliency-infrastructure-as-code-cloudformation-cdk-sam.md#cloudformation)
- ***Flow***:
  - *Application* sends encrypted configuration to *SSM Parameter Store*
  - *SSM Parameter Store* checks IAM permissions
  - *SSM Parameter Store* decrypts configuration using AWS KMS
  - *Application* receives plain-text configuration from *SSM Parameter Store*.
- Supports hierarchy, e.g. `my-department/my-app/dev/db-url`.
- Configurations are retrieved using `GetParameters` or `GetParametersByPath` API.
- Two ways of accessing from portal:
  1. *EC2 Systems Manager - Parameter Store*
  2. from *AWS Systems Manager* -> Parameter Store.
- Two tier of parameters:

  | Feature | Standard | Advanced |
  | ------- | -------- | -------- |
  | Max parameter | 10,000 | No limit |
  | Max value size | 4 KB | 8 KB |
  | Parameter policies | None | Available |
  | Pricing | Free | Charges apply |

  - Parameter must have:
    - Name (e.g. `/my-app/dev/db-url`)
    - Type: `String`, `StringList` (comma separated), `SecureString`

### AWS Secrets Manager

- Helps you manage, retrieve and rotate secrets
- Can rotate on demand or on schedule automatically.
- Uses [AWS KMS](#aws-key-management-service) to encrypt data across various AWS services.
- Integrates with [IAM](./02-01-security-iam.md)
- Can be integrated in other AWS services logging/monitoring and notification services

### AWS Secrets Manager vs AWS Systems Manager Parameter Store

- Encryption
  - SM: Always encrypted
  - PS: Optionally
- Automatic secret rotation
  - SM: Supports
  - PS: Manually handled
- Common:
  - Store secrets
  - Encrypt
  - Cross-account access [1]

### AWS Certificate Manager

- Also known as *AWS ACM*
- Create and manage public SSL/TLS certificates for AWS-based apps/websites.
- Allows multiple domains to serve SSL traffic over the same IP address.
- 📝 Allows wildcard certificate
  - Public key certificate which can be used with multiple sub-domains of a domain.
  - Protect multiple sub domain names e.g. `*.domainname.com`
  - 💡 For multiple root domain names you need `SNI` through [CloudFront](./04-06-02-networking-edge-cloudfront.md) or ELB.

[1]: https://aws.amazon.com/about-aws/whats-new/2024/02/aws-systems-manager-parameter-store-cross-account-sharing/ "What's New at AWS - Cloud Innovation & News | aws.amazon.com"
