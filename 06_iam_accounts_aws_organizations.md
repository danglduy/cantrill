# IAM Identity Policies

- Policy attached to an IAM Identity (User / Group / Role)
- JSON
- Grant / deny access to AWS Product
- Policy has many statements
- Authenticate
  - Authenticated identity
  - AWS checks policies
  - AWS gets statements
- Statement
  - Statement ID (SID) - Optional
  - Action: Conditional apply when doing some actions
    - 1 Action
    - Wild card
    - Multiple actions
  - Resource: Same as action (Use ARN)
    - 1 resource
    - Wildcard
    - Multiple
  - Effect: Allow / deny
- Statement overlap
  - Explicit deny -> Highest priority
  - Explicit allow
  - Implicit deny
- Inline vs managed policies
  - Inline: Attach to individual identity
  - Managed: A separated policy can be attached to multiple identity
- Usecase
  - Normal: Managed
    - AWS Managed policies - no need to care
    - Customer managed policies
  - Exception: Inline

# IAM Users and ARNs

- IAM user
  - Identity used for long term access
  - User -> User/Password // Access keys -> IAM: Authentication
  - IAM check IAM policy statements: Authorization
- ARNs: Amazon Resource Names
  - Uniquely identified resources within AWS accounts
  - Are used in IAM policies
  - Levels
    - Top level: ARN: arn::aws::s3::::catgifs
    - All ARN: arn::aws::s3::::catgifs/\*
      - => Not overlap with each other
  - Limits
    - 5000 IAM users per account
    - IAM user can be in 10 IAM groups
    - System design impact for application
      - Large scale || large organization
    - IAM roles & Identity Federation fix this

# IAM Groups

- Group of IAM users
- Cannot login
- No credential
- IAM user can be member of 10 IAM groups
- Deny - Allow - Deny
- No "all users" group
- No nested
- Max: 300 groups
- Can't be referenced in _resource_ policy

# IAM Roles

- One type of IAM Identity
- Uncertain users consume a role (can be an internal or an external user)
- Short term identity
- IAM role can attach IAM policy
  - Trust policy
    - Which Identity can assume that role
    - Generate temporary credentials (time limited)
    - Renew when expired
  - Permmissions policy
    - The temporary credentials can access resources
- Role allow users to access different AWS accounts

# When to use IAM Roles

- AWS Services operate on your behalf
  - E.g.: AWS Lambda -> no permission
- No need to hard code AWS Access Keys
  - Create IAM Role
  - Trust the Lambda service
  - Allow the Lambda to consume the role
  - Permissions Policy grant access AWS resources
  - sts: AssumeRole
  - Access cloudwatch, S3
- Emergency
  - Normally: Readonly
  - Emergency -> Use role -> Additional permission
- Multi environments
  - SSO
  - \> 5000 Identites
- Cross account access
  - Your account -> Partner account's role (Resource Policy) -> s3

# Service-linked roles & PassRole

- Role linked to a service
- Predefined by a service
- Can't delete the role until not required
- Service might create/delete
- Provide permission that interact with other resources
- Passrole: Pass a role from user to service

# AWS Organizations

- Manage multiple AWS Accounts
- AWS Account
  - Management Account (master)
  - Create Organizations
    - Create Organization Units
      - Create AWS Member Accounts
- Consolidated billing: Single billing for all member accounts
- Cheaper because bundle discount
- Service control policies
  - Restrict which AWS account can do
- Invite / Create new AWS Account
- Login to a AWS Account
  - Role switch to another acocunt using IAM roles
- Create
  - AWS Organizations -> Create organization
  - Org -> Invite via AWS Account ID or Email
  - Quota: Too many accounts
  - AWS orgs -> Invitation -> Accept
- Role switch
  - Create IAM Role (Target Account)
  - Trusted entity: AWS Account
    - Input source AWS Account
    - Select "Administrator Access"
    - Role name "Organization Account Access Role"
  - Source: Top navigation bar -> Role switch
    - -> Input Target Account ID & Target Role name

# Service control policies

- Restrict AWS Account in organization
- Org root
  - OU -> Accounts
  - Accounts
- Cannot restrict the management account
- Can restrict **identities** inside
- Permission boundary, but not grant permissions
- 2 options
  - Allow List
  - Deny List (_default_)
- Demo
  - Create policy
  - Attach to AWS Account

# CloudWatch Logs

- Public service - usable from AWS & on-premise
- Store, monitor & access logging data
- AWS Integration
- Generate metrics from logs

# CloudTrail

- 90 days by default
- Log API calls/activities
- Enabled by default
- Customize -> create a trail
- Events
  - Management events (create... resources)
  - Data events
    - Resource operations from inside
- By default only log management events
- Regional service
- Trail 1 region / All regions: 1 logical trail
- Global service events -> Need to enable IAM, STS, CloudFront
  - Region: us-east-1
- Trail store logs -> s3 -> compressed jsons
- Can create organizaitonal trail
- NOT realtime, delay < 15 minutes
- Cost
  - Free, 90 days, no s3
  - Paid
    - Management to S3: First trail free, $2 per 100000 events
    - Data event: $0.10/100000 events
    - Cloud Trail insights: $0.35/100000
- Trail
  - Name: 3..128 chars
  - SSE-KMS encrypt
  - Log file validation
  - Cloudwatch logs
  - Role

# AWS Control Tower

- Quick, easy setup multi-account environment
- Orchestrates other AWS Services
- Landing zone - multi-account environment
- SOS, centralized logging & auditing
