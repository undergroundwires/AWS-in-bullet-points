
# Network Security

| Service | Protection Layer | Scope | VPC-Internal? |
| ------- | ---------------- | ----- | ------------- |
| [NACLs](#network-access-control-lists-nacls)/[Security Groups](#security-groups) | L3-L4 | Subnet/Instance | ✅ Yes |
| [AWS Network Firewall](#aws-network-firewall) | L3-L4 | VPC/Subnet | ✅ Yes |
| [AWS WAF](#aws-waf) | L7 | ALB/API Gateway/[CloudFront](./04-06-02-networking-edge-cloudfront.md) | Via ALB/API Gateway |
| [AWS Shield](#aws-shield) | L3-L7 | Edge/VPC resources | ✅ Yes (via ALB) |

## Visibility

- [Gateway Load Balancer (GWLB)](./07-02-resiliency-load-balancers-elb-clb-nlb-alb-gwlb.md#gateway-load-balancer-gwlb)
  - Helps centralize traffic inspection
- [Amazon GuardDuty](./03-02-monitoring-security-cloudtrail-aws-config-trusted-advisor-inspector-security-hub-artifact-detective-audit-manager.md#amazon-guardduty)
  - Leverages VPC Flow Logs to detect network-based threats (e.g., unusual traffic patterns)

### Flow Logs

- ![VPC flow logs](img/networking/vpc-flow-logs.png)
- Capture information about IP traffic going into your interfaces:
  - VPC Flow Logs
  - Subnet Flow Logs
  - Elastic Network Interface Flow Logs
- Can capture from managed services such as ELB, [RDS](./06-02-01-data-databases-rds.md), ElastiCache, Redshift, WorkSpace
- Helps to monitor & troubleshoot connectivity issues & security attacks
  - Can log ACCEPT traffic, REJECT traffic or both.
- Data is sent to either S3 or CloudWatch Logs
  - 📝 You can query VPC flow logs using [Athena](./06-01-03-data-s3-integrated-services-snowball-athena-storage-gateway-object-lambda-transfer.md#amazon-athena) on S3 or ***CloudWatch Logs Insights***
- **Flow Log Syntax**
  - `<version>`, `<account-id>`, `<interface-id>`, `<srcaddr>`, `<dstaddr>`, `<srcport>`, `<dstport>`, `<protocol>`, `<packets>`, `<bytes>`, `<start>`, `<end>`, `<action>`, `<log-status>`
    - `<srcaddr>` and `<dstaddr>` help identify problematic IP
    - `<srcport>` and `<dstport>` help identify problematic ports
    - `<action>`: success or failure (accept / reject) of the request due to  Security Group / NACL
    - `<log-status>`: `OK` -> logging normally, `NODATA`: No network traffic to or from during capture window, `SKIPDATA`: Skipped during capture windows because of e.g. capacity constraint or an internal error.
- Can be used for analytics on usage patterns, or malicious behavior
- ❗ You cannot allow it in peered VPCs unless the peer is in your account.
- ❗ Does not monitor: instances when they contact Amazon DNS server, Windows license activation, to and from `169.254.169.254` instance metadata, DHCP, to the reserved IP address for the default VPC router.

## Package security

![Package security](img/networking/package-security.png)

### Network Access Control Lists (NACLs)

- Like firewall which control traffic from and to subnets.
- Deployed into VPC.
- NACLs are ***stateless*** filtering rules
  - Stateless because if an ingress traffic is allowed, the response is not automatically allowed unless explicitly allowed in the rule for the subnet.
- Applied at the subnet level and applies to every resource deployed to the subnet
  - A subnet can have only one NACL
  - The same NACL can be applied to multiple subnets
- ***NACL Rules***
  - Rules are numbered and evaluated in order
  - Starting with the lowest numbered rule
    - E.g. `#100 ALLOW <IP>` override `#200 DENY <IP>`
    - Last rule is an asterisk (*) and denies a request in case of no rule match
    - 💡 AWS recommends adding rules by increment of 100 for leaving some places for future rules
- Default NACL allows everything outbound and everything in bound
  - ❗ Newly created NACL will deny everything
- **Ephemeral ports**
  - An ephemeral port is a short-lived transport protocol port for Internet Protocol (IP) communications.
  - Ephemeral ports are allocated automatically from a predefined range by the IP stack software.
  - Allows returning traffic to clients
  - Range varies depending on the client's operating system
    - E.g. Linux 32768 - 61000, Elastic Load Balancing 1024 - 65535, Windows Server 49152 - 65535
    - Different OS and different applications will pick different ports.
  - To cover the different types of client that might initiate traffic to public-facing instances in your VPC, you can open ephemeral ports 1024 - 65535 (NAT Gateway ports) to outbound connections.
    - You should deny malicious ports

### Security Groups

- ***Stateful*** e.g. if inbound rule passes, outbound rule passes as well
- Applied at the EC2 instance level
  - More specific: Elastic Network Interface (ENI) level
- See: [Security Groups](./05-01-03-compute-ec2-networking.md#security-groups)

### Security Groups vs Network Access Control Lists

- Differences

  | Attribute | Security Group | NACL |
  | --------- | -------------- | ---- |
  | Execution | Instance (ENI layer) | Subnet boundary |
  | State | Stateful (one direction rules) | Stateless (must configure both inbound & outbound) |
  | Life scope | VPC (any AZ or Subnet) | Subnet Scoped (must be attached explicitly to subnets) |
  | Applies to | Instance only | All instances in the associated subnets |
  | Rules | Allow rules only | Allow and Deny rules both |
  | Rule order | Evaluate all rules as group | In number order, if hit a match, does not check other rules |

- Reasons to use both:
  - SG have soft limit of 50 rules.
  - More security layers = better, protection against misconfigurations
  - Segregation of responsibilities, Network Admins for NACLs, SGs for Server Admins.

## Traffic Mirroring

- VPC feature that copies network traffic from elastic network interface.
- Enables
  - deep packet inspection,
  - threat monitoring,
  - and troubleshooting without impacting production workloads.

## AWS Network Firewall

- 📝 Managed stateful firewall service protecting VPCs with granular traffic controls.
- Features:
  - intrusion prevention,
  - web filtering,
  - centralized rule management.
- Supports HTTP header inspection with FQDN filtering.
- Integrates with [AWS Firewall Manager](#aws-firewall-manager) for policy enforcement.

## AWS Firewall Manager

- Centralized security policy (firewall rule) management service for
  - AWS WAF
  - [Network Firewall](#network-security),
  - [Shield](#aws-shield)
  - [Security Groups](#security-groups)
- Automates rule deployment across accounts/resources to ensure consistent security posture.
- Simplifies compliance and auditing.

## AWS Shield

- Managed Distributed Denial of Service (DDoS) protection service
- Offerings:
  - AWS Shield Standard
    - Provides free basic DDoS protection for all AWS customers.
  - AWS Shield Advanced:
    - Offers enhanced DDoS protection with
      - 24/7 support,
      - cost protection,
      - advanced attack diagnostics
    - 📝 Provides integrations with other AWS services:
      - Elastic IP
      - ELB
      - [CloudFront](./04-06-02-networking-edge-cloudfront.md)
      - [Global Accelerator](./04-06-03-networking-edge-optimization-aws-global-accelerator-wavelength.md#aws-global-accelerator),
      - [Route 53](./04-06-01-networking-edge-route-53.md)
      - [AWS WAF](#aws-waf) is included with *Shield Advanced* at no extra cost

## AWS WAF

- Application-layer (L7) security for web applications.
- Used to secure web apps, APIs, and microservices.
- Protects against: OWASP Top 10 threats (e.g., SQL injection, XSS), bots, and API abuse.
- Both regional/global service depending on what's protected:
  - Regional service for protecting regional resources
    - E.g. ALB, API Gateway, AppSync, Amazon Cognito
    - Must create separate WAF web ACLs in each region where you have resources
  - Global for CloudFront
    - Deployed from the `us-east-1` (N. Virginia) region but protects CloudFront distributions worldwid
- Blocks, allows, and rate-limits traffic based on rules
  - Custom rules + managed rule groups (e.g., AWS, Marketplace)
  - Rules are globally scaled: Your rules run in all AWS Edge Locations
  - Match rule examples:
    - Cross-site scripting
    - IP match
    - Geo match
    - Size constraint match
    - SQL injection match
    - String match
    - Regex match
  - Can rate-limit the maximum number of requests from a single IP address
- Deployment Targets:
  - *VPC-Internal:*
    - Application Load Balancers (ALB)
    - API Gateway
  - *Edge:*
    - [CloudFront](./04-06-02-networking-edge-cloudfront.md) distributions,
    - [AppSync](./08-03-integrations-api-gateway-appsync.md#aws-appsync)
    - [Cognito](./02-02-security-identity-federation-sts-saml-cognito-directory-service-identity-center.md#amazon-cognito)
- Integrations:
  - Integrates with ALB, API Gateway and [CloudFront](./04-06-02-networking-edge-cloudfront.md) but accepts also custom origin.
  - Integration with [AWS Firewall Manager](#aws-firewall-manager) for centralized policy management
  - Included with [AWS Shield](#aws-shield) Advanced for DDoS protection
