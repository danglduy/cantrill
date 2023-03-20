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

# Demo Lambda function <= Rewatch

# Serverless Architecture <= Rewatch

- Not a single thing, but an architecture
- Software architecture, not hardware architecture
- Manage few servers -> low overhead
- Applications are a a collection of small & specialized functions
- Stateless and Ephermal environments - billing based on duration
- Event drive => consumption only when being used
- FaaS (Lambda) is used where possible
- Managed servcies are used where possible

# Simple Notification Service (SNS)

- Public AWS Service
- Coordinates the sending and delivery of messages
- Messages <= 256kb
- SNS Topics are base entity of SNS
- Publisher sends messages to TOPIC
- Topics have subscribers which receive messages:
  - HTTP, Email, SQS, Mobile push, SMS messages, Lambda
  - By default subscribers receive all messages. Can apply filter
  - Fan out (???)
- SNS used across AWS for notifications - CloudWatch & CloudFormation
- Delivery status (including HTTP, Lambda, SQS)
- Delivery retries - reliable delivery
- HA and scalable (region)
- Server Side Encryption (SSE)
- Cross-Account via Topic policy (Like s3 resource policy)

# AWS Step functions

Address limitations of lambda

- Small functions, 15-minute execution timeout
- Can be chained together, but messy at scale

## State machines

- Workflow: Start -> states -> end
- States are things which occur
- Maximum duration: 1 year
- Modes: standard & express workflows
- Can be started via services (lambda, api gateway...)
- Can be exported/imported templates
- IAM Roles is used for permissions

## States

- Succeed & Fail states
- Wait state: wait for a time, or until date/time
- Choice: Take different path based on input
- Parallel: parallel branches. A choice might create multiple sets of things at the same time
- Map: Accepts lists of thing
- Task: A single unit of work to perform action (Lambda, Batch,...)

# API Gateway

- Create and manage APIs
- Acts as endpoint/entrypoint for applications
- Sits between applications & integrations
  - Authorize, validate, transform request
  - Transform, prepare, return
- Highly available, scalable, handles authorizations, throttling, caching, cors, transformations, openapi spec, direct integration
- Can connect to services/endpoints in AWS or on-premises
- HTTP / REST / WebSocket APIs
- CloudWatch store, manage full stage request and response logs
- API Cache reduce number of calls made to backend integrations and improve client performance

## Endpoint types

- Edge-optimized
  - routed to the nearest cloudfront pop
- Regional
  - clients in the same region
- Private
  - Endpoint accessible only within a VPC via interface endpoint

## Stages

- APIs are deployed to stages, each stage one deployment
  - http://api.domain.com/prod => v1
  - http://api.domain.com/dev => v2
- Can be enabled for canary
  - https://docs.aws.amazon.com/apigateway/latest/developerguide/canary-release.html

# Advanced Demo - Build a serverless app

## Purpose

> The application will load from an S3 bucket and run in browser .. communicating with Lambda and Step functions via an API Gateway Endpoint. Using the application you will be able to configure reminders for 'pet cuddles' to be send using email and SMS.

## Tech

- SES: Sandbox mode - emails must be whitelisted to get received
- Create Lambda execution role & Lambda function
- State machine assume permission, invoke lambda function

# Simple Queue Service (SQS)

- Public, fully managed, high-available message queues
  - Standard / FIFO
    - Standard: Multi-lane highway
      - At least once delivery
      - Can scaling (by adding lanes)
    - FIFO: Single single-load road
      - Exactly once delivery
      - Limited performance: 3000 messages per second with batching, or 300 messages per second without batching
  - FIFO maintain order
  - Billed based on 'requests'
    - 1 request can contain 1-10 messages, up to 64KB total
    - More requests -> more costs
    - Short (immediate) vs Long (waitTimeSeconds) polling
    - Should use long polling. Short polling -> many requests, each request can even contains 0 messages
  - Encryption at rest and in-transit
  - Identity policy / queue policy
- Messages up to 256KB in size - link to large data
- Received messages are removed from queue (VisiblityTimeout), then put back to the queue if no response (retry by another client, failure case => ensure fault tolerence) or are explicitly deleted from the queue
- Dead-Letter queues can be used for problem messages
- Great for scaling: Auto scaling groups can scale based on the length of the queue, Lambdas can be invoked when messages appear on a queue
- SNS & SQS Fanout
  - Normal: when a video is uploaded, a message is pushed to the queue. Worker pool ASGs will get queue and process the video, resize into different sizes and put into the result bucket
  - Fanout: When a video is uploaded, a message is pushed to a SNS topic. Independent queues handling different sizes resizing are subscribers of the SNS topic. Each resizer for each size are independent queues, with independent scaling abilities

