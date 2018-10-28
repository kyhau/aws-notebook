# DynamoDB

Table of Contents
- [Limits](#limits)
- [Partitions and Provisioning Reads and Writes](#partitions-and-provisioning-reads-and-writes)
- [Indexing - Secondary, LSI, GSI](#indexing)
- [Reading data - Primary Key Lookup, Query, Scan, Eventually Consistent vs. Strongly Consistent](#reading-data)
- [Writing data](#writing-data)


## Limits

- Max item size: 400KB
- Nested attributes up to 32 levels deep
- Max num of tables: 256 (default)
- US East:
    - Per table: 40K RCUs and 40K WCUs (default)
    - Per account: 80K RCUs and 80K WCUs (default)
- Other regions:
    - Per table: 10K RCUs and 10K WCUs (default)
    - Per account: 20K RCUs and 20K WCUs (default)
- LSIs per table: 5
- GSI per table: 5
- Max num of projected attributes per index: 20


## Partitions and Provisioning Reads and Writes

- DynamoDB read and write speeds are determined by the number of available partitions.
- Partitions are automatically added or removed based on the provisioned throughput and storage requirements by DynamoDB.
- Provisioning Reads and Writes
    - Read Capacity Unit (RCU) provides 1 strongly consistent read (or 2 eventually consistent reads) per second for
      items < 4KB in size.
    - Write Capacity Unit (WCU) provides 1 write per second for items < 1KB in size.
- A partition can support a maximum of 3000 RCUs OR 1000 WCUs.
    - E.g. If you provision 5500 RCUs and 1500 WCUs
    - (5500/3000) + (1500/1000) = 3.333 -> 4 partitions
- A partition can hold ~ 10GB of data.
    - E.g. If you provision 45GB of data
    - (45/10) = 4.5 -> 5 partitions


## Indexing

- Indexing allows for faster retrieval of data.
- The primary key is indexed by default.
- Secondary indexes
    - LSI: Use Local Secondary Indexes when you need to change the sort key.
        - Use the primary key but with different sort keys; same attributes.
        - Up to 5 LSIs.
        - LSIs count against the provisioned throughput (performance) of the DDB table.
        - They need to be created at the time of table creation, and cannot be modified or deleted.
        - Eventual consistency or strong consistency.
        - Total size limited to 10GB.
    - GSI: Use Global Secondary Indexes when you need to use a different partition key.
        - Can use any attribute as secondary key, different sort key.
        - Limit the projected attributes.
        - Unlimited number of GSIs.
        - Can be added to existing tables.
        - Eventual consistency
        - Consistency lags behind the main DynamoDB table.
        - No size restrictions.
        - Like adding another DynamoDB table which gets changes propagated from the main table.
        - GSIs have their own provisioned throughput independent of the main table.
        - If GSIs run out of provisioned throughput, the main table will be throttled not just the GSIs.

- When to use LSI?
    - Index data size has to be less than 10GB.
    - Just need to have an additional sort key using the same primary key.
    - All of those were true at the time you created the table.
    - Strong consistency is required.
    
- When to use GSI?
    - Index data size is greater than 10GB.
    - Eventual consistency works for you.
    - Thought about it after you created the table.
    - One of the attributes has too much data and it is not required; exclude it in the projected attributes.
    - Sparse index (e.g. only users have a certification).
    - Much smaller data set.

- Projected Attributes - attributes copied from the table to the index, in additional to the primary key attributes and
  index key attributes.
  
- Projection Type
    1. KEYS_ONLY - Only the index and primary keys are projected (smallest index - more performant)
    2. INCLUDE - Only specified attributes are projected.
    3. ALL - All attributes are projected (biggest index - least performant).


## Reading Data
1. Primary key lookup
    - Very efficient because it searches indexes only.
2. Query
    - Can be used on any table with a composite primary key (partition and sort key)
    - Look up the primary key by value and search through the sort key
3. Scan
    - Don't do it! Scanning the whole table; Lots of RCUs if it is a big table.
    - You can reduce Page Size of an operation with the Limit parameter, to limit how much data you try to retrieve at
      the same time.
    - Avoid performing Scans on mission-critical tables.
    - Program your application logic to retry any requests that receives a response code saying that you exceeded
      provisioned throughput (or increase throughput).

### Data accuracy on read: Eventually Consistent vs. Strongly Consistent
#### Eventually Consistent Reads
When you read data from a DynamoDB table, the response might not reflect the results of a recently completed write
operation. The response might include some stale data. If you repeat your read request after a short time, the response
should return the latest data.

#### Strongly Consistent Reads
When you request a strongly consistent read, DynamoDB returns a response with the most up-to-date data, reflecting the
updates from all prior write operations that were successful. A strongly consistent read might not be available if
there is a network delay or outage.

DynamoDB uses eventually consistent reads, unless you specify otherwise. Read operations (such as **GetItem**,
**Query**, and **Scan**) provide a **ConsistentRead** parameter. If you set this parameter to true, DynamoDB uses
strongly consistent reads during the operation.


## Writing Data

- Writes are free Reads
- Atomic Counters
    - Allow you to increment or decrement the value of an attribute without interfering with other Write requests.
        - $a += 1
        - $a -= 1
    - Requests are applied to the order that they are received.
- Conditional Writes
    - Help coordinate Writes
    - Checks for a condition before proceeding with the operation.
    - Supported for **PutItem**, **DeleteItem**, **UpdateItem** operations.
    - Specify conditions in ConditionExpression:
    - Can contain attribute names, conditional operators, and built-in functions.
    - A failed conditional write returns **ConditionalCheckFailedException**.

## Throughput

### Provisioning Reads and Writes

- **Read Capacity Unit (RCU)** provides 1 **strongly consistent read** (or **2 eventually consistent reads**) per second
  for items **< 4KB** in size.
    - E.g. Your items are 10KB in size and you want to read 80 strongly consistent items from a table per second.
    - How many RCUs in 10KB:  10KB / 4KB = 2.5   =>  3
    - Each item requires 3 RCUs; 80 items need:  3 * 80 = 240 RCUs.
    - Strongly consistent reads =>  240 RCUs
    - Eventually consistent reads => 120 RCUs

- **Write Capacity Unit (WCU)** provides 1 write per second for items **< 1KB** in size.
    - Adding, updating, or deleting an item in a table also costs a WCU and additional WCUs to write to any LSI and GSI.
    - E.g. Your items are 1.5KB in size and you want to write 20 items per second.
    - How many WCUs in 1.5KB:  1.5KB / 1KB = 1.5   =>  2
    - Each item requires 2 RCUs; 20 items need:  2 * 20 = 40 WCUs

- Scale without downtime but takes time.

### Throughput - Throttling

- Consistently reading or writing more than the provisioned RCU/WCU per partition.
- Using one partition or key extensively.
- Stale or unused data occupying partition and key space.

### Throughput - Burst Capacity

- Underutilized RCU/WCU are retained as burst capacity by DynamoDB if available.
- Five minute of unused read and write capacity
- Used for floods.
- Not guaranteed.

### Throughput - Reserved Capacity

- Discounted capacity units can be purchased ahead of.
- Reserved automatically get applied to your RCU/WCU usage.
- Over time, this can significantly reduce your costs.

## Monitoring using Amazon CloudWatch

- ConsumedReadCapacityUnits - RCUs per time period used
- ConsumedWriteCapacityUnits - WCUs per time period used
- ReadThrottleEvents - Provisioned RCU threshold breached
- WriteThrottleEvents - Provisioned WCU threshold breached
- ThrottledRequests - ReadThrottleEvents + WriteThrottleEvents

## Integration
- DataPipeline
- Amazon EMR
- Amazon Redshift
- S3
- Kinesis Client Libraries

## Use Cases
- Logging high throughput data - e.g. coming off Kinesis Streams
- Almost like a cache - but don’t use is as a cache
- State table
- Active logins

**Bad Use Cases**
- You application requires a SQL interface.
- Queries that require joins.
- Data size greater than 400KB per item/row.
- Infrequent access
- Low throughput data
- Full table scans on huge tables

## Best Practices
- Keep item size small
- Store metadata in Amazon DynamoDB and blobs in Amazon in S3
- Use a table with a hash key for extremely high scale
- Avoid hot keys and hot partitions
- Use table per day, week, month etc. for storing time series data
- Use conditional updates
- - Run a cache in front of DynamoDB if needed
- Test applications at scale
- Do not depend on burst capacity
