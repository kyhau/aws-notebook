# Elastic MapReduce (EMR)

Table of Contents

- [EMR vs. Redshift](#emr-vs-redshift)
- [HDFS vs. EMRFS](#hdfs-vs-emrfs)
- [Essential Big Data Tools on EMR](BigDataTools.md)


## EMR vs. Redshift

Redshift is far more cost effective than EMR for analytics that can be performed on a traditional database.

**Redshift**

1. Redshift is ideal for large volumes of structured data that you want to persist and query using standard SQL and
   your existing BI tools.
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


## HDFS vs. EMRFS

**HDFS**

1. HDFS distributes the data it stores across multiple instances in the cluster.
2. HDFS storage is lost when the cluster is terminated.
3. Most often used for intermediate results.
4. The filesystems that can run HDFS: EBS, Local attached storage, S3 on EMRFS

**EMRFS**

1. Implementation of HDFS used fo reading/writing regular files from EMR directly to S3.
2. For storing persistent data in S3 for use with Hadoop while also providing features like S3 server-side encryption,
   read-after-write consistency, and list consistency.
3. **Consistent View**
   1. Allows EMR clusters to check for list and read-after-write consistency for S3 objects written by or synced with EMRFS.
   2. Number of retries: 5 (default);  if an inconsistency is detected, EMRFS tries to call S3 this number of times.
   3. Retry period (in seconds): 10 (default)