# SQS Standard vs FIFO

- FIFO: Single Lane Highway, 300 transactions/s

  - Normal: 1 message/transaction
  - High throughput mode: Batch, 10 messages/transaction
  - Guarantee order
  - Have to have .fifo suffix (?) to be a valid FIFO queue

- Standard: Multi Lane Highway
  - Scalable
  - Best efforts to maintain order
  - A message can be delivered more than once

# SQS Delay Queues

- Normal: A message is added to the queue and displayed immediately in the queue (SendMesage). When a consumer receives a message (ReceiveMessage), the message is hidden from the queue. After some seconds (Visibility Timeout), the message is reappeared automatically or explicitly deleted (DeleteMessage) => Allows for re-processing if there are any errors. Timeout setting: 0 seconds -> 12 hours, set on queue or per-message

- Delay: When a message is sent to the queue, the message is hidden in the queue automatically => ReceiveMessage got not results. The message is appeared on the queue after some seconds (DelaySeconds). Setting: 0 seconds -> 15 minutes, on queue. Per-message invisibility can be set to override the queue setting, but not supported on FIFO queues.

# SQS Dead-Letter Queues

- Handle problematic messages
- Each time a message re-appears on the queue after VisibilityTimeout, the message's ReceiveCount increments by 1.
- redrive policy: Specify the source Queue, the Dead-Letter Queue and conditions that will move messages to the dead letter queue (maxReceiveCount)
- When ReceiveCount > maxReceiveCount & messages are not explicitly deleted, messages are moved to the Dead-Letter queue
- Note: Retention policy: Original enqueue time is not adjusted the moving the message to the Dead-Letter Queue. Example: 2-day retention, 1 day already on the source queue => the message is deleted from the Dead-Letter queue after 1 day.

# Amazon Kinesis Data Streams < Rewatch

- Stream
- Data collected from producers, consumers get data
- Default: 24-hour window. Up to 365 days
- Shard architecture
- When scaling, new shards are added
- Shard capacity: 1MB/second ingestion, 2MBs/second consumption
- Kinesis Data Record: 1MB

## vs SQS

- Ingestion data: Kinesis
- SQS not persistent
- Huge scale ingestion
- Multiple consumers, rolling window
- Data ingestion, analytics, monitoring, app clicks...

# Amazon Kinesis Data Firehose

- Deliver data from Kinesis to destinations (e.g. s3, http splunk, redline, elasticsearch) to keep data from being removed after a time
- Scales automatically, region resilient
- Near-real time delivery (~ 60 seconds)
- Transform data using Lambda
- Billing - volume through firehose
- Firehose can receive data directly from producers

# Kinesis Data Analytics < rewatch

- Real time processing of data
- Using SQL
- Ingest from Kinesis Data Streams or Firehose

# Kinesis Video Streams

- Ingestive live video data from producers
- Security Cameras, smartphones...
- Consumders can access data frame-by-frame, or as needed
- Can persist and encrypt data
- Can't access directly via storage, only via APIs
- Integrates with other AWS services e.g. Reokognition and Connect

# Amazon Cognito

- Authentication, authorization, user management for web/mobile apps
- User pools - sign-in and get a json web token (JWT)
  - User directory management and profiles, sign-up, sign-in, MFA...
  - Social sign-in (facebook...)
- Identity Pools - Allow user to access temporary AWS credentials
  - Unauthenticated identities - guest users
  - Swap external identities - Google, Facebook, Twitter & User Pool for short term AWS credentials to access AWS resources
  - Assumes IAM role defined in Identity Pool and returns AWS credentials to access AWS Services

# AWS Glue

- Serverless ETL
- Move and transform data
- Crawls data sources and generate data catalog
- Sources: s3, rds, jdbc compatible & dynamodb
- Streams: Kinesis Data Stream & Apache kafka
- Targets: S3, RDS, JDBC Databases

## Data Catalog

- Persistent metadata
- One catalog per region per account

# Amazon MQ

- Combination of SQS and SNS
- Apache ActiveMQ
- Open standards: JMS, AMQP, MQTT, OpenWire and STOMP
- Provides queues and topics
- VPC based, not public like SQS, SNS
- No AWS native integration (logging, permissions, service integration...)

# Amazon AppFlow

- Transfer data between Software-as-a-Service (SaaS) applications like Salesforce, SAP, Zendesk, Slack, and ServiceNow, and AWS services like Amazon S3 and Amazon Redshift, in just a few clicks.
