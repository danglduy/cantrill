# Boostrap EC2 using User Data

- To reduce post launch time
- Scripts to be run when first launch: 1 time only!
- More flexible than AMI baking
- Can be used along with AMI baking: AMI baking: install packages, boostraping: customize configuration
- Do not pass long-term credentials into user data, not secure
- User data is converted to base64, not validated
- 16 KB limit
- Where to put user data
  - UI when launch a new instance
  - Cloudformation

# Enhance bootstraping with CFN-INIT

## cfn-init

- cfn-init is installed on EC2 OS
- Can be used as a simple system configuration
- User-data: procedural / cfn-init: desired state
- Provided via Metadata and AWS::CloudFormation::Init on a CFN resource
- Cloudformation Template -> Stack -> Instance -> User data call cfn-init -> cfn-init call cloudformation stack with stackID -> do commands to get the instance to the desired state
- Can be setup with cfn-hup to watch for changes after cloudformation stack is updated

## cfn-signal

- Instance cannot know the state inside of Instance
- cfn-signal will call aws cloudformation stack to update signal when all cfn commands are called successfully
- CreationPolicy: Define a timeout to get the signal, if cfn-signal does not call -> Implicitly failed

# EC2 Instance Roles & Profiles

- When attaching an IAM role into a EC2 Instance on the AWS Console UI, actually a Instance Profile is created with the same name and delivered into the EC2 instance
  - When using CloudFormation -> Need to explicitly create the instance profile
- Credentials are delivered via instance meta-data, always valid
  - iam/security-credentials/role-name
- Always use roles rather than adding access keys into instance
  - Except for s3 presigned-url
    - Role: up to 6 hours
    - Secure token service: up to 36 hours for IAM user, 1 hour for root user
    - IAM user access key: up to 7 days
- CLI tools will use Role credentials automatically

# AWS SSM

- Store configuration and secrets
- String, StringList & SecureString (encrypted)
- Hierachies & Versioning
- Plaintext & Ciphertext (KMS)
- Public Parameters (?)
- Can trigger events when changes => other services can take actions based on triggered events

# System and application logging on EC2

- Cloudwatch watches metrics
- Cloudwatch logs handle logging, but not natively captures log inside EC2 instances
- Need Cloudwatch agent to be installed on EC2 instances to capture EC2 instances' logs
  + Requires configurations and permissions to operate
  
## CloudWatch Agent

- Need Agent Config to know which logs to be sent to CloudWatch
- Need permissions to interact with CloudWatch
  + Long term credentials => Not recommended
  + Use IAM role
- Create log group for each file, view in log stream for each instance

### Demo

- Create AWS Cloudformation Stack using Cloudformation template
- Connect to EC2 instance
- Download and install cloudwatch agent
- Create IAM role, with allow policy to CloudWatch and SSM
- Attach the IAM role (Instance profile) to the EC2 instance
- Config the cloudwatch agent: Files to log, log group, retention policy...
- Confirm to upload the cloudwatch agent config to SSM
- Create `/usr/share/collectd` folder and touch a `types.db` file.
- Start up the agent, pull the config from SSM parameter store.

# EC2 Placement Groups

## Cluster

- EC2 instances are put into a single group, directly connected to each other, lowest latency, maximum package per seconds possible (need to use "enhanced networking" ?)
- Cannot span over AZs
- Can span VPC peering but not recommended, negative performance
- Requires supported instance type
- Same type instance recommended
- Same start time recommmmended
- Single stream transfer rate: 10Gbps/s vs 5Gbps normally

## Spread

- EC2 instances are separated from each other, put in different rack/hardwares to ensure that if there are hardware faults happened to a rack/hardware, other EC2 instances are not affected
- HARD LIMIT: 7 instances per AZ
- No support Dedicated Instances or Hosts
- Use case: Critical applications, small number of instances, need to be separated from each other

## Partition

- EC2 instances are put into group (partition)
- Max 7 partitions per AZ
- Large scale but still need resilience (replicate data but still same hardware => fault => die)
- Suitable for topology aware applications (HDFS, HBase, Cassandra...)

# EC2 Dedicated hosts

- Pay for the whole host
- Not pay for instances inside the host
- Full isolation with other customers
- Can mix different type of instances, make sure within limit
- Limits
  + Cannot use RHEL, SUSE, windows
  + Cannot use RDS
  + Cannot use placement groups
- Can be shared with other ORG accounts with RAM (resource access manager)
  + accounts create instances created in dedicated hosts can only see owned instances.
  + owner of the dedicated host can see all instances created by other accounts, but cannot control

# Enhanced networking & EBS optimized

## Enhanced network

- Improve EC2 networking
- SR-IOV (single-root IO virtualization)
  + Tradition: Physical Network Interface is not aware of virtualization. Ec2 instances network interface talk to EC2 host, Host CPU sits in the middle to handle traffic switching => affect performance, higher latency
  + Enabled: Each instance has its own logical network interface. Host network interface is aware of virtualization, handle network from ec2 instances independently of the host cpu
- Higher I/O, lower cpu usage
- More bandwidth, can scale
- Lower latency
- No charge

## EBS optimized

- EBS = block storage over the network
- Historically network was shared and used by both data and EBS
- EBS optimized: Dedicated capacity for EBS
- Most instance types support and enable by default, no charge
- Some older instance types support but with cost

