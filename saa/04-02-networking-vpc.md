# Amazon VPC

- VPC = Virtual Private Cloud
- ❗ You can have max 5 VPCs in a region - soft limit
- ❗ Your VPC CIDR should not overlap with your other networks (e.g. corporate)
- VPC can be deployed into shared hardware (default) or dedicated hardware (costly).
- Regional: A VPC is multi-AZ but deployed into a single region.
- **IP addressing in VPC**
  - Only private IP ranges as it's private.
  - Can add new ranges as IPv6 or IPv4 CIDR after creating the VPC.
  - ❗ Max CIDR per VPC is 5.
  - ❗ For each CIDR:
    - Min size is /28 = 16 IP addresses
    - Max size is /16 = 65,536 IP addresses
- **Default VPC**
  - All new accounts have a default VPC in each region.
  - Maximum one per region
  - It comes with
    - Subnet per AZ
    - Internet gateway
    - Network ACL: allows all inbound and outbound traffic
    - Main route table that routes VPC CIDR range to local and `0.0.0.0/0` to Internet gateway.
    - Security group denying inbound from internet, allowing outbound to internet.
  - Instances get public & private IPv4 + hostnames
  - Can be deleted and recreated
  - Default VPC CIDR is always: `172.31.0.0/16`
    - `/20` subnet is created in each AZ in the region
  - 💡 Best practice is to not use Default VPC but custom VPCs
- **Custom VPC**
  - Private by default:
    - Provides private IP + hostname, and public depending on settings
  - Can have multiple VPCs per account
- **VPC Components**
  - ![VPC network components](img/networking/networking-components.png)
- **DNS Resolution in VPC**
  - `enableDnsSupport` (DNS Resolution setting)
    - Helps decide if DNS resolution is supported for the VPC
    - If true (default), queries the AWS DNS server at `169.254.169.253`
  - `enableDnsHostname` (DNS Hostname setting)
    - If True, assign public hostname to EC2 instance if it has a public IP
      - Private DNS is always assigned regardless true or false
    - False by default for newly created VPC, True by default for Default VPC
    - Won't do anything unless `enableDnsSupport=true`
  - 📝 Private DNS + [Route 53](./04-06-01-networking-edge-route-53.md): enable DNS Resolution + DNS hostnames (VPC) DNS domains and their subdomains keeping the resources masked from the Internet.
- ❗ **Multicast** is not supported.
  - Multicasting: one or more sources can transmit network packets to subscribers
  - 💡 You can enable virtual overlay network
    - It's IP level multicast across network fabric unicast IP routing is supported by VPC.

## Inter-VPC Connectivity

### VPC Peering

