# Encryption solution for data at rest and data in transit

Table of Contents
- [Athena](#athena)
- [CloudFront](#cloudfront)
- [CloudTrail](#cloudtrail)
- [DynamoDB](#dynamodb)
- [EBS](#ebs)
- [EMR](#emr)
- [Kinesis Data Stream](#kinesis-data-stream)
- [Kinesis Data Firehose](#kinesis-data-firehose)
- [RDS](#rds)
- [Redshift](#redshift)
- [S3](#s3)

See also [KMS](KMS.md).

---

## Athena
- You can encrypt
  - the results of all queries in S3, which Athena stores in a location known as the S3 staging directory, and
  - the data in the AWS Glue Data Catalog. 
- Data at rest
  ```
  - SSE-S3   (Server-Side Encryption with Amazon S3-Managed Keys)
  - SSE-KMS  (Server-Side Encryption with AWS KMS-Managed Keys)
  - CSE-KMS  (Client-Side Encryption with AWS KMS-Managed Keys)
  - NOT SUPPORTED: SSE-C (Server-Side Encryption with Customer-Provided Keys)
  ```
- Data in transit
  ```
  - TLS (transport layer security) encrypts objects in-transit between Athena resources, and between Athena and S3. 
  - Query results stream to JDBC clients as plain text and are encrypted using TLS.
  ```

## CloudFront
- **Field-level encryption** allows you to securely upload user-submitted sensitive information to your web servers. 
  - The sensitive information provided by your clients is encrypted at the edge closer to the user and remains encrypted
    throughout your entire application stack, ensuring that only applications that need the data - and have the
    credentials to decrypt it - are able to do so. 
  - You can encrypt up to 10 data fields in a request. (You can't encrypt all of the data in a request with field-level
    encryption; you must specify individual fields to encrypt.)

## CloudTrail
- Data at rest
  - CloudTrail logs are encrypted in SSE-S3 by default; can be changed to SSE-KMS.

## DynamoDB
- Data at rest (KMS)
  - **How does it work**
    ```
      DynamoDB                             AWS KMS
          |     (1) Generate data key         |
          |---------------------------------> |  CMK
          |                                   |
      Table key <-----------------------------|
          |
          |  (2) encrypt
          v
    Data encryption key(s)
          |
          |  (3) encrypt
          v
        Table
    ```
    1. DynamoDB uses the CMK for the table to generate and encrypt a unique **data key** for the table, known as the
       **table key**. 
       - The **table key** persists for the lifetime of the encrypted table.
       - The **table key** is used as a key encryption key.
       - The **table key** is managed by DynamoDB and stored with the table in an encrypted form.
    2. DynamoDB uses this **table key** to protect **data encryption keys** that are used to encrypt the table data.
    3. DynamoDB generates a unique **data encryption key** for each underlying structure in a table, but multiple
       table items might be protected by the same **data encryption key**.
  - **Encryption types**
    - All DynamoDB tables are encrypted. There is no option to enable or disable encryption for new or existing tables.
    - When creating a new table, you can choose one of the following customer master keys (CMK) to encrypt your table:
      - **AWS owned CMK** – Default encryption type. The key is owned by DynamoDB in each region (no additional charge).
      - **AWS managed CMK** – The key is stored in your account and is managed by AWS KMS (AWS KMS charges apply).
    - Encryption at rest does not support **customer managed CMKs**. 
    - You can switch between the AWS owned CMK and AWS managed CMK at any given time. 
      DynamoDB continues to deliver the same single-digit millisecond latency that you have come to expect, and all
      DynamoDB queries work seamlessly on your encrypted data.
  - **Protection includes**
    - Primary key and local and global secondary indexes.
    - DynamoDB streams, global tables, and backups whenever these objects are saved to durable media.
  - **Table Key Caching**
    - To avoid calling AWS KMS for every DynamoDB operation, DynamoDB caches the **plaintext table keys** for each
      connection in memory.
    - Table keys are cached for up to **12 hours in plaintext** by DynamoDB, but a request is sent to KMS after
      **5 minutes** of table key inactivity to check for permission changes.
    - If DynamoDB gets a request for the cached table key after **5 minutes of inactivity**, it sends a new request
      to AWS KMS to decrypt the table key. This call will **capture any changes made** to the access policies of the
      CMK in AWS KMS or AWS IAM since the last request to decrypt the table key.
  - See also [AWS Developer Guide](https://docs.aws.amazon.com/kms/latest/developerguide/services-dynamodb.html).

## EBS
- Data at rest (KMS)
  - EBS Volume is encrypted using a **DataKey** generated from a **CMK**.
  - **Encrypted data key** is stored with the volume.
  - Used by the **Hypervisor** to decrypt upon attaching to the EC2 instance.
  - IO, Snapshots and Persisted data are encrypted.

## EMR
- Data at rest
  - For EMRFS on S3
    - Server-side: SSE-S3, SSE-KMS, or
    - Client-side: CSE-KMS, CSE-Custom
  - For cluster nodes (EC2 instance volumes; except boot volumes)
    - HDFS LUKS (Linux Unified Key System)
- Data in transit
  - For EMRFS traffic between S3 and cluster nodes (enabled automatically):
    - TLS encryption (certification using PEM).
  - For data in transit between nodes in a cluster: 
    - SSL (Secure Sockets Layer) for MapReduce
    - SASL (Simple Authentication and Security Layer) for Spark shuffle encryption
- Data being spilled to disk or cached during a shuffle phase:
  - Spark shuffle encryption or
  - LUKS encryption

## Kinesis Data Stream
- Data are already encrypted.  Kinesis Data Streams automatically encrypts data before it is at rest by using an KMS
  CMK you specify. 
- Data is encrypted before it is written to the Kinesis stream storage layer, and decrypted after it is retrieved from
  storage. As a result, your data is encrypted at rest within the Kinesis Data Streams service.

## Kinesis Data Firehose
- Kinesis Data Firehose allows you to encrypt your data after it is delivered to your S3 bucket. 
- While creating your delivery stream, you can choose to encrypt your data with an KMS key that you own.

## RDS
- Data at rest (KMS)
  - RDS utilizes EBS for its encryption. RDS instances are managed versions of EC2 instances, configured to act as a
    managed DB cluster. 
  - In a similar way to EC2, encrypted volumes attached to RDS are handled by the host, with persistent data, snapshots
    and IO encrypted and decrypted using KMS.

## Redshift
- Database Encryption
  - None
  - KMS
  - HSM (Hardware security model)

## S3
- Data at rest
  ```
  - SSE-S3  (Server-Side Encryption with Amazon S3-Managed Keys)
  - SSE-KMS (Server-Side Encryption with AWS KMS-Managed Keys)
  - SSE-C   (Server-Side Encryption with Customer-Provided Keys)
  - CSE-CMS (Client-Side Encryption with KMS-Managed Customer Master Key)
  - CSE-C   (Client-Side Encryption with Client-Side Master Key)
  ```  
- Data in transit
  ```
  - SSL
  ```
- Objects are encrypted and the settings are defined at an **object level**.
- You can now set **S3 Default Encryption** on a **bucket level**. If set, then any objects put into a bucket without
  encryption headers are encrypted using the bucket-level default settings.
  - **Bucket policies** can be used to **DENY** attempts to put objects into a bucket with individual encryption methods.
- Each object is encrypted with a unique key employing strong encryption. As an additional safeguard, it encrypts the
  key itself with a master key that it regularly rotates.
- When using KMS
  - The **DataKey** is generated from a **CMK** (**service default**, or a **custom CMK**).
  - Every object in a bucket is encrypted by S3 using a **DataKey** provided by KMS.
  - **CipherText DataKey** is stored with the object as metadata.
  - When decryption is needed, it is passed to KMS (`kms:Decrypt`), and used by S3 to decrypt the object.
- Use Case:
  - You work on a large scale online media website and provide access to videos which are hosted on S3 via pre-signed
    URLs. The media is stored as objects in an S3 bucket in an encrypted state. Your large user base is generating
    slightly over 10,000 media views per second and your application logs have started to show `ThrottlingException`
    errors. 
  - Potential solution: Lodge a support request to increase KMS limits.
