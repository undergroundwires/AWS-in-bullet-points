# AWS in bullet points

[![Quality checks status](https://github.com/undergroundwires/AWS-in-bullet-points/workflows/Quality%20checks/badge.svg)](https://github.com/undergroundwires/AWS-in-bullet-points/actions)
[![GitHub sponsors](https://undergroundwires.dev/img/badges/donate/flat.svg)](https://undergroundwires.dev/donate?project=AWS-in-bullet-points)

- This repo contains study notes for different AWS exams.
- The notes are comprehensive and written with goal of covering all exam areas.
  - I passed the exam with 956/1000 score with these notes.
- Good luck & enjoy studying! ☕

## Symbols

- There are some symbols used throughout the documentation:

  | Symbol | Description |
  |:------:|-------------|
  | 💡 | Best practice or an important solution |
  | ❗ | An important limitation, challenge or an exception |
  | 📝 | Common exam area |
  | 🤗 | Fact / trivia (most likely unrelated to the exam) |

## Content

### AWS Certified Solutions Architect – Associate

1. [Introduction](./saa/1.%20Introduction.md)
   1. [Interacting with services - AWS CLI & SDK](./saa/1.1.%20Interacting%20with%20services%20-%20AWS%20CLI%20&%20SDK.md)
2. Security
   1. [IAM & Organizations](./saa/2.1.%20Security%20-%20IAM%20&%20Organizations.md)
   2. [Identity federation (STS, SAML, Cognito, SSO, Directory Service)](./saa/2.2.%20Security%20-%20Identity%20federation%20(STS,%20SAML,%20Cognito,%20SSO,%20Directory%20Service).md)
   3. [Encryption in AWS (SSM, KMS)](./saa/2.3.%20Security%20-%20Encryption%20in%20AWS%20(SSM,%20KMS).md)
3. [Monitoring (CloudWatch, CloudTrail, AWS Config)](./saa/3.%20Monitoring%20-%20CloudWatch,%20CloudTrail,%20AWS%20Config.md)
4. Compute
   1. [EC2](./saa/4.1.%20Compute%20-%20EC2.md)
   2. [Elastic Beanstalk](./saa/4.2.%20Compute%20-%20Elastic%20Beanstalk.md)
   3. [Containers - ECS & EKS](./saa/4.3.%20Compute%20-%20Containers%20-%20ECS%20&%20EKS.md)
   4. [Lambda](./saa/4.4.%20Compute%20-%20Lambda.md)
5. Storage
   1. [EC2 Storage (EBS & EFS - Instance Store)](./saa/5.1.%20Storage%20-%20EC2%20Storage%20(EBS%20&%20EFS%20&%20Instance%20Store).md)
   2. [S3](./saa/5.2.%20Storage%20-%20S3.md)
   3. [S3 - Security](./saa/5.3.%20Storage%20-%20S3%20-%20Security.md)
   4. [S3 - Integrated Services (Snowball, Athena, Storage Gateway)](./saa/5.4.%20Storage%20-%20S3%20Integrated%20Services%20(Snowball,%20Athena,%20Storage%20Gateway).md)
6. Networking
   1. [IP Addresses (Elastic IP)](./saa/6.1%20Networking%20-%20IP%20Addresses%20(Elastic%20IP).md)
   2. [Route 53](./saa/6.2.%20Networking%20-%20Route%2053.md)
   3. [VPC](./saa/6.3.%20Networking%20-%20VPC.md)
   4. [Subnets](./saa/6.4.%20Networking%20-%20VPC%20-%20Subnets.md)
   5. [Hybrid connections](./saa/6.5.%20Networking%20-%20Hybrid%20connections.md)
7. [CDN - CloudFront](./saa/7.%20CDN%20-%20CloudFront.md)
8. High Availability
   1. [High Availability and Scalability](./saa/8.1.%20High%20Availability%20-%20High%20Availability%20and%20Scalability.md)
   2. [Load Balancers (ELB, CLB, NLB and ALB)](./ssa/../saa/8.2.%20High%20Availability%20-%20Load%20Balancers%20(ELB,%20CLB,%20NLB%20and%20ALB).md)
   3. [Scalability (ASG)](./saa/8.3.%20High%20Availability%20-%20Scalability%20(ASG).md)
   4. [Disaster Recovery](./saa/8.4.%20High%20Availability%20-%20Disaster%20Recovery.md)
   5. [Infrastructure as Code](./saa/8.5.%20High%20Availability%20-%20Infrastructure%20as%20Code.md)
9. [Databases - Overview](./saa/9.%20Databases%20-%20Overview.md)
   1. [RDS](./saa/9.1.%20Databases%20-%20RDS.md)
   2. [Aurora](./saa/9.2.%20Databases%20-%20Aurora.md)
   3. [DynamoDB](./saa/9.3.%20Databases%20-%20DynamoDB.md)
   4. [Redshift](./saa/9.4.%20Databases%20-%20Redshift.md)
   5. [ElastiCache (Redis & Memcached)](./saa/9.5.%20Databases%20-%20ElastiCache%20(Redis%20&%20Memcached).md)
   6. [Other databases (DocumentDB, Athena, ElasticSearch, Neptune)](./saa/9.6.%20Databases%20-%20Other%20databases%20(DocumentDB,%20Athena,%20ElasticSearch,%20Neptune).md)
10. Integrations
    1. [Queues - Overview](./saa/10.1.%20Integrations%20-%20Queues%20-%20Overview.md)
       1. [SQS](./saa/10.1.1.%20Integrations%20-%20Queues%20-%20SQS.md)
       2. [SNS](./saa/10.1.2.%20Integrations%20-%20Queues%20-%20SNS.md)
       3. [Kinesis](./saa/10.1.3.%20Integrations%20-%20Queues%20-%20Kinesis.md)
       4. [Amazon MQ](./saa/10.1.4.%20Integrations%20-%20Queues%20-%20Amazon%20MQ.md)
    2. [API Gateway](./saa/10.2.%20Integrations%20-%20API%20Gateway.md)
    3. [Workflows (SWF & Step Functions)](./saa/10.3.%20Integrations%20-%20Workflows%20-%20SWF%20&%20Step%20Functions.md)
11. [Big Data & Analytics](./saa/11.%20Big%20Data%20&%20Data%20Analytics.md)
12. [Other services](./saa/12.%20Other%20services.md)
13. [Well Architected Framework](./saa/13.%20Well%20Architected%20Framework.md)
14. [Exam Readiness](./saa/14.%20Exam%20Readiness.md)

## Support

- ⭐️ Simplest way to say thanks is just to it a star 🤩
- ❤️ To show more support:
  - ☕️ [buy me a coffee](https://buymeacoffee.com/undergroundwire)
  - 👏🏿 [sponsor me](https://github.com/sponsors/undergroundwires)
  - 🎈 [donate using another way](https://undergroundwires.dev/donate)
- ✨ Contributions of any kind are welcome!