- 💡 Prefer [Transit Gateway](#aws-transit-gateway) as default choice for most use cases for simplicity.
- Connect two VPC, privately using AWS network
- Make them behave as if they were in the same network
- ❗ Unsupported VPC Peering Configurations
  - Must not have overlapping CIDR
  - VPC Peering connection is not transitive
    - Must be established for each VPC that need to communicate with one another
    - E.g. if a <-> b and b <-> c it does not mean a <-> c
  - Edge to Edge Routing Through a Gateway or Private Connection
    - E.g.
      - Corporate network through Direct Connect
      - Internet through Internet Gateway
      - Internet in subnet through NAT
      - [VPC endpoint](#vpc-endpoints) to an AWS service and Ipv6 ClassicLink connection
- You can do ***cross-account*** & ***cross-region*** VPC peering
- You must update route tables in each VPC's subnets to ensure instances can communicate,
- The accepter VPC must "accept" the peering connection,
- Flow in console:
  1. Go to VPC -> Peering connections -> Create ***peering connection***
  2. Define requester & accepter VPC.
  3. Go to accepter VPC -> right click -> accept request
  4. In each VPC, route CIDR range of other VPC to the ***Peering Connection***

### AWS Transit Gateway

- Connects VPCs, AWS accounts and on-premises networks
- Simplifies network without complex peering
- Acts as router, each new connection is made only once.
- No need to manage peering, updating route tables.

### VPC Peering vs Transit Gateway

- Decision:
  - **Choose VPC Peering:** ≤3 VPCs, cost-sensitive, performance-critical, simple requirements
  - **Choose Transit Gateway:** ≥3 VPCs, need centralized control, complex routing, hybrid connectivity
- Comparison:

  | Aspect | VPC Peering | AWS Transit Gateway |
  |------------|-----------------|--------------------------|
  | **Best for** | 2-3 VPCs, simple connectivity | 3+ VPCs, complex architectures |
  | **Topology** | Full mesh (each VPC peers directly) | Hub-and-spoke (centralized) |
  | **Scaling** | O(n²) connections - becomes unmanageable | O(n) connections - scales linearly |
  | **Cost** | Lower for small setups | Higher base cost, more economical at scale |
  | **Latency** | Lowest (direct connection) | Slightly higher (one additional hop) |
  | **Bandwidth** | No limits | 50 Gbps per attachment limit |
  | **Routing** | Static, point-to-point | Dynamic, centralized route tables |
  | **Management** | Distributed across peering connections | Centralized control and policies |
  | **Transitive routing** | Not supported | Fully supported with route control |
  | **Cross-region** | Limited to 1:1 inter-region peering | Native multi-region support |
  | **On-premises connectivity** | Requires separate VPN/DX per VPC | Single VPN/DX connection to all VPCs |
  | **Route table segmentation** | Not available | Advanced traffic isolation capabilities |
  | **Multicast** | Not supported | Supported |
  | **Internet egress centralization** | Complex to implement | Built-in with inspection VPCs |
  | **Monitoring** | Per-peering connection basis | Centralized flow logs and metrics |
  | **Setup complexity** | Simple for few VPCs | More complex initial setup |
  | **Operational overhead** | High with many VPCs | Low once configured |
  | **Security** | Network ACLs and security groups | Additional route-based controls |

## AWS PrivateLink

- Provides private connectivity between VPCs and services
- Underlying technology behind [VPC Endpoints](#vpc-endpoints)
- Created inside VPC, to connect AWS services without internet/NAT gateway

### VPC Endpoints

- Allow you to connect to AWS Services using AWS network instead of the public Internet.
- They scale horizontally and are redundant
- They remove the need of IGW, NAT etc... to access AWS Services
- Powered by [AWS PrivateLink](#aws-privatelink)
- Eliminates need to set-up:
  - Internet access for communication between VPCs
  - Enabling VPC peering that causes
    - management overhead as architecture scales
    - all peered VPCs to see each other
- Two types of endpoints:
  - **Interface**
    - Also known as *Interface Endpoints*
    - Provisions an ENI (Elastic Network Interface with private IP address) as an entry point
    - Must attach security group
    - Most AWS services
      - Used for AWS resources not supporting gateway endpoints (anything except S3 or DynamoDB)
      - Services hosted by other AWS customers and partners in their own VPCs
      - AWS marketplace services
      - E.g. EC2, Lambda, SNS, SQS, Systems Manager, Secrets Manager, CloudWatch, KMS.
    - You can see the resource but can't manage
    - You have to define which subnets the endpoint will live.
    - Ensure that the attributes `Enable DNS hostnames` and `Enable DNS Support` in VPC are set to true.
    - Interface endpoints use [security groups](./04-04-networking-security.md#security-groups), not policies.
  - **Gateway**
    - Also known as *Gateway Endpoints* or *Gateway VPC endpoints*
    - S3 and DynamoDB only (public services)
    - They use routing, and need an entry on the route table.
      - In route table it provisions a target for internal S3/DynamoDB to the VPC endpoint automatically.
    - Ensure resources accessing S3 (e.g. EC2) has IAM role to the resource
    - Single endpoint serves single service
      - ❗ Cannot use same Gateway Endpoint for S3+DynamoDB
      - ❗ Cannot create endpoints per service (e.g. one per table/bucket)
        - ❌ VPC → 4 Gateway Endpoints (S3) → Individual Policies → Individual Buckets
        - ✅ VPC → 1 Gateway Endpoint (S3) → Endpoint Policy → Multiple Trusted Buckets
    - Can be restricted using endpoint policies.
      - Endpoint policy = WHO can access WHAT behind this endpoint
    - You need route table entry.
      - Yo must select which route tables to associate, AWS automatically creates the routes.
      - Example:

        ```txt
          Destination: pl-12345678 (S3 prefix list)
          Target: vpce-12345678 (your gateway endpoint)
        ```

- In case of issues:
  - Check DNS Setting Resolution in your VPC
  - Check route tables
  - Ensure you're trying to reach endpoint within same region

## Connect & manage private instances

- **Bastion Host** (Jump Boxes)
  - Designed to withstand security attacks and usually hosts only a proxy server
  - Used to administer EC2 (SSH/RDP)
    - Does not replace NAT gateway/instance as they are not used for SSH/RDP but to provide internet traffic to private subnets.
  - The bastion is in the public subnet (DMZ) which is then connected to all other private subnets
  - 💡 Small EC2 instances are enough as they do not require much processing power.
  - Bastion Host security group must be tightened
    - 📝💡 Make sure the bastion host only has port 22 traffic from the IP you need, not from the security groups of your instances

- **SSM Session Manager**
  - Systems Manager allows you to remotely execute commands on managed hosts without using a bastion host
    - Way to go in future and will replace bastion hosts
    - You don’t have to manage SSH keys.
  - Free & serverless AWS managed service
  - Requires an IAM policy that allows users or roles to execute commands remotely.
  - Agents require an IAM role and policy that allow them to invoke the Systems Manager service.
    - The Systems Manager agent is an open-source executable that runs on supported versions of Linux and Windows
  - Systems Manager immutably logs every executed command

## Egress-Only Internet Gateway

- Also known as "EGW"
- Attached at the VPC level not on subnet.
- Blocks incoming traffic while still allowing outbound traffic
- ❗ IPv6 only.
  - All IPv6 are public addresses; instances become publicly accessible.
  - Use [NAT Gateway](./04-03-networking-vpc-subnets.md#nat-gateway) for IPv4.
- ❗ Edit the route tables, after creating; outbound route to destination (`::/0`), target: gateway
