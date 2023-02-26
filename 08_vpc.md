# VPC Basics <= Rewatch

## Considerations

- Size of the VPC
- IP Range, not overlap other networks
- Future plan
- VPC Structure - Tiers & Resiliency (Availability) Zones

## Steps
- Talk to IT Department
- Clear out all the IP ranges to be excluded while making IP address plan

## Considerations
- Minimum /28, maximum /16
- Avoid common ranges: 10.0, 10.1... => Start 10.16.x.y
- Reserve 2+ networks per region being used per account

# Custom VPCs

- Regional service - All AZs
- Isolated network
- Nothing IN or OUT without explicit configuration
- Flexible - simple or multi-tier
- Hybrid network - other cloud & on-premises
- Default or Dedicated Tenancy
- IPv4 Prviate CIDR blocks & Public IPs
- 1 primary private IPv4 CIDR block
- Min /28 (16 IPs), max /16 (65536 IPs)
- Optional secondary IPv4 Blocks
- Optional single assigned IPv6 /56 CIDR Block

# Stateful vs Stateless Firewalls

## TCP

- Layer 4
- HTTP on TCP 80 / HTTPS TCP 443

## Stateless firewall

- Request: source: client IP + temporary port => dest: server IP + well-known port
- Response: source: server IP + well-known port => dest: client IP + temporary port

Cannot identify links between requests and responses
=> 2 rules for each direction

## Stateful firewall

Can identify links between requests and responses. If allow request => response automatically allowed
=> 1 rule for each request

# NACL

- Traditional firewall for VPC
- Control data in/out of subnets. Data between instances inside a subnet is not affected
- Stateless
- Process in order, match => stop processing
- Default NACL: Allow all requests & responses
- Custom NACL: Only 1 inbound rule implicit (\*) deny => if attach to any subnet => Deny all requests by default.

# Security groups

- Stateful - detect response traffic automatically
- No explicit deny => Can only allow or implicit deny
- Can reference logic resources (other security groups or self references (?))
- Not attached to instances/subnets, but to elastic network interfaces (ENIs ???)

# NAT & NAT Gateway

- IP masquerade: Multiple private IP addresses behind one public IP address.

## NAT Gateway

- AZ resilient
- Managed services
- 45 gbps/seconds
- Cannot use security group, only NACL.
- Don't work with IPv6

## NAT Instance
- Use EC2 instance to function like NAT gateway
- Need to disable source/destination checks
- Can use security group, multi-purpose usage because it's EC2

## Demo create a NAT gateway
- Create NAT Gateway, associate with a public subnet (a subnet as a route table to IGW), allocate an elastic IP
- Create route table, destination 0.0.0.0/0, target NAT gateway
- Associate the route table to private subnets
