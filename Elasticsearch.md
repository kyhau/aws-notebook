# Amazon Elasticsearch Service (ES)

Table of Contents
- [Limits](#limits)
- [Concepts](#concepts)
- [Monitoring with CloudWatch](#cloudwatch)
- [Benefits](#benefits)
- [Use cases](#use-cases)
- [Pricing](#pricing)


## Limits
- Elasticsearch automated snapshots are kept for 14 days.

## Concepts
- Elasticsearch has cluster node allocation across two Availability Zones in the same region, known as zone awareness.
- Indexing is adding **records** to ElasticSearch.
- An Elasticsearch cluster with 3 master nodes is optimal.

## CloudWatch
https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/es-managedomains.html#es-managedomains-cloudwatchmetrics

## Benefits
Amazon ES offers the following benefits of a managed service: 
- Cluster scaling options
- Data durability 
- Enhanced security
- Node monitoring
- Replication for high availability 
- Self-healing clusters 

## Use Cases
https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/es-aws-integrations.html

- Amazon Elasticsearch Service supports integration with Kibana.
- The easiest way to put data into Elasticsearch is S3 Import.
- You can use Lambda to send data to your Amazon ES domain from Amazon S3.
  New data that arrives in an S3 bucket triggers an event notification to Lambda, which then runs your custom code to
  perform the indexing. This method of streaming data is extremely flexible. You can index object metadata, or if the
  object is plaintext, parse and index some elements of the object body. 

## Pricing
- Manual snapshots are stored in your S3 bucket and will incur relevant Amazon S3 usage charges.
- There is no additional charge for the automated daily S3 snapshots. The snapshots are stored for free in an Amazon
  Elasticsearch Service S3 bucket and will be made available for node recovery purposes. 
- You can use the Elasticsearch snapshot API to create additional manual snapshots in addition to the daily-automated
  snapshots created by Amazon Elasticsearch Service. 
  The manual snapshots are stored in your S3 bucket and will incur relevant Amazon S3 usage charges.
