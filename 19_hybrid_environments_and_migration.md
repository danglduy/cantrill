# Border Gateway Protocol (BGP) 101

# IPSec VPN Fundamentals

# AWS Site-to-Site VPN

- VPN between VPC and on-premises network encrypted using IPSec, running over the public internet
- Full HA
- Quick to provision ... less than an hour
- VPC
- Virtual Private Gateway (VGW)
- Customer Gateway (CGW)
- VPN Connection between Virtual Private Gateway & Customer Gateway

## Demo

### Arch

- 3 AZs in a a region
- AWS Public Zone > Virtual Private Gateway
  - Virtual Private Gateway has multiple physical endpoints => High Availability by Design
- Public Internet
- Customer Gateway (CGW) with extenral IP

### VPNs

- Static
  - Have to tell AWS about network range that's in use of on-premises network
  - Have to set Customer Gateway a static config for IP addresses
  - Single point of failure: Customer Gateway => partially high available
  - Full high available: Add another customer gateway, separate internet connection, separate physical location (building...). VGW add new endpoints to create connections to the new customer gateway
- Dynamic
  - Use BGP (Border Gateway Protocol)
  - BGP is configured on both AWS and Customer sides
  - Networks are exchanged via BGP
  - Multiple VPN connections
- Needs to link VPN to virtual private gateway, each endpoint connects to customer gateway interfaces
- At least VPN active => 2 networks are connected

### Considerations

- Speed limitations ~ 1.25 Gbps
- Latency: Inconsistent, public internet
- Cost - AWS hourly cost, GB out cost, data cap (on premises)
- Speed of setup - hours.. all software configuration
- Can be used as a backup for Direct Connect
- Can be used with Direct Connect (VPN initially, Direct Connect later because of longer provision time)

# Demo VPN Site2site

# Direct Connect (DX) Concepts

- Physical connection (1, 10 or 100 Gbps)
- Business Premises connect to DX Location, then to AWS Location
- Port allocation at a DX Location
- Cost: Hourly cost & outbound data transfer
- Provision time: physical cables & no resilience
- Low & consistent lantecy + High speed
- Direct connect can be used to access AWS Private Services (VPCs) and AWS PUblic Services - No Internet Access

# Direct Connect (DX) Resilience & HA

- DX no resilience by default
- Resilience
  - Multiple AWS DX Router - Multiple Customer DX Router - Multiple Customer premises routers

# Direct Connect (DX) - Public VIF + VPN (Encryption)

# AWS Transit Gateway (TGW)

- Network TRansit Hub to Connect VPCs to on premises networks
- Reduce network complexity
- HA and scalable, single network object
- Attachments to other networks
- VPC, Site-to-Site VPN, Direct Connect Gateway

## Compare

### Without Transit Gateway

- Multiple VPCs
- Each VPC connects to other via VPC Peering
- Each VPC connects to customer gateway => increase number of connections => increase complexity

### With Transit Gateway

- Customer gateways connect to a single transit gateway
- Inter VPC routers: Transit gateway connects to each VPC subnet, as VPC attachments
- Can connect peer transit gateways across accounts/same/across regions
- Can connect to Direct Connect Gateway
- Supports multiple route tables allowing complex network topology

# Storage Gateway - Volume
