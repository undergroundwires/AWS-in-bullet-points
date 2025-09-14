
# EC2 Networking

## Enhanced Networking

- A feature can be enabled on specific instances with a driver to reduce networking overhead + faster networking.
- 📝 Provides Single Root I/O Virtualization (SR-IOV)
- Utilizes Elastic Network Adapter (high Performance Network Interface for Amazon EC2)
- Free
- ❗ Limitations
  - Instances must be launched in a VPC
  - Instances must be launched from a HVM AMI
- **Elastic Network Adapter**: High performance
- **Elastic Fabric Adapter**: High Performance Computing
  - ❗ Linux machines only

## Elastic Network Interface (ENI)

- Each EC2 has one default ENI.
- Each ENI have
  - one primary private IPv4, can have additional private IPv4 (optional)
  - one **Elastic IP** (static IP) per private IPv4 (optional)
  - one public IPv4
  - one or more IPv6
  - one or more security group
  - a mac address
  - source destination check flag
  - description of ENI
- Can attach a secondary network interface
- You can attach a network interface to an EC2 instance in the following ways:
  - **Hot attach**: When it's running
  - **Warm attach**: When it's stopped
  - **Cold attach**: When the instance is being launched
- Termination with instance termination
  - Default interfaces are terminated with instance termination.
  - Manually added interfaces are not terminated by default

## Security Groups

- Virtual firewall for EC2 for inbound/outbound traffic.
- Attached to Elastic Network Interfaces (ENI) of EC2 instances or other resources
  - Have an explicit deny, meaning anything not explicitly allowed will be denied
- Controls authorized IP ranges (IPv4 and IPv6) and ports with:
  - **Inbound rules**
    - From other to the instance
    - All inbound rules are denied by default
    - You define protocol (e.g. TCP) & port range & source
      - **Source**
        - 📝Refer another security group (type in Security Group ID)
        - An IPv4 or IPv6 CIDR block
        - A single IPv4 or IPv6 address (e.g. anywhere: `0.0.0.0/0` for IPv4 and `::/0` for IPv6)
  - **Outbound rules**
    - From the instance to other
    - Open to everywhere by default
    - You define protocol (e.g. TCP) & port range & destination.
      - **Destination**
        - 📝Another security group
        - An IPv4 or IPv6 CIDR block
        - A single IPv4 or IPv6 address
        - An AWS service (i.e. prefix list ID)
- Can be attached to multiple instances
- 📝 Security groups are stateful
  - Once allowed inbound, connection passes through outbound and vice versa
  - Making it simpler than NACL, less rule to manage
- An instance can have multiple security groups
- Locked down to a region / VPC combination
  - ❗ You need to recreate security groups if you change region / VPC.
  - You can create a snapshot => Create AMI and use AMI to launch in other AZ or copy AMI to new region and launch there.
- 💡 Good practice: maintain one separate security group for SSH access.
- 📝 Any ***timeout errors*** mean a misconfiguration of your security groups
- If your application gives "connection refused" error, then it's an application error or it's not launched
- Being able to reference security groups allows:
  - Easier maintenance as you don't need to work with IP's.
  - E.g. allow EC2 instances to connect with each other by authorizing their attached security groups.
- ❗ You can only delete security group after you delete all EC2 instances associated with the security group.
- ❗📝 You cannot block single IP/IP-range with Security Groups, you'll need NACL (Network Access Control Lists) for it.
  - Security groups can only "allow" services through a default deny
