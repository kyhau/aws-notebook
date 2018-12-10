# Kinesis 


Table of Contents
- [Kinesis Data Streams vs. Kinesis Data Firehose](#kinesis-data-streams-vs-kinesis-data-firehose)
- [KPL (Kinesis Producer Library) vs. AWS SDK](#kpl-kinesis-producer-library-vs-aws-sdk)
- [Comparison of Streams](#comparison-of-streams)


## Kinesis Data Streams vs. Kinesis Data Firehose

- Kinesis Data Stream is the customizable approach.

  1. Best for users who want to build custom applications to process or analyze streaming data for specialized needs.
     Data can be sent to real-time dashboards, used to generate alerts, implement dynamic pricing and advertising
     strategies, and more.
  2. (Not auto-scale) Scaling is possible from a single megabyte up to terabytes per hour.
     The appropriate number of shards must be provisioned for your stream to handle the volume of data you expect to
     process.
     Resharding takes less than a minute but cannot be done in parallel.  That is, if you initially have 1 shard and
     you need 10 shards, you will need to wait ~10 minutes.
  3. Low latency: < 1 second
  4. Retention Period: Defaults to 24 hours, up to 168 hours (7 days)
  5. Consumers: KCL applications, AWS SDK+Lambda

- Kinesis Firehose the simple solution.

  1. Ideal for users who want to load streaming data from a web app, mobile app, or telemetry system directly into AWS
     storage systems to process streaming data. There is no need to write applications or manage resources.
  2. (Auto-scale):  Scaling is possible to gigabytes per second and allows for batching, encrypting, and compressing of
     data.
  3. “Latency”:  1 min - 15 mins (Firehose buffers incoming streaming data to a certain size or for a certain period of
     time before delivering it to destinations.  Buffer interval: 60-900s (1-15m)).
  4. Consumers:  S3, Redshift, Elasticsearch, Splunk; and can then be copied to other services for further processing
     and analysis.


## KPL (Kinesis Producer Library) vs. AWS SDK

1. KPL is best for high rate producers.
   AWS SDK is best for low rate producers (mobile apps, IoT devices, web clients (that are low rate producers)).
2. KPL is best for those that need record (batching, aggregation+collection).
   SDK can create and delete streams, but aggregation is managed by you.
3. For operational reporting:
   KPL implements an asynchronous send function, so it can be used for the informational messages.
   SDK PutRecords is a synchronous send function, so it must be used for the critical events.
4. KPL can incur an additional processing delay of up to RecordMaxBufferedTime within the library (user-configurable).
   Larger values of RecordMaxBufferedTime results in higher packing efficiencies and better performance.
   Applications that cannot tolerate this additional delay may need to use the AWS SDK directly.


## Comparison of Streams

- Kinesis Data Streams: Ordered data, Replay capability, Record-by-record, 1MB per record.
- Apache Spark Streaming: Only-once delivery, Out-of-order data, Micro-batching
- Apache Flink: Only-once delivery, Out-of-order data, Replay capability, Batch processing
- Apache Kafka: Ordered data, Batching processing, > 1MB per record

