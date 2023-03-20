# VPC Flow Logs

- Capture metadata only, not contents. Capture contents => need packet sniffer
- Can be attached to 3 levels
  - VPC: All ENIs in the VPC
  - Subnet: All ENIs in the Subnet
  - ENIs: Individual ENIs
- Not real time, there are delay between flow occuring and the data showing within the Flow Logs product
- Destinations: S3 or CloudWatch Logs, or Athena for querying
- Flow Logs can capture ACCEPTED, REJECTED or ALL metadata
- Flow Log records: individual items of Flog Logs
  - Note: Security group when used only log accepted, because its stateful so response will be automatically returned without any evaluation. NACL will log both request and response if NACL was used.
- Excludes
  - Metadata service: 169.254.169.254, 169.254.169.123
  - DHCP
  - Amazon DNS server
  - Amazon Windows license server

# Egress-Only Internet Gateway

Allows only connections initiated from inside VPC to outside

Reasons:

- IPv4: Private IP & Public IP
- IPv6: Public IP only
- NAT: Only works with IPv4, allows private IPs to access public networks but does not allow external initiated connections
- IPv6 can be accessed publicly, both IN and OUT

=> Egress-Only is outbound-only for IPv6

# VPC Endpoints (Gateway)

- Provide private access to S3 and DynamoDB (these services are public services)
- Before (without using VPC Endpoints): Private instances -> VPC router -> NAT Gateway -> Internet Gateway -> S3
- After (using VPC Endpoints): Private instances -> VPC Router -> Gateway Endpoint -> S3
  - Gateway Endpoint needs to associate to the subnet with "Prefix List" **recheck!**
- Region resilient
- Cross-region not accessible
- S3 also needs to set to private only by denying all access except the gateway endpoint

# VPC Endpoints (Interface) _rewatch_

- Provide private access to AWS Public Services
- Historically not allowing s3 and dynamodb. But not s3 is supported
- **Not HA** by default, added to specific subnets, an ENI, _per AZ_
- Network access controlled via security groups
- Endpoint policies - restrict what can be done with the endpoint
- TCP and IPv4 only
- Uses PrivateLink
- Endpoint provides a NEW service endpoint dns
- Can connect to the NEW service endpoint DNS, or use Private DNS, Private DNS overrides the default service DNS name

# VPC Endpoints - demo _rewatch_

## Gateway

- Private VPC
- No route/internet gateway route to s3
- VPC > Create Endpoint
  - Services: com.amazone.\[region\].s3
  - Services: Choose type "Gateway"
  - VPC: Choose VPC
  - Route tables: Choose route tables to insert endpoint routes (prefix list) into

## Interface

- Private VPC
- No route/internet gateway route to sns
- VPC > Create Endpoint
  - Services: com.amazone.\[region\].sns
  - Services: Choose type "Interface"
  - VPC: Choose VPC
  - Subnets: Choose AZ > subnets
  - Additional: Enable DNS name
- VPC > Edit Endpoint
  - Enable private DNS names - Override dns names of services

## Egress-Only Internet Gateway

- VPC > Egress-only internet gateways
  - Create egress only internet gateway
  - VPC: Attach VPC
  - Create
- VPC > Route tables
  - Choose a route
  - Tab routes > Edit routes
  - Destination: `::/0`, Target: the created egress only internet gateway

# VPC Peering

- Direct unencrypted network link between only 2 VPCs
- Same/cross region, same/cross account
- (optional) Public hostnames resolve to private IPs
- Same region Security Group's can reference peer Security Groups
- VPC Peering does NOT support transitive peering: A-B & B-C !=> A-C
- Routing configuration is needed, SGs & NACLs can filter
- Basically VPC Peering Gateway is created
- VPC CIDRs can not overlap (no overlapping IP addresses)

# Demo VPC Peering

- VPC > Peering connections
  - Create peering connection
  - Select local VPC: select a VPC
  - Select another VPC:
    - Account: Same/Differnet
    - Region This region/another region
    - Select a VPC
  - Create peering connection
  - Actions: Accept request
- VPC > Route tables
  - Choose the first VPC route table
  - Tab routes
  - Edit routes
  - Add route
    - Destination: the second VPC CIDR
    - Target: pcx-xxxxxxxx: The peering connection
  - Choose the second VPC route table
  - Table routes
  - Edit routes
  - Add route
    - Destination: The first VPC CIDR
    - Target: pcx-xxxxxxxx: The peering connection
- Security groups (to allow ping from the first VPC's instances to the second VPC)
  - Edit the second VPC security groups
    - Inbound rules:
      - Edit inbound rules
      - Add rule
      - Type: ICMP - IPv4
      - Source: The first VPC security group ID
