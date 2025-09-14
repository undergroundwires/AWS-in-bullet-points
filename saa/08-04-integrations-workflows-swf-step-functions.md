# Serverless workflows

## AWS SWF - Simple Workflow Service

- 📝 Coordinate work amongst applications
- Code runs on EC2 (not serverless)
- 1 year max runtime
- Requires you to write application code
- **One time only completion** is key feature that ensures that a task is assigned only once and is never duplicated.
- 💡❗ **Step functions is recommended** to be used for new applications, except if you need:
  - *external signals* to intervene in the processes
  - *child processes* that return values to parent processes
  - *long running* up to 1 year
- **Actors**
  - **Workflow starters**: Any application that initiates workflow
  - **Deciders**: control flow activity tasks
  - **Activity workers**: carry out the activity tasks
- **Steps**: Activity, Decision, Human intervention
- **Tasks**
  - **Activity task**: Tell ***activity worker*** to execute its task.
  - **Decision task**: It tells the decider the state of the workflow execution.
  - **Lambda task**: Executes lambda
- 💡 Use case e.g. order fulfilment from web to warehouse to delivery.
- 🤗 AWS uses SWF to run their warehouses.

## AWS Step Functions

- Build serverless visual workflow to orchestrate distributed applications
- Helps adding resilient workflow automation
- 📝 Represent flow as a JSON or visual state machine
  - E.g. Start => Submit Job => Wait X seconds => Get Job Status => Job Complete? next : Go back to Wait X seconds -> End
- Features:
  - Running in sequence, parallel and conditionally.
  - built-in error handling
  - parameter passing
  - recommended security settings
  - state management
- The steps of your workflow can exist anywhere, including in AWS Lambda functions, on Amazon EC2, or on-premises.
- Integrated with & can coordinate many AWS services: *Amazon ECS*, *AWS Fargate*, *Amazon DynamoDB*, *Amazon SNS*, *[Amazon SQS](./08-01-02-integrations-queues-sqs.md)*, *AWS Batch*, *AWS Glue*, and *Amazon SageMaker*.
- ❗ Maximum execution time of 1 year
- Possibility to implement human approval feature
  - E.g. API gateway to send human approval accepted or not query.
- 💡 Use-cases:
  - Distributed applications
  - Automating processes
  - Orchestrating microservices
  - Creating data and ML pipelines
- Monitoring
  - Each executed branch can be monitored and have different steps.
  - Each step has `Success`, `Failed`, `Cancelled`, `In Progress` states.
- Can use **Callback patterns**
  - Pauses execution of the workflow until your application returns a token.
  - Supports callback patterns with Amazon ECS, Amazon SNS, Amazon SQS, AWS Fargate, and AWS Lambda.
  - Can also integrate with on-premises
