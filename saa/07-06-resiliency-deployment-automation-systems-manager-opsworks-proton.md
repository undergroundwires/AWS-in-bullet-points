# Deployment Automation

## CI/CD

### Continuous Integration

- Developers push the code to a code repository often
  - GitHub, BitBucket, ***CodeCommit*** (in AWS) etc...
- A testing / build server checks the code as soon as it's pushed
  - Jenkins CI, TeamCity, ***CodeBuild*** (in AWS), etc...
  - The developer gets feedback about the tests and checks that have passed / failed
- Helps
  - Find bugs early, fix bugs
  - Deliver faster as the code is tested
  - Deploy often
  - Happier developers, as they're unblocked

### Continuous Delivery

- Ensure that the software can be released reliably whenever needed.
- Ensures deployments happen often and are quick
- Automated deployment -> Shift away from e.g. "one release every 3 months" to 5 releases a day.
- ***CodeDeploy*** in AWS, Jenkins CD, Spinnaker, etc...

### Technology Stack

- **AWS CodePipeline** allows you to automate all steps of the deployment:

  | Step | Name | AWS | Others |
  | ---- | --- | ---- | ------ |
  | 1 | Code | CodeCommit | GitHub, GitLab, etc. |
  | 2 | Build & test | CodeBuild | Jenkins CI, TeamCity, etc. |
  | 3 | Deploy | Elastic Beanstalk, CodeDeploy | Octopus |
  | 4 | Provision | Elastic Beanstalk, CloudFormation | Terraform |

- **CodeDeploy**
  - Two deployment options:
    1. **In-place deployment**: EC2/On-Premises apps are stopped & updated & validated.
    2. **Blue/green deployment**: Using slots (deploy A -> switch to A -> stop B) with minimal downtime and rollback capabilities.
       - **EC2/On-Premises**: Must have one or more EC2 with tags or ASG
       - **Lambda**
         - **Canary**: Traffic is shifted in two increments, 90% to first version, 10% to second before 100% to second.
         - **Linear**: Traffic is shifted in equal increments with number of minutes e.g. 10% per minute.
         - **All-at-once**: All traffic is shifted at once.
       - **Amazon ECS**: Task set switches

## AWS Systems Manager

- Serverless & free service for managing AWS resources at scale
- Works by using SSM Agent software, preinstalled in AWS AMIs.
- No bastion hosts required - secure remote management without SSH/RDP
- Unified interface for managing AWS resources across accounts and regions

### AWS Systems Manager Run Command

- 📝 Execute scripts remotely on EC2 instances and on-premises servers
- Supports: Ansible, PowerShell DSC, shell scripts from S3
- No SSH/RDP needed - secure remote execution
- 💡 Use cases: Software installation, configuration changes, log collection

### AWS Systems Manager Patch Manager

- Automates patching for any updates on OS.
- Cross-platform: Linux/macOS/Windows (`yum`, `apt`, Windows Update).
- It can both:
  - Scan missing patches only
  - Scan and automatically install missing patches
- **Patch Baselines**
  - Rules for which patches should/shouldn't be installed
  - E.g. auto-approval, approved/rejected patch lists
- **Patch Groups**
  - Organize instances using tag key "Patch Group"
  - E.g. development, test, production environments
- **Maintenance Windows**
  - Schedule patching during low-impact times
  - Can configure scheduling, duration, targets, tasks

### AWS Systems Manager Session Manager

- Browser-based shell access to instances
- No SSH keys or bastion hosts required
- Audit logging: All session activity logged in CloudTrail
- IAM integration: Granular access control

### AWS Systems Manager Automation

- Automate complex, repetitive tasks
- Automation provided by pre-built runbooks (JSON/YAML)
- 💡 Use cases: OS patching, AMI updates, configuration enforcement
- Can run automations based on [EventBridge](./08-01-02-event-broadcasting-sns-eventbridge.md#amazon-eventbridge) events

## AWS OpsWorks

- AWS managed service for Chef & Puppet
- They work great with EC2 & On-Premises VMs
- Alternative to AWS SSM (Systems Manager)
- **OpsWorks stacks**
  - Groups applications into layers depending on Chef recipes
  - No need to provision chef server
  - Uses the embedded Chef solo client that is installed on EC2 instances on your behalf

## Chef & Puppet

- Help with managing server configuration as code
- Helps in having consistent deployments
  - Can automate: user accounts, cron, NTP, packages, services
- They leverage "Recipes" or "Manifests"
- Similar to SSM / Beanstalk / CloudFormation but they're open-source tools that work cross-cloud.

## AWS Proton

- Fully managed application deployment service
- Provides infrastructure as code templates for containerized and serverless applications
- Enables self-service deployment for developers
- **Templates**: Platform teams create standardized infrastructure templates
- **Services**: Developers deploy applications using approved templates
- **Environments**: Manages deployment targets (dev, staging, prod)
- 💡 Use cases: Microservices deployment, standardized infrastructure patterns, developer self-service
- Integrates with existing CI/CD tools and supports both ECS/EKS containers and Lambda
