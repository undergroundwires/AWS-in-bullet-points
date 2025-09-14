# Subnets

- 📝 A subnet must only belong to one AZ and cannot span AZs.
  - ![VPC and network placement](img/networking/vpc-and-subnet-placement.png)
- ❗ Max allowed CIDR range is `/16`, min is `/28`
- ❗ AWS reserves 5 IP addresses (first 4 and last 1 IP address) in each Subnet
  - These 5 IPs are not available for use and cannot be assigned to an instance
  - E.g. if CIDR block `10.0.0.0/24`, reserved IP are:
    - `10.0.0.0`: Network address
    - `10.0.0.1`: Reserved by AWS
    - `10.0.0.2`: Reserved by AWS for mapping to Amazon-provided DNS
    - `10.0.0.3`: Reserved by AWS for future use
    - `10.0.0.255`: Network broadcast address.
      - AWS does not support broadcast in a VPC but address is reserved.
  - 📝 E.g. if need 29 IP addresses for EC2 instance, you can't choose a Subnet size of /27 (32 IP)
  - Each instance in any subnet is assigned a static private address, i.e. does not change with stopping/starting instances.
- 🤗 Penetration testing & vulnerability scanning to own VPC is allowed!

## Route Table

- Deployed into VPC
- When you create a VPC, it automatically has a ***main route table***.
- Each subnet must have a route-table attached to it.
  - They're associated to main route table if they don't have any.
- Any time when a route destination is down/terminated, the status of the route becomes ***blackhole***.
- Default route in route table is destined for the IP range of the VPC that allows all subnets within a VPC to communicate with each other.
- Route rules
  - Defined destinations (where packet wants to go) and targets (where it will go).
  - Priority: most specific range gets selected if overlapping i.e. **longest prefix match**.
- **Route propagation**: It is the task of your service provider to advertise to the backbone sites that they are the point of connection (and thus the path inward) for your site. This is known as route propagation.

## Private Subnet

- Any subnet without the IGW is regarded as private subnet and have no internet connectivity without NAT gateway.
- Bigger in size as you put all your applications etc.
- Allowing internet traffic:
  - Can deploy [Egress-Only Internet Gateway](./04-02-networking-vpc.md#egress-only-internet-gateway) to allow internet traffic.
    - But it's IPv6 only and attached on VPC level
  - 💡 Use [NAT Gateways](#nat-gateway) instead:
    - Spports IPv4 internet access from private subnets.
    - Flow:
      - Create NAT gateway in public subnet.
      - Configure route tables to point to NAT gateway.

### NAT

- The actual role of a NAT device is address and port address translation (PAT)
- Enables private subnet instances to access internet/AWS services
  - Like a one-way door - private instances can "go out" but nothing can "come in" from the internet.
- Direction:
  - ✅ Outbound: Private instances → Internet (allowed)
  - ❌ Inbound: Internet → Private instances (blocked)
- Common uses:
  - Software updates
  - API calls
  - Downloading packages
- Keeps instances private while allowing necessary outbound access
- Types: [NAT Gateway](#nat-gateway) (managed) or [NAT Instance](#nat-instances) (self-managed)
  - ❗ Both must be deployed in public subnet.

#### NAT Instances

- Outdated (legacy since 2015) and not recommended
  - Ue [NAT Gateway](#nat-gateway) instead
- EC2 instance that allows instances in the private subnets to connect to the internet
  - Amazon Linux AMI pre-configured are available

#### NAT Gateway

- AWS managed NAT, higher bandwidth, better availability, no administration
- Goal = Make private subnets access the internet
  - Public subnets already have [Internet Gateway](#internet-gateway)
- Deployed into public subnet
  - Private subnet route table is configured to point to it
  - Setup:

    ```txt
        Internet
            ↕
        Internet Gateway
            ↕
        Public Subnet (contains NAT Gateway with Elastic IP)
            ↕
        Private Subnet (route table points to NAT Gateway)
    ```

- NAT is created in a specific AZ, uses an EIP (Elastic IP)
  - 💡 For high availability, it's good to deploy different NAT gateways in different AZ.
- Cannot be used by an instance in that subnet (only from other subnets)
- Requires an IGW (internet gateway): Private Subnet => NAT => GW
- **Scalability**
  - 5 GBPS of bandwidth
  - ❗ Automatic scaling up to 45 GBPS
- No security group to manage
- Pay by the hour for usage and bandwidth
- **Security**
  - Use [NACL](./04-04-networking-security.md#network-access-control-lists-nacls) to control traffic to and from the subnet in which your NAT gateway resides.

#### NAT Gateway vs NAT Instance

| Attribute | [NAT Gateway](#nat-gateway) | [NAT instance](#nat-instances) |
| --------- | ----------- | ------------ |
| **High availability** | highly available in each AZ with redundancy | Use a script to manage failover between instances |
| **Bandwidth** | Can scale up to 45 Gbps | Depends on the bandwidth of the instance type |
| **Maintenance** | Managed by AWS | Managed by you |
| **Performance** | Software optimized | Generic Linux AMI |
| **Security Groups** | Cannot be associated (use NACL in its subnet) | Can be associated with |

## Public Subnet

- Usually much smaller in size and contains load balancers only.
- A subnet is public if it has an [Internet Gateway](#internet-gateway) attached.
- You can enable ***auto-assign public IPv4 address***
  - A new EC2 instance by default gets an IP address
  - By default instances do not get a public IP if it's not enabled.

### Internet Gateway

- Also known as *IGW*
- Allows VPC to connect with the internet
  - Route tables must be edited: Destination 0.0.0.0/0 to Internet Gateway
- It scales horizontally and is HA and redundant
- ❗📝 One VPC can only be attached to one IGW and vice versa
- It's **NAT** for instances that have a public IPv4
  - NAT translates the IP addresses of computers in a local network to a single IP address.
- ❗ Edit the route tables, after creating; outbound route to destination (`0.0.0.0/0` and `::/0`), target: gateway

## Gateway Comparison

| Type | Subnet accessibility | When |
| ---- | ---------- | ---- |
| Egress only | Private subnet | IPv6 internet in private subnet. |
| NAT Gateway | Private subnet | IPv4 internet in private subnet. |
| Internet Gateway | Public subnet | Once for VPC for internet access in VPC. |
