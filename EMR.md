# Elastic MapReduce (EMR)

Table of Contents
- [EMR bs. Redshift](#emr-vs-redshift)


## EMR vs. Redshift

Redshift is far more cost effective than EMR for analytics that can be performed on a traditional database.

**Redshift**

1. Redshift is ideal for large volumes of structured data that you want to persist and query using standard SQL and your existing BI tools.
2. Petabyte-scale
3. Redshift is based of PostgreSQL (kind of like a database).
4. Max of 128 nodes of the large sizes.
5. Sources: S3, DynamoDB, EMR, Data Pipeline, SSH enabled host on EC2 or on-premise
6. Encryption: KMS or HSM or none

**EMR**

1. EMR is ideal for processing and transforming unstructured or semi-structured data to bring in to Redshift.
2. EMR is also a much better option for data sets that are relatively transitory, not stored for long-term use.
3. Terabyte-scale, Petabyte-scale
4. Use EMR if you need map-reduce algorithms.
5. Store and process massive quantity of data (100s TBs to PBs) on several servers (100s to 1000s).
6. Sources: AWS services / on-premises -> S3 -> EMR leader node; streams
7. Encryption
   1. Data at rest
      1. For EMRFS on S3: SSE-S3, SSE-KMS, or CSE-KMS, CSE-Custom
      2. For cluster nodes (EC2): HDFS LUKS (Linux Unified Key System)
   2. Data in transit (in-flight)
      1. For EMRFS traffic between S3 and cluster nodes (enabled automatically): TLS encryption (certification using PEM).
      2. For data in transit between nodes in a cluster: 
         1. SSL (Secure Sockets Layer) for MapReduce
         2. SASL (Simple Authentication and Security Layer) for Spark shuffle encryption
