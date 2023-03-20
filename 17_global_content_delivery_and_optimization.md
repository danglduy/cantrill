# CloudFront architecture

- Content delivery network
- Caching contents to distribute to global network

## Terms

- Origin: Source location of content
  - S3
  - Custom
- Distribution: "configuration" unit of cloudfront
- Edge location: Local cache of data
- Regional edge cache: Larger cache

## Architecture

- Download-only / Read-only caching
- Integrates ACM for HTTPS
- Pattern matching by path

# CloudFront behaviors

- Console
- Distribution: unit of configuration
- Price class: Which edge locations distribution to
  - All edge locations: best performance
  - NA & Europe
  - NA, Europe, Asia, Middle East & Africa
- Firewall: AWS WAF web ACL
- Alternate domain names
- Type of SSL:
  - Custom domain name => custom certificate
- Security Policy: periodically updated
  - Choosing a recent one might prevent old versions of browsers from accessing => Consider to choose not the most recent one to balance between securities and clients' versions
- Behaviors
  - 1 default behavior
  - Path pattern: Default wild card (\*) => Match all
  - Viewer protocol policy
  - Allowed HTTP methods
  - Restrict viewer access
    - Viewsers must use CloudFront signed URLs or signed cookies
    - Trusted authorization type
      - Trusted key groups (recommended)
      - Trusted signer
  - Cache setting
    - Cache policy and oiriginal request policy
    - Legacy cache settings

# CloudFront TTL & Invalidations

## TTL

- Source & CloudFront Edge location
- Source stores data
- First request, Edge location gets data from Source, distributes to clients
- Subsequent requests, edge location returns cached object, even though data in Source might have changed
- After TTL, Edge location checked if Source data changed, if changed replace the current version with the new version of data.
- Default TTL 24 hours
- Minimum/Maximum TTL
- Cache control headers: Cache-Control max-age, Cache-Control s-maxage, Expires
- Custom origin or S3

## Cache Invalidations

- Perform in a distribution
- Applies to all edge locations... takes time, not immediately
- Invalidate regardless of TTL
- By path, supports wildcard
- Should use versioned file names instead of cache invalidation if frequently. Should only be used to correct errors.

# AWS Certificate Manager (ACM)

- HTTP - no security, just text
- HTTPS - added security
- Data is encrypted in-transit
- Certificates prove identity
- Chain of trust - Signed by a trusted authority
- ACM allows a public or private Certificate Authority (CA)
- Private CA - Applications need to trust private CA
- Public CA - Browsers trust a list of providers, which can trust other providers
- ACM can generate or import Certificates
- If generated, ACM can automatically renew
- If imported, user responbiel for renewal
- ACM can only deploy certificates to supported services
- CloudFront & ALBs, not EC2/S3
- Regional service
- Certificates cannot leave the region they are generated or imported in
- ALB in region ap-southeast-2 => need a cert in ap-southeast-2 region
- Global services as CloudFront operate as though within us-east-1

# CloudFront & SSL

- Default domain name (CNAME)
- subdomain .cloudfront.net
- SSL supported by default \*.cloudfront.net cert
- Alternate Domain names (CNAMES) cdn.telegram...
- Verify Ownership (optional HTTPS) using a matching certificate
- Generated or import in ACM in us-east-1
- Enable HTTP or HTTPS, redirect HTTP => HTTPS, HTTPS only
- 2 ssl connections: viewer => cloudfront, cloudfront => origin
- Both need valid public certificates

## SNI

- Historically, SSL enabled site needs its own IP
- Encrypt starts at the TCP connection
- Host headers happens after that - Layer 7 // Application
- SNI is a TLS extension (Server Name Indication), allowing a host to be included
- Resulting in multiple SSL Certs/Hosts using a shared IP
- Old browsers do not support SNI => CloudFront charges extra for dedicated IP

## SSL/SNI

- Client -> CloudFront edge locations: Viewer Protocol
- Edge locations -> Sources: Origin Protocol
- Both needs public trusted certificates
- Origin protocol
  - S3: No need to manage certificate
  - Application Load Balancer: Have ACM generate certificates
  - Custom origin EC2 or On-Prem: Self manage certificates

