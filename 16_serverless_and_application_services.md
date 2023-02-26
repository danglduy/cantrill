# Architecture Deepdive

Example: YouTube, allow users to upload videos and share videos to other users

User uploads video to youtube, videos multiple size: small -> large, small size -> big size

After receiving uploaded videos, Youtube needs to process the videos into different sizes: 4k -> 480p, 720p, 1080p

Architectures

- Monolithic: Upload, processing, store & manage components are handled inside a single application
  - Fails together
  - One thing breaks: all other things break
  - Scales together, cannot scale components independently
  - Bills together: system capacity needs to be ready for all components even though components are not handling anythings
- Tiered
  - Components are broken into tiers
  - Tiers can be on same servers or different servers
  - Each tiers connect to others via single endpoints
  - Tiers can be scaled vertically independently
  - Evolve:
    - Tiers connect via load balancers
    - Tiers can scale horizontally independently
    - Connections/requests between tiers need responds, synchornously
    - Each tier has to be running for the app to function
- Evoling with Queues

  - Upload complete -> push message to message queue
  - Processor are launched in auto scaling group based on the queue length -> get message from queue -> get uploaded data -> download file -> process file (create multiple versions of files) -> upload processed files to storage

- Microservice Architecture

  - Each service is a self-serviced application

- Event driven architecture
  - Producer: create event when something happens
  - Consumer: handle created event to take some actions
  - Component: can be producer or consumer
  - Best practice to handle complex applications: event router: HA, central access endpoint
    - Event bus: Events are added to event bus, event router pass events to consumers
  - Only consume resources when required (serverless)

# AWS Lambda

- Function as a service
- Lambda function: A piece of code that Lambda runs
- Functions use a runtime (exp: Python 3.8)
- Functions are loaded and run in a runtime environment
- The environment has a direct memory, based on the memory => cpu is implicitly allocated
- To be used as short running and focused application
- Billed on the duration that a function runs
- Key part of Serverless architectures
- Lambda function is a deployment service which Lambda executes
  - Languages
    - Official: Ruby, Python...
    - Other: Community-contributed languages using Lambda layers
- By default, everytime a Lambda function be executed, it's inside a brand new runtime environment.
- Memory: Define memory from 128MB to 10240MB, 1 mB steps
- CPU: Based on memory. 1vCPU/1769MB
- Storage: by default 512MB temporary storage at /tmp, up to 10240MB.
- Can run up to 900 seconds/15 minutes timeout.
- Role: Controlled by Execution Role (IAM Role assumed by Lambda function)

## Uses

- Serverless apps (S3, API Gateway, Lambda)
- File processing (S3, S3 Events, Lambda)
- Data triggers (dynamodb, streams, lambda)
- Serverless cron (eventbridge/cwevents + lambda)
- Realtime stream data process (kineses + lambda)

## Networking

- Public-mode
  - Access public space service / public internet
  - Best performance, not vpc networking required
  - Cannot access services running in VPC, without VPC configured to allow public access
- VPC-mode (Private)
  - Run in private subnet
  - Obey same rules anything run within VPC
  - Cannot access outside VPC unless configured. E.g. NAT gateway - requires EC2 network configuration via execution role
  - Used to have a disadvantage due to the design of how lambda attach to the VPC
    - Similar to Fargate, 2 VPCs: Lambda service VPC & destination VPC. 1 Lambda fucntion via each invocation will attach to 1 ENI in the destination VPC => take time & add delay to execution time of the lambda function; doesn't scale well
  - New way
    - Only 1 ENI required for functions within a region sharing same security group
    - Take 90 seconds initial setup

## Security

- Runtime environment
- Execution role
- Resource policy: Controls who can access the lambda function
  - Can't be changed using the UI, only CLI or API

## Logging

- Cloudwatch, cloudwatch logs & x-ray
- Cloudwatch Logs requires permissions via Execution Role

## Invocation

- Synchronous invocation
  - CLI/API invoke a lambda function, pass in data AND wait for a response
  - Clients -> API Gateway -> proxied to lambda function -> return response -> API gateway -> response back -> Clients
- Asynchronous invocation
  - Typically used whne AWS services invoke lambda function
    - E.g. S3 calls lambda but doesn't wait for responses. The event is generated and S3 stops tracking
  - Lambda is responsible for any reprocessing, retry number can be configured from 0 to 2.
    - Important: The Lambda function needs to be idempotent so a result should have the same end state
      - E.g. A function to process a thumbnail, then every time the function is executed the result should be same.
      - E.g. 2 a function to set balance of a account to a specific number, like $20, instead of relatively increasing the number with a mount.
  - Lambda can send events which it cannot process after retries: e.g. send events to a dead letter queue for diagnostics
  - Lambda supports destinations: SQS, SNS, Lambda & EventBridge where successful or failed events can be sent
- Event Source mappings
  - Typically used on streams or queues which don't support event generation (Kinesis, DynamoDB streams, SQS)
  - Event Source Mapping connects to a stream-based source (like AWS kinesis) via a long polling connection, receiving events splitted by "batch size" and sending the batch to the lambda. Lambda function handles the batch (need to care about the batch size because Lambda has 15-minute timeout duration, make sure Lambda can handle it).
  - Event source mapping can deliver failed events to DLQ (Dead letter queue) for diagnostics
- Note: Permission
  - Asynchronous: Lambda does not need permission to access the source unless Lambda needs to read additional data from the source
  - Event source mapping: Not delivering an event, but the event source mapping uses the Lambda Execution Role to read the source

## Versions

- Lambda functions have versions - v1, v2, v3
- A version contains the code + the configuration of the lambda function
- Immutable, nevers changes once published, has its own ARN (Amazon resource name)
- $Latest points to the latest version
- Support Aliases (DEV, STAGE, PROD..) point at a version and can be changed

## Startup times

An execution context is the environment where a lambda function runs in

- Cold start: Full creation and configuration including function code download
  - ~ 100ms
- Warm start: Same execution context is reused. Creation is skipped
  - ~ 1-2ms
- Provisioned concurrency: AWS create and keep X contexts warm and ready to use
- /tmp: shared between executions within a execution context. Usage: e.g. download temporary images to be shared between runs.

# CloudWatch Events and EventBridge

- CloudWatch Events: Stream of events generated by AWS products.
- EventBridge: to replace Cloudwatch Events, support third-party applications.
- Operate using a default entity (event bus)
- CloudWatch Events: only one event bus
- EventBridge can have additional busses
- Can create rules, match incoming rules => deliver events to a target
- Scheduled rule: match certain date/time or range of date/time
