# Infrastructure as Code

## CloudFormation

- Code will be deployed and create / update / delete infrastructure
  - Can be JSON or YAML.
- Can be part of disaster recovery strategy.
- Declarative way of outlining AWS infrastructure for any resources (most of them are supported)
  - CloudFormation provisions in the *right order* with the *exact configuration* that you specify.
- 💡 Allows reusing the best practices around your company for configuration parameters.
- **Resource Terminology**
  - Logical Resource: Resource defined in CloudFormation template
  - Physical Resource: Resource created by creating a CloudFormation stack
- **Templates**
  - You can upload in S3 and then reference in CloudFormation
  - Update a template
    - Can't edit previous ones.
    - Have to re-upload a new version of the template to AWS.
  - **Building Blocks**
    - Cloud formation template consists of components and helpers:
      - Template components
        1. **Resources**: your AWS resources declared in the template (❗ mandatory)
        2. **Parameters**: the dynamic inputs for your template
           - Allows you to parameterize & prompt inputs while deploying.
        3. **Mappings**: the static variables for your template
        4. **Outputs**: References to what has been created
        5. **Conditionals**: List of conditions to perform resource creation
        6. **Metadata**
      - Template helpers
        1. References (***Logical IDs*** are used to reference resources within the template)
        2. Functions
- **Templates**
  - JSON or YAML text file that contains instructions for building out AWS environment
- **Change sets**
  - Before making changes to your resources, you can generate a change set, which is a summary of your proposed changes, e.g. resource A will be deleted.
- **Stacks**
  - The entire environment described by the template and created, updated, and deleted as a single unit
  - Related resources in a template.
    - When you update stack, it updates the resources
  - Can rollback on failure after timeout in minutes.
  - Termination protected with stack from being accidentally deleted.
  - **Monitoring**
    - Fires events such as on resource creation
    - Can have monitoring time (CloudFormation monitors resources during X minutes after creation)
    - Can set up CloudWatch alarms if something fails.
  - Can have IAM role or policy to deny/allow access.
  - Can use sample template or create one in Designer.
  - Identified by name
  - Can have input parameters that'll be filled from portal.
  - Reusable output information
    - It's output information (key, value, description) can be imported by other stacks by using an unique export/import name.
  - **Stack Sets** lets you create stacks in AWS accounts across regions by using a single AWS CloudFormation template
  - Operations
    - **Deleting a stack** -> deletes every artifact created by CloudFormation
    - **Deploying templates/stacks**
      1. by uploading YAML files directly or referencing to Amazon S3 URL
      2. Use a sample template
      3. Create template in Designer
         - Helps you to see resources & relations
         - Might be good for PowerPoints
      - ❗ Redeploying a stack deletes old instances.
    - **Changing a stack**
      - Done using **Stack change sets**
      - Warns you about resources that'll be deleted, modified and added.
      - **Drift**: difference between the expected configuration values and actual deployed.
        - Allows you to better manage your CloudFormation stacks and ensure consistency in your resource configurations
- Can deploy Elastic Beanstalk that'll then deploy:
  - E.g. EC2, ALB, ASG, [RDS](./06-02-01-data-databases-rds.md)
- **IAM Conditions for CloudFormation**
  - `cloudformation:TemplateURL`: Ensure template in TemplateURL is used for e.g. update/delete.
  - `cloudformation:ResourceTypes`: Specify which types of resources can be created or updated.
  - `cloudformation:StackPolicyURL`
    - **Stack policies** prevent stack resources from being unintentionally updated or deleted during stack updates
    - You define which actions (t.ex. update) are allowed on which resources in a JSON file.

### Benefits

- **Control**
  - No resources are manually created / configured.
  - Code can be version controlled e.g. using git.
  - Changes are reviewed through code
  - Cost control:
    - Each stack has ID: follow-up costs.
    - Estimate costs of a stack.
    - You can automate deletion of templates at 5 PM and recreation at 8 AM safely.
- **More productivity**:
  - Destroy and re-create an infrastructure on the cloud on the fly.
  - Automated generation of diagram for your templates!
  - Declarative programming (no need to figure out ordering and orchestration)
- Better separation of concerns:
  - Many stacks for many apps, and many layers.
  - E.g. VPC stacks, network stacks, app stacks

## AWS CDK (Cloud Development Kit)

- Infrastructure as Code using programming languages instead of JSON/YAML to manage AWS infrastructure
- Supports: Python, JavaScript/TypeScript, Java, C#/.NET, Go
- Higher-level abstractions called "constructs" that encapsulate AWS resources with sensible defaults
- Compiles to CloudFormation
- **Benefits over raw CloudFormation:**
  - Programming language features: loops, conditions, functions, classes
  - IDE support: auto-completion, refactoring, debugging  
  - Reusable components and libraries
  - Unit testing of infrastructure code

## AWS SAM (AWS Serverless Application Model)

- Open source tool used to package, test, and deploy serverless applications
- YAML based infrastructure as code.
- Serverless applications = E.g. Lambda, DynamoDB, API Gateway...
- Helps run serverless services locally
