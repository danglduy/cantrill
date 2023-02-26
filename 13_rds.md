# ACID vs BASE <= Rewatch

- DB transactions models
- CAP theorem
- ACID = consistency
- BASE = availability

## ACID vs BASE

- ACID = Atomic, consistent, isolated, durable transaction
  - SQL, RDS
- BASE = Basically available, soft state, eventually consistent
  - noSQL, DynamoDB
- DynamoDB does offer ACID features with "DynamoDB transactions"
  - Applications need to be aware

# Databases on EC2

- Should
  - OS level access
  - DB option tuning (DBROOT)
  - Vendor demands
  - DB or DB version AWS does not support
  - Architecture AWS does not provide (replication/resilience)
  - "Just want it"
- Should not
  - Admin overhead
  - Backups / DR Management
  - EC2 is single AZ
  - AWS has some features that on-premise versions do not have
  - Not easy to scale
  - Replication
  - Performance

# RDS architecture

- Database server as a service (DBSaaS)
- No admin overhead
- Multiple database on one DB server
- MySQL, MariaDB, PostgreSQL, Oracle, Microsoft SQL server
- Amazon Aurora: custom database by Amazon
- Managed service, no access to OS or SSH
- Operate within a VPC

## RDS Subnet Group

- List of subnet that RDS can use for an database instance or instances
- In subnet group, RDS will pick one for primary, another for standby
  - Random, but primary & standby in different availability zones
- Subnets can be public or private
  - Public: Can be accessed from the Internet
  - Private: Can be connected from the VPC or connected networks (VPN or Direct Connect)

## RDS Instance

- Can contain multiple databases
- Each instance has **dedicated** storage (EBS)
- Multi-AZ => Primary will replicate to Standby using synchronous replication
  - Data is synchronized to standby right after the primary receives => same set of data
- Read replica
  - Asynchrounous replication
  - Can be in the same region or a different region
  - To be used to scale read load
- Backup
  - Backup and snapshot to S3
  - Multi-AZ => Backup is taken from standby instance => No performance impact

## RDS Costs

- Instance size & type
- Multi-AZ
- Storage type & amount
- Data transfer cost
- Backups & snapshot
  - 2TB of free snapshot
  - Per GB month cost (1 TB \* 1 month = 500 GB \* 2 months)
- Licensing\* (if applicable)

# Migrate EC2 Database to RDS

- Use CloudFormation to create vpc, ec2 instances, subnets, security groups...
- RDS create a subnet group, choose database subnets (check from vpc subnets)
- RDS create a RDS instance, choose region, vpc, subnet group, create security group for the rds
- Allow the EC2 instance to connect to the RDS instance by allowing inbound connections in the new created RDS security group
- Use mysqldump/mysql in the EC2 instance to dump and restore databases into the RDS instance
- Update the application's database configuration

# RDS MultiAZ

- Synchronous replication
- Not freetier

## Old/tranditional MultiAZ

- Standby instance cannot be used for improving performance, only high availability
- Normally requests are called to primary rds instance cname
- When failed, cname are pointed to standby instance
- Delay about 60-120 seconds
- One standby replica only
- Same region only
- Backups/snapshots are taken from standby to prevent performance impact on rds primary instance

## MultiAZ cluster

- 1 writer (also can be read) synchrounously replicate to 2 readers (can only read) in other availability zones.
- Faster hardware than MultiAZ instance mode (nvme ssd)
- Writes to local storage => flushed to EBS
- Replication is via transaction logs ... more efficient (?)
- Failover ~ 35 seconds
- Vs Aurora MultiAZ cluster
  - RDS: 2 readers
  - Aurora: More than 2 readers
- Write data to writer marked as commited only when 1 of the readers confirms
- Ways of access
  - Cluster endpoint => writer. Used for reasd, writes and _administration (?)_
  - Reader endpoint => readers (incl. the writer)
  - Instance endpoint => specific instance (NOT recommended for normal usage, only for testing / fault checking)

# RDS Backup and Restore

## Backups

- Storage: AWS Managed S3, not visible to user in S3 dashboard, but only in RDS dashboard
- Single instance: Taken from the only available instance, I/O interrupted, performance impacted
- MultiAZ: Taken from the standby instance
- Snapshot: Manual backup
  - First full, later incremental
  - Can be restored to the point of the creation time
  - Manually managed
- Automated Backup: Like snapshot, but automated once per day
  - First full, later incremental
  - Fixed time
  - Every 5 minute, rds transaction logs are also put into s3
  - Can be restored to every 5 minute moment (Every 5 minute Recovery Point Objective can be reached)
  - Not keeping indifinitely, retention time: 0 -> 35 days
    - 0: disable automated backup
  - When delete a RDS instance, user can choose to keep automated backups, but the backups are still affected by the retention policy
    - Keep data => Create a final snapshot
- Can be replicated to other regions (snapshots & transaction logs)
  - Charges: cross-region data copy & storage of the destination region
  - Not enabled by default, need to be configured within automated backups

## Restores

- Create a new RDS instance with new address
- Not fast

# RDS Read Replicas

