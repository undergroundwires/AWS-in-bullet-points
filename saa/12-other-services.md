# Other services

## Amazon Simple Email Service (SES)

- Bulk & secure transactional email-sending service.
- SNS vs SES

  | Attribute | SNS | SES |
  | --------- | --- | --- |
  | Target | For push notifications to AWS users | Third parties e.g. customers |
  | Flow | topic => e-mail opt-in confirmation => sends e-mails | Send rich e-mails through API calls. |
  | Custom headers, MIME types | Not supported | Supported |

- 💡 Use-cases: Email marketing campaigns, transactional emails (order confirmations, password resets), automated notifications, newsletter delivery

## AWS IoT Core

- AWS service that helps you manage IoT devices & harvest data from them to e.g. Kinesis.
- Bi-directional communication between Internet-connected devices and the AWS Cloud.
  - Devices can be e.g. sensors, actuators, embedded micro-controllers, or smart appliances.
- 💡 Use-cases:
  - Smart home automation
  - Industrial IoT monitoring
  - Fleet management
  - Environmental sensors
  - Predictive maintenance

## AWS Amplify

- Framework to build full-scale web and mobile apps
- Cross-platform backend for iOS/Android/Flutter/web or React Native app.
- Provides real-time and offline functionality.
- Helps connecting frontend UI to cloud backend services.
- Features/integrations include authentication, Lambda, analytics, offline data sync.
- Supports deployment to CDN
- 💡 Use-cases: Mobile app backends, web app development, rapid prototyping, startup MVPs, full-stack development

## AWS X-Ray

- Analyze and debug production and distributed applications
- Traces user requests as they travel through applications, helps to debug.
- Filters visual data across payloads, functions, traces, eservices, APIs
- Helps determine bottlenecks, improve performance.
- 💡 Use-cases: Performance monitoring, debugging microservices, request tracing, latency analysis, error tracking

## AWS Elemental MediaConvert

- File-based video transcoding service with broadcast-grade features
- Transcodes video/audio content for delivery to televisions, connected devices, and the web
- Features: Up to 8K resolution, 120 FPS, HEVC/AV1 codecs, Dolby Audio/Atmos, DRM, captions, graphic overlays
- Quality-defined variable bitrate (QVBR) with AI optimization
- Pay-per-minute transcoding with reserved pricing available
- 🤗 Successor to *Amazon Elastic Transcoder* **[DISCONTINUED Nov 13, 2025]**
- 💡 Use-cases: VOD content creation, broadcast delivery, streaming platforms, professional media workflows, fleet management dashcam processing

## AWS WorkSpaces

- Managed, secure cloud desktop service
- Eliminates management of on-premise VDI (Virtual Desktop Infrastructure)
- On Demand, pay per usage
- Secure, Encrypted, Network Isolation
- Integrated with Microsoft Active Directory
- 💡 Use-cases: Remote work, virtual desktops, BYOD policies, contractor access, secure development environments

## Amazon AppStream

- Fully managed application streaming service
- 💡 Use-cases:
  - Software delivery
  - Remote application access
  - Legacy app modernization
  - BYOD environments
  - Secure app streaming

## AWS Device Farm

- App testing service that lets you test and interact with your Android, iOS, and web apps on many devices at once, or reproduce issues on a device in real time
- 💡 Use-cases:
  - Mobile app testing
  - Cross-platform validation
  - Automated testing
  - Bug reproduction
  - App store preparation

## Amazon Pinpoint

- Helps marketers/developers deliver customer communications across channels/segments/campaigns.
- Send customized SMS, push notification, email and custom channel messages to the right person at the right time.

## AWS Data Exchange

- Portfolio of third-party data sets
- Helps find data files, data tables and data APIs
- Integrates withdata and analytics and machine learning services.

## Amazon WorkDocs

- Fully managed  storage, document collaboration, and file sharing service.
- Amazon Workdocs is like Google Docs, Amazon S3 is like Google Drive

## VMware Cloud on AWS

- Fully managed VMware vSphere-based cloud service running natively on AWS infrastructure
- Extends on-premises VMware environments to AWS with same tools, policies, and processes
- Provides bare-metal AWS infrastructure optimized for VMware workloads
- 💡 Use-cases: Hybrid cloud operations, disaster recovery, data center migration, capacity expansion, application modernization

## AWS Serverless Application Repository

- Repository of pre-built serverless applications and components that you can deploy and customize
- Applications are packaged using SAM (Serverless Application Model) templates
- Benefits:
  - Code reuse: Discover and deploy ready-made serverless apps
  - Time-to-market: Skip building common patterns from scratch  
  - Best practices: Applications follow AWS recommended architectures
- Common application types:
  - API backends and microservices
  - Data processing pipelines  
  - Chatbots and Alexa skills
  - Image/video processing functions
- Integrates with AWS SAM CLI for local development and testing