# CloudFront Origin Types & Architecture <= rewatch

- Single origin
- Origin groups
- Can edit behavior to get contents from a single origin/origin groups

## S3 origin

- Put s3 url into Origin domain input field => auto detect s3 bucket
- Can set
  - Origin access
    - Public
    - Origin access controler settings: bucket can restrict access to only cloudfront
    - Legacy access identities: Use cloud original access identity to access the s3 bucket.
  - Custom header
  - Origin shield

## Custom origin

- Custom domain
- Protocol: HTTP only / HTTPS only / Match viewer
- Minimum origin SSL protocol
- Add custom headers
  - To manage access control

# CloudFront Demo - Adding a CDN to a static Website < Rewatch

- Setup distribution S3/custom
- Origin path: Default path of origin. Blank = root of origin
- S3
  - public accessible or cloudfront only
- Viewer protocol: http/https, redirect http to https, https only.
- Allowed HTTP methods: GET / HEAD, GET / HEAD / OPTIONS, All methods
- Restrict viewer access: CloudFront signed urls / signed cookies
- Cache key & origin requests (Config TTL min/max/default; cache policy headers/query strings/cookies)
  - Cache policy & origin request policy (new)
  - Legacy cache settings (old)
- Price class: All edge locations / NA & Europe only / NA, EU, Asia, MiddleEast, Africa
- AWS WAF web ACL: Firewall integrated with CloudFront
- Default root object: e.g. index.html
- Invalidations
  - Create invalidation in the distribuion
  - Input object paths to invalidate
- Alternate domain name & SSL
  - Better if domain name present in route53 hosted zone
  - Request certificate in AWS Certificate Manager (ACM)
  - Input the domain and ceritifcate in CloudFront distribution
  - Go to route53, create routing, route the domain to CloudFront distribution

# CloudFront Security OAI and Custom Origins

- Origin Access Identity
  - Not using static website feature of S3
  - A type of identity
  - Assocaited with CloudFront Distributions
  - CloudFront becomes that OAI
  - That OAI can be used in S3 Bucket Policies
  - DENY all BUT one or more OAI's
- Custom origins
  - Custom headers
  - IP whitelist

# CloudFront Private behaviours Signed URLs & Signed cookies

- Public - Open Access to objects
- Private requests require Signed Cookie or URL
- 1 behaviour: Whole distribution public or private
- Multiple behiavours - each is public or private
- OLD - A CloudFront key is created by an Account Root User, The Account is added as a trusted signer
- NEW - Trusted key group(s) are added

## Signed URLs vs Cookies

- Signed URLs provides access to one object only
- Historically RTMP distributions couldn't use cookies
- Use URL's if client doesn't support cookies
- Cookies provides access to groups of objects
- Use for groups of files/all files of a type
- Or if maintaing application URL's is important

## Private distributions (behaviours)

- Paths: public & private
- Public path => Origin APIGW (Forward Cookies: All) => Lambda signer => set cookies to clients => private path => origin S3 (with OAI - Origin Access Identity configured, Forward Cookies: No) => return objects

# CloudFront using Origin Access Control - New version of OAI

- Edit distribution
- Select "Origin access control settings" option in "Origin access" setting
- Create control setting if there are no settings
- Choose sign requests by default to sign all requests
- Copy policy generated by the access control
- Go to s3 bucket, replace the current bucket policy with the new one
  + S3 only accepts from domain cloudfront.amazonaws.com and specific aws arn of the current distribution

# Lambda@edge

- Run lightweight lambda functions at edge locations
- Adjust data between the viewer and origin
- Supports Node.js and python
- Run in AWS Public Space (not vpc)
- Layers not supported
- Different limits vs normal lambda
  + 128MB
  + 5 seconds timeout
- Use cases
  + Perform A/B testing
  + Migration between S3 origins - origin request
  + Different objects based on device - origin request
  + Content by Country

# Global Accelerator

- Prevent connections jump through multiple hops
- Two anycast Ip addresses: 1.2.3.4, 4.3.2.1
- Allow single IP multiple locations
- Can be used for NON http/s (tcp/udp) vs cloudfront only HTTP/s