- Readonly DB replicas
- Asynchronous synchronization
- Not automatically used, application need to be aware
- Can be created within same region as the RDS or different region
- Purpose: Read scaling
- Read-replicas can be used to create other read-replicas (not recommended, data lag)
- Improve RTO: Can be promoted, faster than restoring data from backups/snapshots
- Readonly, until promoted
- Global availability improvements

# RDS Security

- Authentication
  - Local user: user/password
  - IAM user
    - RDS instance create a local database user account
    - Configure allow the user to authenticate using an AWS authentication token
    - Attach IAM policies to IAM user/role, map IAM identity to the local RDS user
      - Allow identities to run generate-db-auth-token operation => create 15-minute valid token can be used as DB user password
- Authorization
- Encryption in transit
  - Via SSL/TLS
- Encryption at rest
  - Database engine
    - RDS MSSQL & RDS Oracle support TDE
      - TDE: Transparent Data Encryption
    - RDS Oracle supports CloudHSM
      - Stronger key control
      - Encryption key are managed by user, not known by AWS
  - EBS encryption/KMS
    - Handled by HOST/EBS
    - Storage, Logs, Snapshots & replicas are encrypted with same key
    - Encryption cannot be removed

# RDS Custom

- Fills the gap between RDS & EC2 database
- RDS: fully managed
- EC2 db: unmanaged
- Current works for MSSQL & Oracle
- Can connect using SHS, RDP, Session Manager
- RDS Custom Database Automation
  - Need to check to prevent interruption by automation
  - When customize: Pause automation, do customization, resume automation

# Aurora Architecture

- Different from RDS
- Uses a "Cluster"
- 1 single primary instance + 0+ replicas
- Replicas can be used as read (vs standby instance in RDS instance mode)
- No local storage - uses a shared cluster volume
  - Fast provisioning, improved availability, better performance
  - Max 128TB
- Max 6 replicas across AZ
- Data written to primary DB instance => synchronously write to replicas
- Replication happens at storage level
- Max 15 replications

## Aurora Storage Architecture

- All SSD based - high iops, low latency
- No need to allocate storage
- Billed based on usage/consume
- Max 128TIB
- Storage which is freed up can be re-used
- Replicas can be added and removed without requiring storage provisioning
- High water mark - billed for the most used - being removed in the future

## Endpoints

- Cluster endpoint: primary instance
- Reader endpoint: load balance all instances, read only (incl. the primary instance)
- Customer endpoint: can be created to point to a specific instance

## Costs

- No free-tier
- Computer - hourly charge, per second, 10 minute minimum
- Storage - GB-month consumed, IO cost per request
- DB size in backups are included

## Restore, clone & backtrack

- Backups same as RDS: Restore create a new cluster
- Backtrace: be enabled in each cluster, allow in-place rewinds to a previous point in time
- Fast clones: Create a new cluster, share the old storage, store new changes since the point of the clone - copy-on-write => Faster, small changes

# Aurora Serverless

- Aurora Serverless to Aurora ~ Fargate to ECS
- No need to manage individual database instances
- Scalable - base unit: ACU - Aurora Capacity Unit
- Cluster has a MIN & MAX ACU
- Can go to 0 and paused
- Per-second billing
- Same resilience as Aurora (6 copies across multiple AZs)
- Pool hosts multiple ACU, shared across multiple customers

## Use cases

- Infrequently used applications
- New applications - not sure load
- Variable workloads
- Unpredictable workloads
- Development & test databases (can be decreased to 0)

# Aurora Global Database

- 1 master region, upto 5 secondary regions
- Primary region: 1 write, upto 15 read instances
- Secondary regions: Upto 16 read instances
- 1s replication
- Cross region DR & BC
- Global read scaling
- Replication no impact on DB performance

# Aurora Multi-master writes <= **rewatch!!!**

- Default: single master
  - 1 r/w, 0+ RO replicas
  - Cluster endpoint write, read endpoint load balanced reads
  - Failover takes time - replica promoted to R/w
- Multi-master: All instances are R/W
  - No cluster endpoint / read endpoint / load balancer
  - Application need to aware of multiple endpoints and connect to each endpoint individually
  - When there is a write request, the instance proposes the write request to all storage nodes in all AZs
  - Each node needs to confirm/reject the write request. Reject when the request is conflicted with other write requests => Quorum
  - After all nodes agree to write the change, the change is replicated to all other r/w instances => consistent data between r/w instances
  - Necessary to build a fault tolerant system
  - **rewatch!!!**

# RDS proxy

- Instead of connecting directly to database, connect via proxy (connection pool)
  + reduce opening / closing connections, reduce consuming resources
  + reduce latencies
  + multiplex connection from proxy to database
  
# Database migration service

- Managed database migration service
- Runs using a replication instance
- Need to define source and destination endpoints
- Need to define source and target databases
- One endpoint must be on AWS
- Replication jobs mode
  + Full load: one off migration of all data
  + Full load + CDC (Change data capture): On going replication & captures change, applied when the first full load migration is done
  + CDC only (?): First one off is migrated using another tool
  
## Schema conversion tool (SCT)

- Convert one database engine to another
  - Including DB to S3
  - Example: On-premises *MSSQL* to RDS *MySQL*
