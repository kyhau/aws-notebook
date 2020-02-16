# RDS

- [RDS best practices](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_BestPractices.html)

- RDS - https://aws.amazon.com/rds/faqs/
    - up to 5 Read Replicas
    - supports cross-region read replicas 
    - can promote a read replica into a “standalone” DB Instance
    - second-tier read replica: supported by Amazon Aurora, Amazon RDS for MySQL and MariaDB (but not PostgreSQL)

- RDS now allows you to easily stop and start database instances. (See [Source](
https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_StopInstance.html))
    - While a database instance is stopped, RDS does not delete any of your automatic backups or transaction logs.
      This means you can do a point-in-time restore to any point within your specified automated backup retention
      window, even after an instance is started. 
    - Starting an instance restores it to the same configuration as it had when stopped, including its endpoint, DB
      parameter group, security group, and option group membership.
    - You can stop an instance for up to **7 days** at a time. After 7 days, it will be automatically started.
    - The stop/start feature is available for database instances running in a **Single-AZ deployment** which are not
      part of a Read Replica (both source and replica) configuration.

---
## Amazon Aurora with PostgreSQL

- Aurora Serverless with PostgreSQL
    - is not available in Sydney (last checked on 2020.01)
    - Potential issue of Aurora Serverless coldstart to APIG + Lambda
        - "since a sleeping Aurora DB cluster can take up to 25 seconds to be awakened... we hit the hard limit on Gateway API endpoint."
        - https://dev.to/dvddpl/how-to-deal-with-aurora-serverless-coldstarts-ml0
- See [RDS Aurora FAQs](https://aws.amazon.com/rds/aurora/faqs/)
- Up to 3X Higher Throughput than PostgreSQL
- Storage limits: 10 GB - 64 TB
- One area where Amazon Aurora improves upon PostgreSQL is with highly concurrent workloads. 
  In order to maximize your workload’s throughput on Amazon Aurora, we recommend building your applications to
  drive a large number of concurrent queries and transactions.
- Serverless Configuration - enables you to run your database in the cloud without managing any database instances
 (auto scaling).
- [Securing Amazon RDS and Aurora PostgreSQL database access with IAM authentication](
  https://aws.amazon.com/blogs/database/securing-amazon-rds-and-aurora-postgresql-database-access-with-iam-authentication/)
- Dive into new functionality for PostgreSQL 11
    - https://aws.amazon.com/blogs/database/diving-into-new-functionality-for-postgresql-11/
        - Enhancement on time-based partitioning
        - Parallelism
        - JIT compilation

---
## Database Migration Service (DMS)
- Sources
    - On-premises and Amazon EC2 instance databases
        - PostgreSQL version 9.4 and later (for versions 9.x), 10.x, and 11.x.
    - RDS instance databases
        - PostgreSQL
- Targets
    - On-premises and Amazon EC2 instance databases
    - RDS
        - PostgreSQL version 9.4 and later (for versions 9.x), 10.x, and 11.x.
        - Auroa with PostgreSQL compatibility
        - MySQL
        - Auroa with MySQL compatibility
        - Oracle
        - Microsoft SQL Server
        - MariaDB
    - Redshift
    - S3
    - DynamoDB
    - Amazon Elasticsearch Service 
    - Kinesis Data Streams
    - DocumentDB (with Mongo compatibility)

- **AWS Schema Conversion Tool (SCT)** 
    -  Supports a range of database and data warehouse conversions which are listed here. Note that SCT can be used to:
        - Copy a database schema from a source to a target
        - Convert a database or data warehouse schema
        - Analyze a database to determine the conversion complexity
        - Analyze a database to determine any possible restrictions to running on Amazon RDS
        - Analyze a database to determine if a license downgrade is possible
        - Convert embedded SQL code in an application
        - Migrate data warehouse data to Amazon Redshift
    - When a code fragment cannot be automatically converted to the target language, SCT will clearly document all locations that require manual input from the application developer.
- RDS ReadReplica vs. ElastiCache (Redis and Memcached)
