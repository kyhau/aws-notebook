# DynamoDB

Table of Contents
- [Partitions and Provisioning Reads and Writes](#partitions-and-provisioning-reads-and-writes)
- [Indexing - Secondary, LSI, GSI](#indexing)
- [Reading data - Primary Key Lookup, Query, Scan, Eventually Consistent vs. Strongly Consistent](#reading-data)
- [Writing data](#writing-data)


## Partitions and Provisioning Reads and Writes

- DynamoDB read and write speeds are determined by the number of available partitions.
- Partitions are automatically added or removed based on the provisioned throughput and storage requirements by DynamoDB.
- Provisioning Reads and Writes
    - Read Capacity Unit (RCU) provides one strongly consistent read (or 2 eventually consistent reads) per second for
      items < 4KB in size.
    - Write Capacity Unit (WCU) provides one write per second for items < 1KB in size.
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

DynamoDB uses eventually consistent reads, unless you specify otherwise. Read operations (such as GetItem, Query, and
Scan) provide a ConsistentRead parameter. If you set this parameter to true, DynamoDB uses strongly consistent reads
during the operation.


## Writing Data