# Regional and Global AWS Architecture

- Global service location & discovery: What happens when users go to domain (exp: netflix.com), how machines discover where to connect
- Content delivery (CDN) & optimization
- Global healthcheck & failover
- Regional entry point
- Regional scaling & resilience
- Application services & components

## Global

- DNS for ip resolution: Route53
  - Service discovery, regional based health checks & request routing (based on specific factors: location, latency...)
- Content Delivery (CDN): CloudFront
  - Cache content globally - as close to end users as possible to improve performance

## Regional

- Web tier: Application Load Balancer / API Gateway
- Compute tier: EC2, Lambda, or Containers
- Storage tier: EBS (Elastic Block Storage), EFS (Elastic File System), S3 (Simple Storage Service)
- Caching: ElastiCache / DynamoDB Accelerator (DAX)
- Database tier: RDS, Aurora, DynamoDB, Redshift
- Application Services: Kinesis, Step Functions, SQS, SNS

# Evolution of Elastic Load Balancers

3 types of elastic load balancers available: v1 (not recommmended), v2 (recommended, faster, cheaper, support target group & rules)

- Classic load balancer (v1)
  - Not really layer 7, lacking features, 1 SSL per CLB
- Application load balancer (v2)
  - Layer 7
  - HTTP/S/WebSocket
- Network load balancer (v2)
  - Layer 4
  - TCP, TLS & UDP

# Elastic Load Balancer Architecture (ELB)

- We have multiple AZs in a region
- We choose to put ALB into AZs
- When connect to the DNS name of the load balancer, connections are made to the nodes of that load balancer
  - DNS name are resolved to all the nodes of the load balancer
  - Equally distributed
  - Each nodes are in multiple AZs, scale within AZs
- Internet facing / Internal
  - Internet facing: Nodes of the load balancer are given public addresses and private addresses
    - Can access both public & **private** instances
  - Private: Nodes are given private addresses only
    - Usually be used to allow scaling between application tiers (example: web connect to application servers via internal load balancer)
- Listener configuration: configuration how connections from users arrive at the load balancer of nodes => controls what the load balancer is listening to: protocols & ports will be accepted & communicate with targets on a port and protocol
- Load balancers need 8 or more free ip addresses in subnets
  - Minimum /28 subnet might work but not properly
  - /27 subnet is suggested
- ELB be used to allow scaling between application tiers
  - Architecture: 2 AZs, 3 tiers: web, app, db across 2 AZs.
  - Web: 1 public facing ELB, 1 auto scaling group
  - App: 1 private internal ELB, 1 auto scaling group
  - DB: 1 private internal ELB, 1 auto scaling group
  - When 1 tier has difficulties handling connections/requests, auto scaling group will scale the tier without affecting other tiers (other tiers are not aware of internal scaling behaviors because tiers are connecting via private load balancers) => tiers be scaling in/out independently

## Cross-zone load balancing

- Same zone load balancing: Uneven distribution to nodes within different AZs
  - AZ A 4 nodes, AZ B 1 node => 1 node of AZ B will get 50% of total connections
- Cross-zone load balancing: Distribution to all registered instances across all AZs
  - Not enabled by default for NLB.
  - Enabled by default for ALB.
  - Has cross zone data transfer fee

# ALB vs NLB

## v1 (CLB)

- Domain -> CLB -> Auto scaling Group
- CLB supports only 1 SSL only
- Multi domains => multi CLBs
- Not scale
- SNI not supported

## v2 (ALB/NLB)

- 1 ALB supports multiple SSLs
- Support rules & target groups
- Host based rules using SNI
- Allows consolidation

## ALB

- Layer 7
- Supports HTTP/HTTPs only
- L7 => Can inspect content type, cookies, headers, ...
- HTTPS (SSL/TLS) always terminated => no unbroken SSL (security)
- A new connection is made to the application
- ALB must have SSL certs if HTTPS is used
- Health checks evaluation application health ... layer 7 (instead of checking connection status, can inspect content types...)
- Rules <= **rewatch**
  - Default: catchall
  - Processed in priority order
- Example:
  - Domain xxx.xx
  - Standard rule: Forward to a normal target group
  - Source IP rule: If check source IP match xx.xx.xx.xx => match an alternative rule => forwarded to an alternative target group
  - ...

## NLB

- Layer 4: TCP, UDP, TLS...
- High performance, latency ~ 25% of ALB
- Health check: Check ICMP / TCP Handshake
- NLB can have static IPs - useful for whitelisting
- Can forward TCP ... unbroken encryption
- Used with private link to provide services to other VPCs

# Launch configuration and templates

- Allow to define the configuration of an EC2 instance
  - AMI, Instance type, storage, key pair
  - Networking, Security groups
  - Userdata, IAM role
- Launch configurations
  - Cannot be modified after created, only can be replaced
  - Be used to launch auto scaling groups
- Launch templates
  - Can be updated & versioned
  - Has additional features: T2/T3 Unlimited, placement groups, capacity reservation, _elastic graphics_
  - Be used to launch auto scaling groups & EC2 instances

# Auto Scaling Groups

