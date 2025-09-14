# Containers

## Docker

- Allows application to work the same way anywhere where docker is installed
- Containers are isolated from each other
- Control how much memory / CPU is allocated to your container
- Ability to restrict network rules
- More efficient than Virtual machines
- Scale containers up and down very quickly

## ECS

- Fully managed container orchestration service
- Deploy, manage, scale containerized applications.
- Built-in AWS/third-party integrations and best-practices
- E.g. [Elastic Container Registry (ECR)](#elastic-container-registry) and [Docker](#docker)

### ECS Anywhere

- Extends ECS to run containerized workloads on anywhere while using the same AWS ECS control plane.
  - E.g. on-premises servers, VMs, or edge infrastructure
- Uses EXTERNAL [launch type](#ecs-task-launch-types)

### ECS Use Cases

- Run microservices
  - Ability to run multiple docker containers on the same machine
  - Easy service discovery features to enhance communication
  - Direct integration with Application Load Balancers
  - Auto scaling capability
- Run batch processing / scheduled tasks
  - Schedule ECS containers to run on On-demand / Reserved / Spot instances
- Migrate applications to the cloud
  - Dockerize legacy applications running on premise
  - Move Docker containers to run on ECS

### ECS Tasks

- An application can have many tasks.
- A task contains one or more (max 10) containers.
- Applications are defined using **task definitions**.
  - You can apply a single IAM role to a task definition.
  - Other task definitions include: *Which docker images*, *CPU/memory*, *whether containers are linked*, *networking mode*, *ports that should be mapped/opened*, *whether task should continue if container finished or fails*, *commands to execute when container is started*, *environment variables to pass*, *data volumes*.
  - Can have two [launch types](#ecs-task-launch-types)
- IAM security and roles at the ECS task level

#### ECS Tasks Scheduling

- **Service scheduler** for long-running stateless tasks and applications.
- **Manually running tasks** ideal for batch jobs that perform work and stop.
- **Scheduled tasks (cron)**: run based on cron expression or based on [CloudWatch Events](./03-01-monitoring-cloudwatch-grafana.md#cloudwatch-events) rule.
- **Custom schedulers**: Your own schedulers or third party e.g. Blox.

#### ECS Task Launch Types

- **EC2** (original): Runs ECS on user-provisioned EC2 instances
  - Requires installation of **ECS Container Agent**
  - Both for Windows + Linux
  - Included in the Amazon ECS optimized AMI
  - Can also be installed on any EC2 instance that supports the ECS specification
- 📝 **FARGATE**: Runs ECS tasks on AWS provisioned compute (serverless)
- **EXTERNAL**:
  - Enables ECS Anywhere
  - Runs ECS tasks on customer-managed infrastructure
  - Requires SSM agent and ECS agent on on-premises/edge infrastructure
  - Allows hybrid cloud deployments with same ECS control plane

### ECS Clusters

- Logical grouping of container instances that you can place tasks on.
- Set of EC2 instances
  - Clusters can contain tasks using BOTH the Fargate and EC2 launch type
  - For tasks using the EC2 launch type, clusters can contain multiple different container instance types.
- Each container instance may only be part of one cluster at a time
- Clusters are Region-specific.
- **ECS service**: Application definitions that manage tasks running on ECS cluster

### ECS Security

- **ECS IAM roles**: Roles assigned to [tasks](#ecs-tasks) to interact with AWS.
- **Security groups**: Can be associated to container instances.

### ALB Integration

- 📝 Application Load Balancer (ALB) has a direct integration feature with ECS called "port mapping"
  - Allows you to run multiple instances of the same application on the same EC2 machine
  - Dynamic port mapping is not available in classic load balancer and allows the same port to be mapped to many.
    - E.g. containers on port 6789, 9586, 3748 can be exposed on port 80 /443 by application balancer.
- Use cases:
  - Increased resiliency even if running on one EC2 instance
  - Maximize utilization of CPU / cores
  - Ability to perform rolling upgrades without impacting application uptime

## ECS Setup and Configuration

- Run an EC2 instance, install the ECS agent with ECS config file
  - Or use an ECS-ready Linux AMI (still need to modify config file)
    - Config file is at `/etc/ecs/ecs.config`

      ```bash
        ECS_CLUSTER=MyCluster #Assign EC2 instance to an ECS cluster
        ECS_ENGINE_AUTH_DATA={..} #to pull images from private registries
        ECS_AVAILABLE_LOGGING_DRIVERS=[..] #CloudWatch container logging
        ECS_ENABLE_TASK_IAM_ROLE=true #Enable IAM roles for ECS tasks
      ```

## Elastic Container Registry

- Fully managed service to store, manage and deploy your containers on AWS
- Supports Cross-Region Replication for multi-region
- Integrated IAM & ECS
- Sent over HTTPS (encryption in flight) and encrypted at rest
- ECS pulls from ECR with IAM role, while CodeBuild pushes images (CI/CD) to ECR.

## Elastic Kubernetes Service

- Also known as *EKS*
- Running Kubernetes on AWS (running on EC2)
- Pros:
  - Service discovery
  - Big open source ecosystem and good competence
  - Internal networking & network overlay
  - No vendor lock-in
- Cons: more complexity & costs
- Can use envelope encryption of Kubernets secrets using [KMS](./02-03-security-encryption-ssm-kms-secrets-manager-certificate-manager.md#aws-key-management-service)
  - Secrets are stored encrypted in EKS cluster's `etcd` key-value store

### EKS Distro

- Open-source Kubernetes distribution with the same components and versions used by Amazon EKS.
- Can be deployed anywhere.
- Provides
  - reproducible builds,
  - extended security patching support,
  - and follows same Kubernetes release cycle as EKS

### EKS Anywhere

- Container management software for running Kubernetes clusters on-premises and at edge locations.
- Built on EKS Distro.
- Customer-managed solution that can run in air-gapped environments.
  - Supports VMware vSphere, bare metal, Nutanix, Apache CloudStack, and AWS Snow.

## Amazon Managed Service for Prometheus

- Managed monitoring service for containerized applications
- Compatible with open-source Prometheus
- Integrates with [Amazon EKS](#elastic-kubernetes-service), [Amazon ECS](#ecs), and AWS Distro for OpenTelemetry
- Serverless, automatically scales
- Use cases: Container and microservices monitoring
