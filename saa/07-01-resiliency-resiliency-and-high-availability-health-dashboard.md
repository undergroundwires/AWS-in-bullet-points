# High Availability and Scalability

## Resiliency Concepts

- [resiliency-concepts.drawio.png](./img/high-availability/resiliency-concepts.drawio.png)
- **Resiliency**
  - The system's ability to withstand and recover from failures while maintaining core functions
  - Resiliency cover =
    - Scalable and loosely coupled architectures
    - Highly available and/or fault tolerant architectures
  - E.g. a cloud architecture that automatically handles server crashes, regional outages, and traffic spikes without service disruption.
- [**High-availability (HA)**](#high-availability)
  - Reduce downtime when things break
  - E.g. load balancers failover to backup servers if one crashes.
- **Fault-tolerance (FT)**
  - Keep working despite failures
  - E.g. redundant power supplies prevent server downtime during power failures.
- **Disaster Recovery (DR)**
  - Rebuild after a catastrophe
  - E.g. database replicated to another region in case of earthquake.
- **[Scalability](./07-03-resiliency-scalability-auto-scaling-asg.md)**
  - Expand to handle increased load
  - E.g. auto-scales servers during Black Friday traffic (preventing overload-induced outages).
- **Redundancy**
  - Duplicate critical components to eliminate single points of failure
  - E.g., RAID disk arrays, dual power supplies, or multi-homed network connections.

## High Availability

- Focuses on minimizing downtime during routine failures.  
  - I.e. maximizing systems online time.
- Goes hand in hand with horizontal scaling
- Running application in at least 2 data centers (= Availability Zones)
  - Goal is to survive a data center loss
- Can be:
  - Passive e.g. [RDS Multi AZ](./06-02-01-data-databases-rds.md#multi-az)
  - Active e.g. for horizontal scaling
- For EC2, you can use *Auto Scaling Group* with multi AZ enabled and *Load Balancer* with multi AZ enabled.
- 💡 Design for failure as it can happen

### Common architectures

- **Single region HA architecture**
  - ![Single region AZ failure tolerant architecture](img/high-availability/single-region-architecture.png)
  - 3 tier application with 3 tiered security

    | Tier | Role | Example service | Security Group |
    | ---- | ---- | --------------- | -------------- |
    | 1 | Client | Application Load Balancer | Allow HTTP + HTTPs |
    | 2 | Application  | Application servers (EC2) | Allow HTTP from Client |
    | 3 | Database | Amazon RDS | Allow e.g. MySQL from application SG |

  - In a single VPC
    - At least 2 AZ and and in each AZ:
      - A public subnet and in each public subnet:
        - Deploy NAT gateway
        - Load balance public subnet in AZs through application load balancer
      - A private subnet and in each private subnet:
        - Deploy application server in same auto-scaling group
        - Create route table that routes internet (0.0.0.0/0) to the NAT gateway of a public subnet in same AZ
        - Configure auto-scaling group to put servers in public Load balancer
        - Deploy [RDS](./06-02-01-data-databases-rds.md) instance in one, and [RDS multi-AZ](./06-02-01-data-databases-rds.md#multi-az) standby in other one
  - 📝 E.g. if you have minimum 6 instances running and can tolerate failure of 1 AZ failure:
    - Deploy 3 AZ with 3 instances in each AZ so you always have 6 instances running.
- **Multi region HA architecture**
  - E.g. using S3
    - ![Multi region HA architecture using S3](img/high-availability/multi-region-s3-architecture.png)
  - E.g. using [RDS](./06-02-01-data-databases-rds.md)
    - ![Multi region HA architecture using RDS](img/high-availability/multi-region-rds-architecture.png)

## AWS Health Dashboard

- Provides real-time visibility into AWS service health and your account's resource health
- Two views:
  - **Service Health Dashboard**
    - Overall AWS service status across all regions
  - **Personal Health Dashboard**
    - Personalized view of AWS service health affecting your resources
    - 💡 Use to get alerts specific to your AWS resources rather than general service status
- Features:
  - 📝 **Proactive notifications** about planned maintenance and service issues
  - **Historical information** about service disruptions
  - **Remediation guidance** for affected resources
  - Integration with CloudWatch Events for automated responses
- Can trigger Lambda functions or SNS notifications when health events occur
