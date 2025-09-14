# EC2 Provisioning

## EC2 Instance Types

- Type describes how powerful the machine is (vCPUs, Memory, Network performance etc.)
- 📝**Main types**

  | Type | Mnemonic | Category | Description | Use-cases |
  | ---- | -------- | -------- | ----------- | --------- |
  | M | (M)ain / (M)edium | General purpose | Balance of compute, memory, network resources | General, mid-size databases, web apps |
  | C | (C)ompute |  Compute optimized | Advanced CPUs | Modeling, analytics, databases |
  | H | (H)DD | Storage optimized | Local HDD storage | Map reduce |
  | R | (R)am | Memory Optimized | More ram per $ | In-memory caching |
  | X | (X)treme | Memory Optimized | Terabytes of RAM and SSD | In-memory databases |
  | I | (I)ops | IO optimized | Local SSD storage, High IOPS | NoSQL databases |
  | G | (G)PU | GPU Graphics | GPUs with video encoders | 3D rendering |
  | P | (P)ictures | GPU Compute | GPUs with tensor cores | Machine learning |
  | F | (F)ield | Accelerated Computing | Field Programmable Gate Array, custom hardware accelerations | Genomics, financial analytics |
  | T | (T)iny / (T)urbo | Cost optimized | Burstable, Shared CPUs, lowest cost | Web servers |

  - Remember all: PHD McGift Rx
- Numbers change over time and indicate generation e.g. `t2.`, `t3.`.
- **Burstable instances**
  - Good performance for a burst / short while e.g. spike and your CPU goes to 100%
  - Burst credits
    - Bursting consumes burst credits and low CPU gets you credits.
    - You get more credits when your CPU is low.
- T2 / T3 unlimited provides *unlimited burst* but you pay extra if you go over your credit balance.
- You can also have *Bare Metal* instances to eliminate virtualization overhead.
- X1e is part of Memory Optimized  instance family and recommended for databases.

## EC2 Instance Launch Types

### On-Demand Instances

- Start whenever you want & stop without any long time commitment.
- Pricing
  - Pay for what you use (billing per second, after the first minute)
  - Has highest cost but no upfront payment
- 💡 Recommended for short-term and uninterrupted workload, where you can't predict how the application will behave.
  - E.g. when doing auto-scaling

### Reserved instances

- **Standard Reserved instances**
  - Reserve an instance for 1 to 3 years
  - Enables you to modify Availability Zone, scope, network platform, and instance size (within the same instance type)
    - Up to 75% discount compared to On-demand
    - Payment options are "no upfront" (pay monthly), "partial upfront", "all upfront" (cheaper).
  - You can sell your unused instances on the AWS Reserved Instance Marketplace.
  - 💡 Recommended when e.g. when running a database over a year
  - 💡 Reserve minimum capacity you'll use in year e.g. 1 EC2 instance in each AZ in multi-AZ architecture.
  - 💡 Can be sold in Reserved Instance Marketplace
- **Convertible Reserved Instances**
  - Convertible means you have option to change:
    - Instance family, instance type, platform, scope, and tenancy
    - Only if the creation of new instance is equal or greater value
  - Pricing: Up to 54% discount
  - 💡 Workloads are likely to change
- 🤗 [DISCONTINUED 2024] **Scheduled Reserved Instances**
  - Launch within time window you reserve, e.g. every week during one hour only.

### Spot instances

- Cheapest (up to 90% discount compared to on demand) but risk of losing instance
  - Spot instance is charged per second.
  - If AWS shuts down your instance in first hour you don't pay, but you have to pay if you terminate.
- You bid a maximum price and get the instance as long as it's under the price
  - You lose the instance if you get outbid
    - Instances are reclaimed with a 2 minute notification warning when the spot price goes above your bid.
    - ***Persistence request*** allows instances to come back when the price drops to the right pricing.
  - Price varies based on offer and demand at times
- **📝Spot instance shut down priority**: The one who had longer time (e.g. 50 min) will be shut down faster than the shorter time (e.g. 5 min)
- 💡 For short & fault tolerant workloads e.g. batch jobs, Big Data analysis.
- In the event of an interruption, you can set instances to be **terminated** or **hibernated**.
- You may get insufficient capacity error
  - Stop all instances & start again to end up in a new hardware.
- ❗ 20 On-Demand & 20 Reserved & 20 Spot instances per region (soft limit)
- 🤗 [DISCONTINUED 2022] **Spot Block**
  - Historically allowed you to request spot instances for 1 to 6 hours at a time

## EC2 Capacity Management

### On-Demand Capacity Reservations

