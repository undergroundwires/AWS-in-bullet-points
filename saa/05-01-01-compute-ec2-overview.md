# EC2 - Elastic Compute Cloud

- Virtual machines on AWS
- Some capabilities/integrations:
  - **EC2**: Renting virtual machines
  - **EBS**: Storing data on virtual drives
  - **ELB**: Distributing load across machines
  - **ASG**: Scaling the services using an auto-scaling group

## EC2 Pricing

- Billed only when in `running` state.
  - And `stopping` state only if it's hibernating.
  - Also `stopped` state for reserved instance.
- Billed by the hour or the second, depending on which AMI you're running
- Free tier: `t2.micro` is free
- **Data transfer costs**
  - Free between EC2 and other services.
  - Free between EC2 instances in same availability zone using private IP addresses only.
  - $0.01/GB for transfer between instances in different availability zones
  - See also [data transfer costs](./01-02-01-aws-management-economics-pricing-pricing-calculator-data-transfer-savings-plans.md#data-transfer-charges)

## EC2 Availability

- EC2 instances run in a single availability zone (AZ).
- **Multi-AZ patterns:**
  - Common patterns:
    - Launch EC2 instances across multiple AZs
    - Use Application Load Balancers can route traffic across instances in multiple AZs
    - Use [Auto Scaling Groups](./07-03-resiliency-scalability-auto-scaling-asg.md#asg-auto-scaling-group) to automatically distribute instances across AZs
      - 💡 Easier to manage as remediation it is automated
  - Latency: Low (sub-millisecond)
- **Multi-Region:**
  - Common patterns:
    - Route 53 latency-based routing or geolocation routing.
    - Route 53 health checks with failover routing.
    - [AWS Global Accelerator](./04-06-03-networking-edge-optimization-aws-global-accelerator-wavelength.md#aws-global-accelerator) for performance optimization.
  - ❗ Must manage data replication and synchronization
  - ❗ VPCs are region-specific, so multi-region deployments require VPC peering, Transit Gateway, or internet routing
  - Latency: Varies by distance
  - Pricing: cross-region transfer incurs additional costs

### EC2 Placement Groups

- Use to control how EC2 instances will be placed within AWS infrastructure.
- ❗ Works only for certain types of instances
- ❗ You can't merge placement group
- ❗ Placement group must have unique name within AWS account.
- ❗ You can't move existing EC2 to placement group -> you can create AMI and launch new.
- 💡 AWS recommends homogenous instances within placement groups e.g. using same instance types.
- ***Creation***
  - Console -> EC2 -> Placement Groups -> Create Placement Group -> Select a strategy
  - You can now launch new EC2 instances in the placement group.
- **Placement Group Strategies**
  - **Cluster**
    - Clusters instances into a low-latency group in a single Availability Zone
      - ❗ Cannot span multiple AZ
    - High performance, high risk
      - High network throughput, low-latency (10 Gbps bandwidth between instances)
      - Same rack & same AZ -> If the rack fails, all instances fails at the same time.
    - 💡 Use cases
      - Big Data job that needs to complete fast
      - Application that needs extremely low latency and high network throughput.
  - **Spread**
    - Spreads instances across underlying hardware (❗ max 7 instances per group per AZ)
      - Ensures all instances are located in different hardware.
    - Can span across multi AZ.
    - Minimizes failure risks.
  - **Partition**
    - Spreads instances across many different partitions (which rely on different sets of racks) within an AZ.
    - Partitions are isolated from each other from hardware failures in different racks but not VMs.
    - Can span across multi AZ.
    - Scales to 100s of EC2 instances per group.
    - ❗ Up to 7 partitions per AZ
    - EC2 instances get access to the partition information as metadata
    - 💡 Use cases: HDFS, HBase, Cassandra, Kafka

## EC2 Monitoring

- ❗📝 CloudWatch does not monitor memory usage as well as disk space utilization
  - 💡 Requires installation of CloudWatch agent or use custom scripts to send custom metrics.
- **Status Checks** checks physical machine (underlying hyper-visor) and returns pass & fail.
  - You can set-up **Status Checks** alarms.
  - Auto Scaling Group uses this check by default.
    - Can set to use Elastic Load Balancer's health check.

## EC2 Instance Metadata

- Allows AWS EC2 instances to learn about themselves without using an IAM Role / permission.
- The internal URLs are accessible only by EC2
- You can retrieve the IAM Role name from the metadata
  - ❗ You can not retrieve the IAM policy
- Metadata types:
  - **Meta data**
    - Info about the EC2 instance at `http://169.254.169.254/latest/meta-data`
    - Your EC2 instance has always credentials to the role that usually expires in 1 hour
      - Get IAM credentials: `http://169.254.169.254/latest/meta-data/iam/security-credentials/<iam-role-name>`
  - **User data**
    - Launch script of the EC2 instance `http://169.254.169.254/latest/user-data`
- **📝 Getting metadata**
  - Request to EC2 metadata service from your EC2 instance
    - `http://169.254.169.254/latest/meta-data`
    - It returns the properties (recursively)
    - Get data further e.g. `http://169.254.169.254/latest/meta-data/placement/availability-zone`
  - You can also use `Instance Metadata Query Tool` which is a simple bash script that does the curl.
