# R53 hosted zones

## Hosted zones

- Public/private
- DNS database for a domain

## Public hosted zones

- DNS database (zone file) hosted by Route53 (Public Name Servers)
- Accessible from the public internet and VPCs
- By default, 4 Route53 Name servers host a zone
- To use the NS servers, point NS records to the NS servers' IP addresses
- In hosted zone, create resource records (DNS records)

## Private hosted zones

- Not public
- Associated with VPC only
- Can be accessed by UI, CLI, API own account
- Can be accessed by CLI, API different account
- Split-view (?) - overlapping public & private for public and internal use with the same zone name

### Demo steps

- Create a private hosted zone, random domain
- Associate it with default vpc
- Create EC2 Instance, associate it with another vpc
- Connect to ec2 instance, ping the domain => no response
- Back to the hosted zone, edit. Associate the hosted zone with the vpc of the instance. 
- Wait some minutes, back to the ec2 instance console, ping the domain => Got repsonse

# CNAME vs Alias

- CNAME can only be used on sub-domain => ALIAS record can be used on naked domain (without any sub/prefix)
- Free of charge for ALIAS request pointing at AWS resources

# Simple routing

- Create a record, A, can have multiple values. All values are returned in a random error, client chooses and uses 1 value
- Does not support health check
 
# Route53 health checks

- Independent from records, used by records
- Checkers located globally
- By default check every 30s (costs extra if check every 10s)
- Health check tcp, http/https, http/https with string matching
  + tcp: Must connect within 4 seconds
  + http/https: Must connect within 4 seconds, return http code 200/300 within 2 seconds after connecting
  + http/https with string matching: same as http/https, then response body must contain a string. The string must be in the first 5120 bytes of the response body
- Healthy / unhealthy

# Failover Routing

- Multiple records, same name, different endpoint
- 1 primary, other secondaries/backups
- Health check on primary
- Primary dead => Request go to backups
- Use case: secondaries/backups point to s3 to show an out-of-band (503) page

## Demo Steps

- Create EC2 Instance, host some html
- Create S3 bucket, upload some html, allow public access & apply resource policy
- Create elastic IP, associate the IP to the EC2 instance
- Create health check, set 10 seconds interval, point to domain html path
- On Route53, create failover routing
  + Primary: A -> elastic IP. Health checker ID point to the created health check. Record ID: EC2
  + Secondary: A -> Alias to S3 website endpoint -> the S3 endpoint. Record ID: S3. Uncheck "Evaluate target health"
  
# Multi Value Routing

- Multiple records, same name, different endpoints. Up to 8 records will be returned to requesters/clients. > 8 records => random select 8 records
- Each record has associated health check
- Increase availability to clients

# Weighted Routing

- Same as multi value routing, can be combined with health check or not. If unhealthy selected, skip to a healthy
- Each record has a weight
- 0 => never return

# Legacy-based routing

- Return record with estimated lowest latency to the user.
- Should be used when optimize for performance & user experience
- Like multi value routing, each region only has 1 record only 
- Can be combined with health check. If health fail, next lowest latency address will be returned to client

# Geolocation Routing

- Verify the IP of the user => get location (continent/country)
- Check matching records based on the location of the user and location of records
- Should be used when restrict contents to users/language specific contents...
- State => country => contitnent => default. No default => NO answer

# Geoproximity

- Return record with the calculated shortest path to the user.
- Internal resource => region
- External resource => Provide longtitude & latitude of the resource
- Can be adjusted with bias

# R53 Interoperability <= Rewatch

- Domain registrar
- Domain hosting

# DNSSEC with Route53 <= Rewatch