- Also known as *ODCRs*.
- 📝 Reserve compute capacity in a specific Availability Zone for any duration.
- **Key Features**:
  - No long-term commitment required (unlike [Reserved Instances](#reserved-instances))
  - Can be created at any time and choose when it starts
    - Reserve capacity up to 120 days in advance
  - Can modify or cancel at any time
  - Priced exactly the same as equivalent On-Demand instance usage
  - If fully utilized, you only pay for instance usage
  - If partially utilized, you pay for instance usage + unused portion of reservation
  - Can Split, move, and modify
    - Divide existing reservations, transfer capacity between reservations
  - Can control instances to provision exclusively on ODCRs or ODCRs before on-demand
- **Use Cases**:
  - 💡 Business-critical workloads requiring guaranteed capacity
  - 💡 Scaling events (product launches, Black Friday, etc.)
  - 💡 Regulatory requirements and disaster recovery
  - 💡 When you need capacity assurance but not cost savings
- **Combination Benefits**:
  - Can be combined with [Savings Plans](./01-02-01-aws-management-economics-pricing-pricing-calculator-data-transfer-savings-plans.md#savings-plans) or [Regional Reserved Instances](#reserved-instances) to receive discounts
  - Provides capacity guarantee + cost optimization
- **Configuration Options**:
  - Open: Automatically matches running instances with matching attributes
  - Targeted: Must specifically target the reservation to use it

### Capacity Blocks for ML

- Specialized reservations for GPU instances for defined periods (future dates)
- **Use Cases**:
  - 💡 ML model training and fine-tuning
  - 💡 ML experiments and prototypes requiring GPU instances
  - 💡 Handling temporary surges in inference demand
- Ensures uninterrupted access to GPU resources for ML workloads

### EC2 Fleet

- Collection of [On-Demand instances](#on-demand-instances), [Spot Instances](#spot-instances), and [Reserved Instances](#reserved-instances) managed automatically
- Maintains target capacity across multiple instance types, sizes, and AZs
- Automatically replaces interrupted Spot Instances
- Allocation strategies:
  - Diversified: Spreads instances across all Spot pools
  - Lowest-price: Launches instances from lowest priced pools
  - Capacity-optimized: Launches instances from pools with optimal capacity
- Can specify target capacity mix (e.g., 70% Spot, 30% On-Demand)
- Supports auto scaling integration
- 🤗 Successor of **Spot Fleet**
  - Primarily designed for Spot instances, use [EC2 fleet](#ec2-fleet) for new implementations.

## EC2 Tenancy

- Each instance that you launch into a VPC has a tenancy attribute
  - **Default**: Your instance runs on shared hardware.
  - **[Dedicated](#dedicated-instances)**: Your instance runs on single-tenant hardware.
  - **[Host](#dedicated-hosts)**: Your instance runs on an isolated server with configurations that you can control.

### Changing Tenancy

- You only switch between dedicated instances i.e. from *host* to *dedicated* and from *dedicated* to *host*.
- ❗ [Cannot change tenancy](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/dedicated-instance.html#dedicated-howitworks) after launching instances when
  - From *default* to *dedicated* or *host*.
  - From *dedicated* or *host* to *default*.
- A VPC tenancy can be changed to any but it only affects new instances.

### Dedicated Instances

- Instances running on hardware that's dedicated to you
  - You don't control your hardware.
  - May share hardware with other instances in same account
- No control over instance placement (can move hardware after Stop / Start)

### Dedicated Hosts

- Physical dedicated EC2 server for your use
- Full control of EC2 instance placement
- Visibility into the underlying sockets / physical cores of the hardware
  - You have direct access to the CPU
- Allocated for your account for a 3 year period reservation
- More expensive
- You can allow instance auto-placement where untargeted EC2s will be launched in dedicated host(s).
- Useful for
  - Complicated licenses such as BYOL - Bring Your Own License, or you pay based on CPU cores etc.
    - ❗ A Dedicated Host is required if you'd like to use your existing Windows Server licenses.
  - Regulatory & compliance needs.

### Dedicated Instances vs Dedicated hosts

| Characteristics | Dedicated Instances | Dedicated Hosts |
| --------------- | ------------------- | --------------- |
| Enables the use of dedicated physical servers | X | X |
| Per instance billing (subject to a $2 per region fee) | X | |
| Per host billing | | X |
| Visibility of sockets, cores, host ID | | X |
| Affinity between a host and instance | | X |
| Targeted & automatic instance placement | | X |
| Add capacity using an allocation request | | X |

## EC2 User Data

- Enables to bootstrap instances using EC2 data script.
  - *bootstrapping* means launching commands when a machine starts
- Script is only run once at the instance first start
- Enables automating boot tasks such as *installing updates*, *installing software*, *downloading common files from internet* ....
- Runs with the root user (`sudo` rights)
- 📝Get user data -> Request to `http://169.254.169.254/latest/user-data` from your EC2 instance
- **Flow**
  - Create EC2 -> Configure Instance -> Advanced Details -> User data -> Paste commands
  - 💡 In first line add `#!/bin/bash` -> tells Unix what program to use to run it
- For faster deployment without bootstrapping use **Golden AMI**s:
  1. Create a golden AMI with the static installation components already setup
  2. Use EC2 user data to customize the dynamic installation parts at boot time.
     - 🤗 Elastic Beanstalk is a good example for mixing golden AMIs and EC2 user data.
