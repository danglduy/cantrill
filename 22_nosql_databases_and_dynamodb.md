# Amazon DynamoDB Architecture

- NoSQL Public DBaaS - Key/Value & Document
- No self-managed servers
- Scaling options
  - Choose provisioned capacity
    - Manual: fully control the performance
    - Automatic: Allow the system to adjust performance automatically
  - On-Demand
- Availability: Highly resilient, across AZs
  - Data is replicated across multiple storage nodes by default
- Fast performance, single-digit miliseconds
- Handles backups, point-in-time recovery, encryption at rest
- Event-driven integration, watch changes for data, take actions when data changes

## DynamoDB Tables

- Base entity inside the DynamoDB product
- Grouping of Items with the same Primary Key
- Primary Key can be a single key or combination of 2 keys (composite keys) - Partition Key & Sort Key
- Items can have different attributes
- Each item size is 400KB max
- Capacity (speed) is set on a table
  - Writes: 1 WCU = 1 KB per second
  - Reads: 1 RCU = 4KB per second

## Backups

- On demand backup - Full copy of table retained until removed
  - Restore
    - Same or cross-region
    - With or without indexes
    - Adjust encryption settings
- Point in time recovery
  - Not enabled by default
  - Continous record of changes allows replay to anypoint in the window
  - Restore
    - Can restore to new dynamodb table with 1 second interval within 35 days

## Considerations

- NoSQL ... => DynamoDB
- Relational Data => Not DynamoDB
- Key/Value ... => DynamoDB
- Billed based on RCU, WCU, Storage and features

# DynamoDB - Operations, Consistency and Performance

## Reading and Writing

- On-demand
  - Do not need to decide capacity of DynamoDB table
  - Unknown capacity, unpredictable, low admin overhead
  - Pay for price per million R or W units
- Provisioned
  - RCU and WCU are set on table basis
  - Every table has a RCU and WCU burst pool/capacity (300 seconds) **recheck**
- Every operation consumes at least 1 RCU/WCU

## Query

- Query based on partition key only

## Scan

- Iterate through every time
- Expensive
- Consumer RCU all items

## Consistency Model

- Eventual consistent
  - Check 1/3 nodes
  - 50% cost vs strong consistent
- Strong/immediately consistent
  - Always use leader node when reading
  - Costs normal RCU than eventual consistent

# DynamoDB Local (LSI) and Global (GSI) Secondary Indexes

- Improve efficiency of data retrieval within DynamoDB
- Query is the most efficient operation, but limited to work on 1 Partition key value at a time
- Indexes are alternative views on a table data
- Local Secondary Index
  - Different sort key
- Global Secondary Index
  - Different partition key and sort key
- When creating the 2 kinds of index, users can choose some or all attributes of the base table

## LSI

- Can only be created when creating the base table. Cannot add LSI after the table be created
- Maximum 5 LSI per table
- Alternative sort key on the table
- Shares the RCU and WCU with the table

## GSI

- Can be created after the base table be created
- Default 20 GSI per base table
- Alternative Partition Key and Sort key
- Have their own RCU and WCU
- Always eventually consistent. Replication between base and GSI is asynchronous

# DynamoDB Streams & Triggers

## Streams

- Time ordered list of item changes in a table
- 24-hour window
- Actually a kinesis stream
- Enabled per table

## Trigger

- Actions to take place in the event of a change of data
- = Streams + Lambda

# DynamoDB Global Tables

- Multi-master cross-region replication
- Conflict resolution: Last writer wins
- Consistent: Same region writes - strongly consistent read. Others: eventual consistency

# DynamoDB Accelerator (DAX)

- DynamoDB in-memory cache
- Traditional cache: application to cache, cache miss, application to db, updates to cache, cache hit
- DAX: Application to DAX, DAX returns data from db or the cache itself

## Architecture

- Needs to deploy in different AZs
- 2 types
  - item cache
  - query cache
    - cache parameters, whole query data can be rerun and returned same cached data
- Every DAX has an endpoint
- Write-through caching
- Primary node (writes) & replicas
- HA
- Can scale up and scale out
- Much faster reads, reduced costs
- DAX is not public service, but VPC-based

# DynamoDB TTL

- Time to an item to consider the item no longer necessary and then be removed (firstly mark them as expired, then a periodically process will find expired items and delete them)
- Epoch format
- Can create a separate stream to track delete events caused by TTL
- Deletion by TTL not cause costs

# Amazon Athena (#rewatch)

- Serverless interactive querying service

# Elasticache

- Managed in-memory cache
- Manage redis/memcached engines
- Support read-heavy applications

## Memcached vs Redis

- Memcached
  - simple data structures
  - No replication
  - Multi nodes (sharding)
  - No backups
  - Multi-threaded
- Redis
  - advanced structures
  - Multi AZ
  - Replication (scale reads)
  - Backup & Restore
  - Transactions (multiple operations at once, success all or none)

# Redshift architecture

- Petabyte-scale Data warehouse
- Column database
