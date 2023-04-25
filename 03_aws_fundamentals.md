# Public vs Private services

- Networking viewpoint
- Public: S3
  - Public accessible
  - Restrict by permission
- Private: EC2
  - Can only access by VPC
- Zones
  - Public internet zone
  - AWS Public Zone
  - AWS Private Zone

# AWS Global Infrastructure

- Region
  - Full computer, storage, database
  - Advantages
    - Geographic separation
    - Geopolitical separation
    - Location control - performance
- Edge location
  - Content distribution. e.g.: Netflix
  - Edge compution
  - More locations than regions
- Region & AZs
  - Region code
  - Region name
- Service Resilience
  - Global resilient: IAM
  - Region Resilient: VPC
  - AZ Resilient: EC2
- VPC Basics
  - Virtual network
  - Regional resilient
  - Private & Isolated by default
  - 2 types of VPC: Default & Custom
- Default VPC
  - VPC CIDR 172.31.0.0/16
  - Each AZ has 1 subnet, can't be overlap
  - One per region. Can be deleted
  - /20 each AZ subnet
  - Internet Gateway (IGW)
    - Security group
    - NACL
  - Subnet assign public IPv4 Address for all services

# EC2 Basics

- IAAS - provides virtual machines
- Private by default - uses VPC
- Allow public access - VPC supports
- AZ resilient
- Different instance sizes & capabilities (GPU, networking, speed)
- On demand (per second)
  - CPU, Memory
  - Storage
  - Extra software
- Storage
  - Local on-host storage
  - Elastic block storage (EBS) - network storage
- State
  - Running
  - Stoppped
  - Terminated
- AMI - Amazon Machine Image
  - Can be used to create EC2 instances
  - Can be created from EC2 instances
  - Permissions
    - Public - Anyone can use
    - Owner - Implicit allow
    - Explicit - Specific AWS Accounts allowed
  - Boot volume: Required - to boot OS
  - Block device mapping - map volume with device ID
- Connect to EC2
  - Windows: RDP
  - Linux: SSH

# S3 Basics

- Global resilient - regional based resilient
- Public service, unlimited data & multi-user
- Economical & can be accessed using UI/CLI/API/HTTp
- Objects & Buckets (containers)
  - Key/value
  - 0 byte ... 5TB
  - Metadata
  - Version ID
  - Access control
  - Sub resources (?)
- Bucket - in region
  - Name: Globally unique
  - Unlimited number of objects - each 0..5TB
  - Flat structure
  - name: 3-63 chars, lowercase, no underscores
  - start with lowercase letter/number
  - Limit: 100 soft, 1000 hard per AWS account
- S3 patterns
  + Object storage - not file or block storage
  + Can't mount as drive
  + Great for large scale distribution
  + Great for off loading data
  + Should by default storage for AWS Services
- ARN: Amazon Resource name
  + arn::aws::S3::::xxxx

# CloudFormation Basics
- Create, update, delete AWS infrastructure in a consistent way using templates
  + Sync 2 way
- Format: YAML / JSON
- All templates have a list of resources
  + at least one resource
  + Description: free text
    * If the template has "AWSTemplateFormatVersion", the description needs to follow the format
  + Parameters: Add fields to prompt user
  + Mappings: Lookup table from parameters to real values
  + Conditions: Decision making in template
  + Outputs: When finished, show results 
- Recap
  + Template
  + Stack
  
# Cloudwatch basics
- Collects & manages operational data
- Metrics 
- Logs
- Events

# Shared responsibility

# HA vs FT vs DR

- High availability
  + E.g.: Active / standby
  + Maximize a system's online time
  + Failed components can be replaced / fixed quickly
  + Metrics: % of uptime
- Fault tolerance
  + E.g.: Active / active
  + System can operate normaly when some components of the system failed
  + Much more complex and expensive than HA
- Disaster recovery
  + What to plan and do when disaster occurs
  + 2 parts
    * Before/pre-planning
      - Backup
      - IT Staff
      - Resilience
    * After / DR Process

# Route53 fundamentals

1. Register domains
2. Hosted zones ... managed nameservers