- Free
- Provides auto scaling and self-healing for EC2
- Uses Launch templates / configurations
- Has a minimum, desired, and maximum size (eg 1:2:4)
- Keep running instances at the desired capacity by provisioning or teminating instances
- Scaling policies automate based on metrics
- Auto Scaling Groups define where to put instances to launch
  - Linked to a VPC and subnets within the VPC
  - Try to launch instances evenly between AZs, but not always accomplished

## Scaling policies

- Manual scaling - manual
- Scheduled scaling - time based adjustment
- Dynamic scaling
  - Simple scaling: 1 provision, 1 termination. Exp: CPU > 50% + 1, CPU < 50% - 1. Requires cloudwatch metrics
  - Stepped scaling: similar to simple, but more detailed rules. **<= Check docs**
  - Desired tracking: Ideal amount or sth, ASG handle itself. Exp: Aggregate CPU all instances < 40% => ASG need to scale in/out to keep the desire amount within the target range
- Cooldown periods: Seconds to wait from last scale in/out action before continuing doing so

## Auto scaling groups + Load balancers

- User connect to a domain, the domain point to a load balancer
- The load balancer point to a target group
- Instead of fixing the instances of the target group, auto scaling group can be used to dynamically put instances into the target group
- The auto scaling groups can be configured to use the Load Balancer health checks instead of ec2 status check
  - Can monitor state of HTTP/HTTPS requests
  - Need to be careful to use Load Balancer check.
    - Example 1: Complex app but test static html => static html healthy but app still failed
    - Example 2: App has database, database failed => terminate & re-provision instances => still failed

## Scaling Processes **<= rewatch**

# Auto Scaling Groups Scaling Policies

- Auto scaling policies are not mandatory when creating auto scaling groups
- When there are no policies, auto scaling groups has static values - min, max, desired capacity
- Manual - testing & urgent situations
- Dynamic
  - Simple scaling: 1 factor to decide scaling in/out
  - Step scaling: Range of values (upper/lower bound) of factors to decide scaling in/out
  - Target tracking: Define ideal value, ASG calculates and scales in/out automatically
  - Scaling based on SQS **<= check**

# Auto Scaling Groups Lifecycle Hooks

- Custom actions on instances during ASG actions
- Instance launch or instance terminate transitions
- Instances are paused within the launch/terminate flow until...
  - timeout (default 3600s)
  - explicit resume after custom actions finished using ASG process CompleteLifecycleAction
- ASG Lifecycle Hooks can be integrated with EventBridge or SNS Notifications **<= check**
- Flow
  - Scale out
    - Normal: instance launch Pending -> instance launch InService
    - With lifecycle hooks: instance launch Pending -> (doing custom actions like loading data...) Pending: Wait -> Pending: Proceed -> InService
  - Scale in
    - Normal: instance terminate Terminating -> instance terminate Terminated
    - With lifecycle hooks: instance terminate Terminating -> (doing custom actions like backup data...) Terminating: Wait -> Terminating: Proceed -> Terminated

# ASG HealthCheck: EC2 vs ELB

- EC2 (default), ELB, Custom
- EC2 - Stopping, stopped, terminated, shutting down or impaired (not 2/2) = UNHEALTHY
- ELB - HEALTHY = Running & passing ELB health check
  - can be more application aware (Layer 7)
- Custom - health & unhealthy marked by an external system
- Health check grace period (default 300s)
  - Delay before starting checks
  - Prevent system starting but not ready, health check marked as unhealthy => terminate & re-provision again and again

# SSL Offload & Session Stickiness

## SSL Offload

- Bridging

  - ELB listener is configured for HTTPS
  - Connection is terminated on the ELB
  - ELB initiates a new SLS connection to backend instances
  - EC2 instances will need to decrypt the connections => big overhead for large volume of connections
  - Negatives
    - ELB need to store the certificates
    - Admin overhead before EC2 instances need to store the certificate

- Pass-through

  - ELB does not encrypt certificate (NLB)
  - ELB can see source, destination IP and ports
  - EC2 instances need to decrypt the connections
  - ELB does not touch the encryption

- Offload
  - Same as bridging, except ELB does not encrypt connections when passing the connection to EC2 instances
  - EC2 instances do not need to do encrypt/decrypt operation
  - Cons:
    - Data is in plaintext form

## Connection Stickiness

- User connects to ec2 instances equally distributed: every connection connects to a different ec2 instance
  - Sessions not handled properly => Not working
- Connection stickiness
  - AWS creates a cookie called AWSALB, valid from 1 second to 7 days
  - Connections will be connected to the same backend ec2 instance by checking the cookie
  - Cons: Uneven load between ec2 instances
- Enable by edit Target Group attributes

# Advanced Demo

# Gateway Load Balancers

## What is it?

- Help run and scale 3rd party appliances, like firewalls, intrusion detection & prevention systems
- Handle inbound/outbound traffic
- Traffic enters/leaves GWLB endpoints
- GWLB balances across multiple backend appliances (ec2 instances running security softwares)
- Traffic and metadata is tunneled using GENEVE protocol

## How it works

- Traffic source client -> gateway load balancer endpoint -> gateway load balancer -> (GENEVE Encapsulation tunnel) Appliances -> Gateway load balancer -> gateway load balancer endpoint -> traffic destination app server
- Original packets remain unaltered and back
