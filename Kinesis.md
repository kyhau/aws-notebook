# Kinesis 

## Kinesis Benefits
- Real-time data streaming
- Ordered record delivery
- Replicated to 3 AZs for durability (Streams)
- Decoupled applications
- Replay data inside the data retention period
- Zero downtime scaling
- Pay as you go
- Parallel processing - multiple producers and consumers

## Kinesis Stream

## Kinesis Firehose

1. Firehose provides the following Lambda blueprints that you can use to create a Lambda function for data 
   transformation. 
    1. General Firehose Processing — Contains the data transformation and status model described in the previous
       section. Use this blueprint for any custom transformation logic. 
    2. Apache Log to JSON — Parses and converts Apache log lines to JSON objects, using predefined JSON field names. 
    3. Apache Log to CSV — Parses and converts Apache log lines to CSV format. 
    4. Syslog to JSON — Parses and converts Syslog lines to JSON objects, using predefined JSON field names. 
    5. Syslog to CSV — Parses and converts Syslog lines to CSV format.
