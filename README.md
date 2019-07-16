# AWS Notebook

Topics
- [AWS Certified Big Data - Specialty](#aws-certified-big-data---specialty)
- [AWS Certified Security - Specialty](#aws-certified-security---specialty)
- [AWS Certified Solutions Architect - Professional](#aws-certified-solutions-architect---professional)
- [Multi-region active-active architecture] (TODO)
- [Disable TLS 1.0 and use TLS 1.1 or higher](ConfigureTLS.md)
- [Known Issues](KnownIssues.md)

---
## AWS Certified Big Data - Specialty

- **Collection**
  - IoT
  - Greengrass
  - [Kinesis](Kinesis.md)
    - Kinesis Data Streams vs. Kinesis Data Firehose
    - KPL (Kinesis Producer Library) vs. AWS SDK
    - Auto Scaling in KCL
    - Data encryption in Kinesis Data Stream
    - Data encryption in Kinesis Data Firehose
    - Comparison of Streams
  - [Snowball and Snowmobile](MigrationAndTransfer.md)
  - [Database Migration Service (DMS)](MigrationAndTransfer.md)
- **Storage**
  - [S3](S3.md)
  - [DynamoDB](DynamoDB.md)
  - RDS
- **Processing**
  - EMR
    - [EMR vs. Redshift](EMR.md)
    - [HDFS vs. EMRFS](EMR.md)
    - [Essential Big Data Tools on EMR](BigDataTools.md)
  - [Amazon Machine Learning (AML)](MachineLearning.md)
  - [SageMaker](MachineLearning.md)
  - [Lambda](Lambda.md)
  - Data Pipeline
  - [Glue](Analytics.md)
  - [Big Data Tools](BigDataTools.md)
- **Analysis**
  - [Elasticsearch Service](Analytics.md)
  - [Athena](Analytics.md)
  - [Kinesis Analytics](Kinesis.md)
  - Redshift
- **Visualisation**
  - [QuickSight](Analytics.md)
- **Data Security**
  - IAM for Big Data
  - Database Encryption
  - Auditing with CloudTrail

---
## AWS Certified Security - Specialty

- **Incident Response and Automated Alerting**
- **[Logging and Monitoring](LoggingAndMonitoring.md)**
  - CloudTrail
  - CloudWatch
  - Config
  - S3 server access logging for bucket
  - Inspector
  - Systems Manager (SSM)
  - Trusted Advisor
  - Packet Capture Agent
  - VPC Flow Logs
  - DNS Logs
  - CloudWatch Buses
  - GuardDuty
  - Macie
- **Infrastructure Security**
  - Design edge security on AWS
    - [CloudFront - Global Content Delivery Network (CDN)](DesignEdgeSecurity.md)
    - Forcing S3 Encryption
    - S3 Cross-Region Replication (CRR Security)
    - Protecting Web Applications
      - [AWS WAF](DesignEdgeSecurity.md)
      - [AWS Shield](DesignEdgeSecurity.md)
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
- **[Identity and Access Management](IdentityAndAccessManagement.md)**
  - IAM Policies
  - Permission Boundaries
  - Policy Evaluation
  - KMS Key Policies
  - Organisations and Service Control Policies
  - Cross-account access to S3 bucket and objects
  - Cross-account access to Lambda
  - Cognito
  - Identity Federation (Web Identity, SAML 2.0, Custom ID Broker)
  - Systems Manager Parameters Store
- **Data Protection**
  - Key management with [Key Management Service (KMS)](KMS.md)
  - Key management with [CloudHSM & On-Premises HSM](CloudHSM.md)
  - [Encryption solutions for data at rest and data in transit](Encryption.md)
    - Data in Transit: ACM (AWS Certificate Manager)
    - Data at Rest: SSE-C (Server Side Encryption - Customer Key)
    - Data at Rest: KMS

---
## AWS Certified Solutions Architect - Professional

- **[DynamoDB](DynamoDB.md)**
- **[ECS (Elastic Container Service)](ECS.md)**
  - Auto Scaling policies
  - Migrating from Amazon Linux to Amazon Linux 2 for ECS
  - Subscribing to Amazon ECS-Optimized Amazon Linux AMI Update Notifications
  - Retrieving Amazon ECS-Optimized AMI Metadata
- **[ECS vs. EKS (Elastic Kubernetes Service)](EKS_v_ECS.md)**
- **[Elastic Beanstalk](ElasticBeanstalk.md)**
- **[ELB](ELB.md)**
  - Application Load Balancer
  - Network Load Balancer
  - Classic Load Balancer
- **[Lambda](Lambda.md)**
- **[S3](S3.md)**

---
## Multi-region active-active architecture

- TODO
