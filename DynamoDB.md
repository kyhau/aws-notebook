
## Partitions and Provisioning Reads and Writes

- DynamoDB read and write speeds are determined by the number of available partitions.
- Partitions are automatically added or removed based on the provisioned throughput and storage requirements by DynamoDB.
- Provisioning Reads and Writes
    - Read Capacity Unit (RCU) provides one strongly consistent read (or 2 eventually consistent reads) per second for items < 4KB in size.
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
    - GSI: Use Global Secondary Indexes when you need to use a different partition key.
    - LSI: Use Local Secondary Indexes when you need to change the sort key.
- Projected Attributes - attributes copied from the table to the index, in additional to the primary key attributes and index key attributes.
- Projection Type
    1. KEYS_ONLY - Only the index and primary keys are projected (smallest index - more performant)
    2. INCLUDE - Only specified attributes are projected.
    3. ALL - All attributes are projected (biggest index - least performant).
