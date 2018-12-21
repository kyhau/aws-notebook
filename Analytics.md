# Analytics

- [S3 -> Athena (+ Glue) -> QuickSight](#s3---athena--glue---quicksight)
- [DynamoDB -> Glue -> Athena -> QuickSight](#dynamodb---glue---athena---quicksight)
- Athena
- Data Pipeline
- [Elasticsearch Service](#elasticsearch-service)
- [EMR](EMR.md)
- Glue
- [Kinesis](Kinesis.md)
- QuickSight


## S3 -> Athena (+ Glue) -> QuickSight

- Athena supports compression formats Snappy (.snappy), Zlib (.bz2), GZIP (.gz).
- Use `SHOW tables` on the Query Editor can see all tables.

- Create Views for filtering and data tranformation.
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
    

## DynamoDB -> Glue -> Athena -> QuickSight

- Athena does not read from DynamoDB, but Glue can handle DynamoDB (last checked on 2018-12-20).

![Simplify Amazon DynamoDB data extraction and analysis by using AWS Glue and Amazon Athena](
  https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2018/09/12/simplify-amazon-dynamodb-glue-athena-1-2.gif
  "Simplify Amazon DynamoDB data extraction and analysis by using AWS Glue and Amazon Athena")
See [Simplify Amazon DynamoDB data extraction and analysis by using AWS Glue and Amazon Athena](
  https://aws.amazon.com/blogs/database/simplify-amazon-dynamodb-data-extraction-and-analysis-by-using-aws-glue-and-amazon-athena/).



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

