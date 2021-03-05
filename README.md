# AWS Notebook

Topics
- [Advanced Networking](#advanced-networking)
- [Big Data](#big-data)
- [Database](#database)
- [Security](#security)
- [Solutions Architect](#solutions-architect)
- [Architecture resources and notes](Architecture.md)
- [Security resources and notes](Security.md)
- [Known Issues](KnownIssues.md)

---
## Advanced Networking
- [Key notes](Networking.md)


## Big Data

- **Collection**
  - IoT
  - Greengrass
  - [Kinesis](Kinesis.md)
  - [Snowball and Snowmobile](MigrationAndTransfer.md)
  - [Database Migration Service (DMS)](MigrationAndTransfer.md)
- **Storage**
  - [S3](S3.md)
  - [DynamoDB](DynamoDB.md)
  - [RDS](RDS.md)
- **Processing**
  - [EMR](EMR.md)
  - [EMR vs. Redshift](EMR.md)
  - [HDFS vs. EMRFS](EMR.md)
  - [Essential Big Data Tools on EMR](BigDataTools.md)
  - [Amazon Machine Learning (AML)](MachineLearning.md)
  - [SageMaker](MachineLearning.md)
  - [Lambda](Lambda.md)
  - [Glue](Analytics.md)
  - [Big Data Tools](BigDataTools.md)
- **Analysis**
  - [Elasticsearch Service](Analytics.md)
  - [Athena](Analytics.md)
  - [Kinesis Analytics](Kinesis.md)
  - [Redshift](Database.md)
- **Visualisation**
  - [QuickSight](Analytics.md)

---
## Database
- [Key notes](Databases.md)
- [Aurora, RDS, DMS, SCT](RDS.md)
- [DynamoDB](DynamoDB.md)
- [ElastiCache for Memcached](ElastiCache.md)
- [ElastiCache for Redis](ElastiCache.md)

---
## Security

- **Incident Response and Automated Alerting**
- **[Logging and Monitoring](LoggingAndMonitoring.md)**
  - [VPC Flow Logs](VpcFlowLogs.md)
- **Infrastructure Security**
  - [Design edge security on AWS](DesignEdgeSecurity.md)
  - [Design and implement a secure network infrastructure](SecureNetworkInfrastructure.md)
  - [Design and implement host-based security](HostBasedSecurity.md)
- **[Identity and Access Management](IdentityAndAccessManagement.md)**
- **Data Protection**
  - Key management with [Key Management Service (KMS)](KMS.md)
  - Key management with [CloudHSM & On-Premises HSM](CloudHSM.md)
  - [Encryption solutions for data at rest and data in transit](Encryption.md)
  - [Secrets Manager](SecretsManager.md)

---
## Solutions Architect

- **[DynamoDB](DynamoDB.md)**
- **[EC2](EC2.md)**
- **[ECS (Elastic Container Service)](ECS.md)**
  - Auto Scaling policies
  - Migrating from Amazon Linux to Amazon Linux 2 for ECS
  - Subscribing to Amazon ECS-Optimized Amazon Linux AMI Update Notifications
  - Retrieving Amazon ECS-Optimized AMI Metadata
- **[ECS vs. EKS (Elastic Kubernetes Service)](EKS_v_ECS.md)**
- **[Elastic Beanstalk](ElasticBeanstalk.md)**
- **[ElastiCache](ElastiCache.md)**
  - Redis
  - Memcached
- **[ELB](ELB.md)**
  - Application Load Balancer
  - Network Load Balancer
  - Classic Load Balancer
- EventBridge
- **[Lambda](Lambda.md)**
- **[S3](S3.md)**

---
## Multi-region active-active architecture

- TODO

---
## Machine Learning

- [SageMaker]((MachineLearning.md))

---
## AR & VR

- Amazon Sumerian

---
## Blockchain

- Amazon Managed Blockchain
- Amazon QLDB

---
## GameTech

- GameLift Realtime Servers - you can create multiplayer game servers that can be customized with just a few lines
  of JavaScript and then scale to millions of players for a fraction of a penny per player per month.
- Lumberyard - a game engine with no royalties or seat fees, frictionless integration with Twitch and AWS.
