# AWS Notebook about Provisioning, Limitations, and Performance

**[Known Issues](KnownIssues.md)**

**Analytics**
- [Athena](Analytics.md)
- Data Pipeline
- [Elasticsearch Service](Analytics.md)
- EMR
  - [EMR vs. Redshift](EMR.md)
  - [HDFS vs. EMRFS](EMR.md)
  - [Essential Big Data Tools on EMR](BigDataTools.md)
- [Glue](Analytics.md)
- [Kinesis](Kinesis.md)
  - Kinesis Data Streams vs. Kinesis Data Firehose
  - KPL (Kinesis Producer Library) vs. AWS SDK
  - Auto Scaling in KCL
  - Data encryption in Kinesis Data Stream
  - Data encryption in Kinesis Data Firehose
  - Comparison of Streams
- [QuickSight](Analytics.md)

**Application Integration**
- Step Functions
- Amazon MQ
- SNS
- SQS
- SWF

**Compute**
- Lambda
- [ECS](ECS.md)
  - Auto Scaling policies
  - Migrating from Amazon Linux to Amazon Linux 2 for ECS
  - Subscribing to Amazon ECS-Optimized Amazon Linux AMI Update Notifications
  - Retrieving Amazon ECS-Optimized AMI Metadata
- ECR
- [Elastic Beanstalk](ElasticBeanstalk.md)

**Database**
- RDS
- [DynamoDB](DynamoDB.md)
- ElastiCache
- Redshift

**Machine Learning**
- [SageMaker](MachineLearning.md)
- [Amazon Machine Learning (AML)](MachineLearning.md)
- Polly
- Rekognition

**Management & Governance**
- [CloudWatch](LoggingAndMonitoring.md)
- Auto Scaling
- CloudFormation
- CloudTrail
- [Config](LoggingAndMonitoring.md)
- [Systems Manager (SSM)](LoggingAndMonitoring.md)
- [Trusted Advisor](LoggingAndMonitoring.md)
- [CloudWatch Buses](LoggingAndMonitoring.md)

**Migration & Transfer**
- [Database Migration Service (DMS)](MigrationAndTransfer.md)
- [Snowball and Snowmobile](MigrationAndTransfer.md)

**Mobile**
- Amplify

**Networking & Content Delivery**
- [Design edge security on AWS](DesignEdgeSecurity.md)
  - CloudFront - Global Content Delivery Network (CDN)
  - Forcing S3 Encryption
  - S3 Cross-Region Replication (CRR) Security
  - Protecting Web Applications
- [Design and implement a secure network infrastructure](SecureNetworkInfrastructure.md)
  - VPC Design and Security
  - Security Groups
  - VPC Peering
  - NACL (Network Access Control List)
  - VPC Endpoints (Gateway Endpoints and Interface Endpoints)
  - Serverless Security
  - NAT Gateways (Network Address Translation Gateways)
    - NAT Gateways vs. NAT Instances
  - Egress-Only Internet Gateways for IPv6
    - Egress-Only:  NAT Instance/Gateway for IPv4  vs.  Egress-Only Internet Gateway for IPv6
  - Bastion Hosts / Jump Boxes
- [Design and implement host-based security](HostBasedSecurity.md)
  - AWS Host / Hypervisor Security (disk/memory)
  - Host Proxy Servers  (aka. Instance Proxy Servers)
  - Host-based IDS/IPS (Intrusion Detection/Prevention System)
  - Systems Manager (SSM)
  - Packet Capture on EC2
- API Gateway
- Direct Connect
- Route 53

**Security, Identity, & Compliance**
- [Identity and Access Management](IdentityAndAccessManagement.md)
  - IAM
  - Cognito
  - STS, Web Identities, Identity Federation
  - Organisations
- [Data Protection](Encryption.md)
  - [Encryption solutions for data at rest and data in transit](Encryption.md)
  - [Key Management Service (KMS)](KMS.md)
  - CloudHSM & On-Premises HSM
- [GuardDuty](LoggingAndMonitoring.md)
- [Inspector](LoggingAndMonitoring.md)
- [Macie](LoggingAndMonitoring.md)
- Single Sign-On (SSO)
- [WAF & Shield](DesignEdgeSecurity.md)
- [Packet Capture Agent](LoggingAndMonitoring.md)
- [VPC Flow Logs](LoggingAndMonitoring.md)
- [DNS Logs](LoggingAndMonitoring.md)
- [S3 server access logging for bucket](LoggingAndMonitoring.md)

**Storage**
- [S3](S3.md)
- EFS

