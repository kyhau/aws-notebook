# Analytics

- [S3 -> Athena (+ Glue) -> QuickSight](#s3---athena--glue---quicksight)
- [DynamoDB -> Glue -> Athena -> QuickSight](#dynamodb---glue---athena---quicksight)
- [Athena](#athena)
- Data Pipeline
- [Elasticsearch Service](#elasticsearch-service)
- [EMR](EMR.md)
- Glue
- [Kinesis](Kinesis.md)
- QuickSight

---
## S3 -> Athena (+ Glue) -> QuickSight

- Athena supports compression formats Snappy (.snappy), Zlib (.bz2), GZIP (.gz).
- Use `SHOW tables` on the Query Editor can see all tables.

- Create Views for filtering and data transformation.
    - E.g.
    
    ```
    CREATE OR REPLACE VIEW simple_report AS 
    SELECT 
      user_id
    , team_id
    , date_diff('second', date_parse(start_time, '%Y-%m-%dT%H:%i:%s.%fZ'), date_parse(stop_time, '%Y-%m-%dT%H:%i:%s.%fZ')) AS duration
    FROM simple_table
    WHERE team_id = 'awesome';
    ```
    
---
## DynamoDB -> Glue -> Athena -> QuickSight

- Athena does not read from DynamoDB, but Glue can handle DynamoDB (last checked on 2018-12-20).

![Simplify Amazon DynamoDB data extraction and analysis by using AWS Glue and Amazon Athena](
  https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2018/09/12/simplify-amazon-dynamodb-glue-athena-1-2.gif
  "Simplify Amazon DynamoDB data extraction and analysis by using AWS Glue and Amazon Athena")
See [Simplify Amazon DynamoDB data extraction and analysis by using AWS Glue and Amazon Athena](
  https://aws.amazon.com/blogs/database/simplify-amazon-dynamodb-data-extraction-and-analysis-by-using-aws-glue-and-amazon-athena/).

---
## Athena

- Athena allows to use standard SQL to explore your AWS S3 data.
- Athena can handle complex analysis, including large joins, window functions, and arrays.
- Work with many data types: Apache Web Logs, JSON, CSV, TSV, etc
- But recommended to store in columnar formats in S3:  Apache Parquet, ORC (better performance).
- Can use EMR to convert data to columnar formats when outputting it.
- Support Geospatial Query
- Provide [built-in functions (Presto)](https://docs.aws.amazon.com/athena/latest/ug/presto-functions.html)
- Support [UDFs (user-defined functions) in Java](
  https://aws.amazon.com/about-aws/whats-new/2019/11/amazon-athena-adds-support-for-user-defined-functions-udf/)
  (Preview)
- Support running SQL queries across data stored in relational, non-relational, object, and custom data sources. 
  With federated querying, customers can submit a single SQL query that scans data from multiple sources running
  on-premises or hosted in the cloud. 
  ([Preview](
  https://aws.amazon.com/about-aws/whats-new/2019/11/amazon-athena-adds-support-for-running-sql-queries-across-relational-non-relational-object-custom-data-sources/))

More
- Partitioning
  - Athena allows you to partition your data on any column. 
  - Partitions allow you to limit the amount of data each query scans, leading to cost savings and faster performance. 
  - You can specify your partitioning scheme using the `PARTITIONED BY` clause in the `CREATE TABLE` statement. 

- Athena uses **SerDes** to interpret the data read from S3.
  - SerDe (Serializer/Deserializer), which are libraries that tell **Hive** how to interpret data formats.
    Hive DLL statements require you to specify a SerDe, so that the system knows how to interpret the data that
    you are pointing to. 

- Access Control
  - Athena uses IAM policies to restrict access to Athena operations.

- Encryption
  - You can encrypt
    - The results of all queries in S3, which Athena stores in a location known as the S3 staging directory. 
    - The data in the AWS Glue Data Catalog. 
  - Data at rest
    - SSE-S3   (Server-Side Encryption with Amazon S3-Managed Keys)
    - SSE-KMS  (Server-Side Encryption with AWS KMS-Managed Keys)
    - CSE-KMS  (Client-Side Encryption with AWS KMS-Managed Keys)
    - NOT SUPPORTED SSE-C (Server-Side Encryption with Customer-Provided Keys)
  - Data in transit
    - TLS (transport layer security) encrypts objects in-transit between Athena resources and between Athena and S3. 
    - Query results stream to JDBC clients as plain text and are encrypted using TLS.

- Limits
  - Default query timeout is 30 minutes.
  - Limit for S3 buckets per account: 100
  - Default number of partitions per table is 20,000.
  - You can submit up to 20 queries of the same type (DDL or DML) at a time (default).
    - DDL queries include CREATE TABLE, CREATE TABLE ADD PARTITION queries.
    - DML queries include SELECT and CREATE TABLE AS (CTAS) queries.
  - Default number of API calls per second:
    - GetQueryExecution, GetQueryResults: 25
    - Other API calls: 5

- Common Integrations
  - Athena is ideal for quick, ad-hoc querying and integrates with Amazon QuickSight for easy visualization.
  - AWS Glue Data Catalog can store table metadata for Athena.
  - AWS Glue can perform ETL on data that Athena wants to interact with.
  - Many other services can output or transfer data into S3 and Athena can then query it.
  - Querying Logs (CloudTrail, CloudFront, Load Balancer, VPC Flow, etc)

---
## Elasticsearch Service

- Elasticsearch has cluster node allocation across two Availability Zones in the same region, known as
  **zone awareness**.
- An Elasticsearch cluster with **3 master nodes is optimal**.
- Elasticsearch automated snapshots are kept for **14 days**.
- Indexing is adding **records** to Elasticsearch.
- Manual snapshots are stored in your S3 bucket and will incur relevant Amazon S3 usage charges.
- There is no additional charge for the automated daily S3 snapshots. The snapshots are stored for free in an Amazon
  Elasticsearch Service S3 bucket and will be made available for node recovery purposes. 
- You can use the Elasticsearch snapshot API to create additional manual snapshots in addition to the daily-automated
  snapshots created by Amazon Elasticsearch Service. 
  The manual snapshots are stored in your S3 bucket and will incur relevant Amazon S3 usage charges.
- Use Cases
   - [Loading Streaming Data into Amazon Elasticsearch Service](
       https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/es-aws-integrations.html).
   - Amazon Elasticsearch Service supports integration with Kibana.
   - The easiest way to put data into Elasticsearch is S3 Import.
   - You can use Lambda to send data to your Amazon ES domain from Amazon S3.
     New data that arrives in an S3 bucket triggers an event notification to Lambda, which then runs your custom code to
     perform the indexing. This method of streaming data is extremely flexible. You can index object metadata, or if the
     object is plaintext, parse and index some elements of the object body. 

