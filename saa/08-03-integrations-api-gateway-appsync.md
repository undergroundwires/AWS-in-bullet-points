# API Integrations

## API Gateway

- AWS API Management platform
- Supports stateful (WebSocket) and stateless (HTTP) APIs.
- Allows you to build completely serverless applications
  - 💡Common architecture: Client *<-- REST API -->* API Gateway *<-- Proxy requests -->* AWS Lambda *<-- CRUD -->* Amazon DynamoDB
- Supports:
  - API response caching
  - API versioning (v1, v2...)
  - Different environments (dev, test, prod...)
  - Transforming and validating requests and responses
  - **Auto-documentation**
    - Swagger / Open API import to quickly define APIs
    - Generate SDK and API specifications
  - **Monitoring**
    - Using [AWS X-Ray](./12-other-services.md#aws-x-ray) to trace messages as they travel through the APIs to backend services.
    - CloudTrail logs.
  - **Metering**
    - Define plans that meter and restrict third-party developer access to APIs.
    - Define a set of plans, configure throttling, and quota limits on a per API key basis.
    - Automatically meters traffic to your APIs and lets you extract utilization data for each API key.

### APIs

- You create different **APIs** (containers)
- Made of ***methods*** and ***resources***.
- **API endpoint types**
  - **Regional**: Default, deployed to the specified region.
  - **Edge-optimized**: API requests are routed to the nearest [CloudFront](./04-06-02-networking-edge-cloudfront.md) Point of Presence (POP)
  - **Private**: Exposed through interface [VPC endpoints](./04-02-networking-vpc.md#vpc-endpoints) for private API access inside VPC without Internet.
- **Methods**
  - Has HTTP verb (`ANY`, `DELETE`, `GET`, `HEAD`, `OPTIONS`, `PATCH`, `POST`, `PUT`)
  - Encapsulates frontend by method requests and method responses
    - During execution, you can modify data in each phase called modules:
      - Method request -> Integration Request -> Integration -> Integration Response -> Method response
    - In integration response you can set *Output Passthrough*
      - Passthrough: pass the client-supplied request payload through the integration request to the backend without transformation.
    - You can also use data transformations with  Velocity Template Language (VTL) templating scripts.
  - Allows ***Mock integrations*** without any back-end.
  - **Endpoint**
    - 💡 Public endpoints are always HTTPS!
    - Types:
      - **Regional**
        - For clients in the same region.
        - Reduces connection overhead
      - **Edge optimized**
        - For geographically distributed clients
        - API requests are routed to the nearest [CloudFront](./04-06-02-networking-edge-cloudfront.md) Point of Presence (POP).
      - **Private**: Can only be accessed from your VPC using an interface VPC endpoint
  - **Integration type**: Lambda, HTTP, MOCK, AWS Service, VPC Link (for internal endpoints)
    - **Proxy** integrations (can be HTTP or lambda) sends & passes entire payload without modifications (passthrough)
      - **Lambda** allows **Lambda proxy integration**: Requests will be proxied with request details available in the `event` of the handler function.
  - Can use default timeout (29 seconds)
- **Resources**
  - Resources groups **methods** or other nested resources.
  - Each resource has name (e.g. `Houses`), path (e.g. `/resources`) and option to enable CORS.

#### Stages

- APIs are deployed to stages (e.g. dev / test / prod)
- You get an URL for stages
- Stage options:
  - ***Settings***: Cache, Method throttling (with rate & burst), Client certificate
  - ***Logs/Tracing***: Enable CloudWatch logs / metrics, enable custom access logging, enable X-Ray tracing.
  - ***Stage variables*** that are key value pairs like environment variables
  - ***SDK Generation*** to Android, JavaScript, iOS, Java SDK, Ruby etc.
  - ***Export documentation*** as Swagger / OpenAPI 3 with optionally API Gateway or Postman extensions
  - ***Deployment history*** with ability to roll back
  - ***Documentation history*** with ability to roll back
  - ***Canary*** for testing with percentage of API traffic e.g. 95% of users will go to one API version, 5% go to another API version

#### Custom Domains

- ACM Region
  - 📝 **Regional endpoints**: Certificate must be in same region as API Gateway
  - **Edge-optimized endpoints**: Certificate must be in `us-east-1` (for CloudFront)
- Process:
  1. Create custom domain for API Gateway
     - Resource: `DomainName` (CDK), `aws_api_gateway_domain_name` (Terraform), `AWS::ApiGateway::DomainName` (CF)
  2. Create/import cert in ACM
     - ACM must be in same region for Regional endpoints, `us-east-1` for Edge-optimized endpoints.
     - Can request new cert from ACM OR import existing cert
     - Resource: `Certificate` (CDK), `aws_acm_certificate` (Terraform), `AWS::CertificateManager::Certificate` (CF)
  3. Reference cert ARN in the custom domain resource properties (step 1)
     - *Resource: `DomainName` (CDK), `aws_api_gateway_domain_name` (Terraform), `AWS::ApiGateway::DomainName` (CF)*
  4. Create base path mapping
     - Links custom domain to specific [API stage](#stages)
     - *Resource: `BasePathMapping` (CDK), `aws_api_gateway_base_path_mapping` (Terraform), `AWS::ApiGateway::BasePathMapping` (CF)*
  5. Configure Route 53 with alias record pointing to the API Gateway custom domain's target domain name
     - Target domain name example: `d-xxxxxxxxxx.execute-api.region.amazonaws.com`
     - *Resource: `ARecord` (CDK), `aws_route53_record` (Terraform), `AWS::Route53::RecordSet` (CF)*

### Throttling

- API gateway scales up to the default throttling limit of (10,000 requests per second)
  - It can burst past that up to 5,000 RPS.
- Throttling is used to protect back-end instances from traffic spikes
- Throttling can be configured at multiple levels including Global and Service Call.

### Security

- Track and control usage using API keys
  - Can also Lambda authorizers or usage plans to control access to your APIs.
- CORS
  - CORS is used by browsers against cross side scripting (XSS) attacks.
    - E.g. a malicious script in website requests to unknown origins.
  - ❗📝 You must enable CORS for API gateway to send right headers to `OPTIONS` requests.
- **Integrations with other VPC services**
  - Outside of VPC
    - Any AWS Service (AWS Lambda, Endpoints on EC2, Load Balancers)
    - External and publicly accessible HTTP endpoints
  - ❗ Inside of VPC: only AWS Lambda and EC2 endpoints in your VPC.
  
#### Authorization and Authentication of backend

- **IAM Roles / Users - AWS Signature Version 4 Signing Process (Sig v4)**
  - Allows you to sign all requests using an access key (derivation of access key ID + secret access key) in header
- **IAM policies with Lambda Authorizer** (formerly Custom Authorizers)
  - Uses AWS Lambda to validate the token in header being passed
    - Lambda must return an IAM policy for the user to gateway
    - IAM policy decides whether the gateway can call the back-end
  - Option to cache result of authentication
  - 💡 Helps to use OAuth / SAML / 3rd party type of authentication
- **[Amazon Cognito](./02-02-security-identity-federation-sts-saml-cognito-directory-service-identity-center.md#amazon-cognito) User Pools**
  - [Amazon Cognito](./02-02-security-identity-federation-sts-saml-cognito-directory-service-identity-center.md#amazon-cognito) fully manages user lifecycle without custom implementation
  - API gateway verifies identity automatically from [Amazon Cognito](./02-02-security-identity-federation-sts-saml-cognito-directory-service-identity-center.md#amazon-cognito)
  - ❗ [Amazon Cognito](./02-02-security-identity-federation-sts-saml-cognito-directory-service-identity-center.md#amazon-cognito) only helps with authentication, not authorization
  - You can enable MFA to help your users.
  - Flow
    1. Client authenticates & gets token from Amazon Cognito User Pools
    2. Client sends request to Gateway with a token
    3. Gateway validates the token
  - 💡 Good for Google & Facebook etc. authentication
- ***IAM vs Custom Authorizer vs Cognito User Pools***

  | Attribute | IAM | Custom authorizer | Cognito User Pool |
  | --------- | --- | ----------------- | ----------------- |
  | **Use case** | Great for users / roles already within your AWS account | Great for third party tokens | You manage your own user pool (can be backed by Facebook, Google, login etc..) |
  | **Authorization** | Handles authentication & authorization | Handles authentication & authorization | Handles only authentication, authorization must be implemented in the back-end |
  | **Implementation** | Leverages Sig v4 | Very flexible in terms of what IAM policy is returned | No need to write any custom code for authentication |
  | **Pricing** | Free (transfer charges) | Pay per Lambda invocation | Free (transfer charges) |

## AWS AppSync

- 📝 Serverless GraphQL Pub/Sub APIs
- Helps querying, updating or publishing data
- Store and sync data across mobile and web apps in real time
- Helps querying multiple databases, microservices, and APIs from singe GraphQL endpoint
- Client code can be generated automatically
- Integrations with DynamoDB / Lambda
- Real-time data updates to subscriptions through serverless WebSocket connections.
- 💡 Use-cases: Real-time chat apps, collaborative editing, live dashboards, mobile-first applications, offline-capable apps
- 🤗 Replaces Amazon Cognito Sync [DISCONTINUED 2024] with Graph QL and offline data synchronization capabilities.
