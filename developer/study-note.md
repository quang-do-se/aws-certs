
# Table of Contents

[Elastic Compute Cloud (EC2)](#Elastic-Compute-Cloud-(EC2))

[CloudFormation](#CloudFormation)

[Elastic Beanstalk](#Elastic-Beanstalk)

[DynamoDB](#DynamoDB)

[Cognito](#Cognito)

[Kinesis](#Kinesis)

[API Gateway](#API-Gateway)

[Secutity Token Service (STS)](#Security-Token-Service-(STS))

[Simple Queue Service (SQS)](#Simple-Queue-Service-(SQS))

[Simple Notification Service (SNS)](#Simple-Notification-Service-(SNS))

[Simple Storage Service (S3)](#Simple-Storage-Service-(S3))

[Serverless Application Model (SAM)](#Serverless-Application-Model-(SAM))

[Serverless Services](#Serverless-Services)

[Lambda](#Lambda)

[Step Functions](#Step-Functions)

[Identity and Access Management (IAM)](#Identity-and-Access-Management-(IAM))

[Route 53](#Route-53)

[CodeCommit](#CodeCommit)

[CodeBuild](#CodeBuild)

[CodeDeploy](#CodeDeploy)

[CodePipeline](#CodePipeline)

[CodeStar](#CodeStar)

[Key Management Service (KMS)](#Key-Management-Service-(KMS))

[Elastic Container Service (ECS)](#Elastic-Container-Service-(ECS))

[Elastic Container Registry (ECR)](#Elastic-Container-Registry-(ECR))

[CloudFront](#CloudFront)

[Elastic Block Storage (EBS)](#Elastic-Block-Storage-(EBS))

[X-Ray](#X-Ray)

[Relational Database Service (RDS)](#Relational-Database-Service-(RDS))

[CloudWatch](#CloudWatch)

[ElastiCache](#ElastiCache)

[Simple Work Flow (SWF)](#Simple-Work-Flow-(SWF))

[Others](#Others)

[Virtual Private Network (VPC)](#Virtual-Private-Network-(VPC))

## Elastic Compute Cloud (EC2)

- Instance type:

  - On Demand: predictable pricing, launch instantly

  - Reserved: long term commitment

  - Convertible Reserved: long term commitment but can change instance's attributes, e.g. instance family, instance type, scope, platform

  - Scheduled Reserved: launched within a specified window

  - Spot: cheap, unpredictable

  - Dedicated Instances: no other customers will share your hardware

  - Dedicated Hosts: book an entire physical server, control instance placement

  - Spot fleet: combination between on-demand and spot

- AMI is regional

- Application Load Balancer

  - Supports HTTP, HTTPS and WebSockets

  - Can edit `Listeners` rules to redirect to a different `Target Groups`

  - To capture true IP address of the client, use header **X-Forwarded-For**

  - **X-Forwarded-Port** for port and **X-Forwarded-Proto** for protocol

- Network Load Balancer

  - Supports TCP, UDP, TLS and WebSockets

  - For extreme performance

- Auto Scaling

  - EC2 Auto Scaling is limted to a certain number of default metrics such as CPU usage, Network In/Out

  - We must use `AWS Auto Scaling` for a custom metric (e.g. number of SQS messages) in `CloudWatch`. EC2 instance can `put-metric-data` to `CloudWatch`

  - Unhealthy instances will be terminated and a new instance will be deployed to replace it

- Monitoring

  - By default, EC2 instance publish metrics to CloudWatch every **5 minutes**

  - **Enable Detailed Monitoring** will publish metrics every **1 minute**. This is good if we have custom metrics for auto-scaling

  - Use `aws ec2 monitor-instances` to turn on detailed monitoring

  - Default metrics are: Network I/O, Disk I/O, CPU utilization

- Target Groups

  - Health checks: Load balancer des regular health checks to EC2 instances

- Metadata service

  - Call this URL on EC2 instance http://169.254.169.254/latest/meta-data/

  - Used to retrieve dynamic information such as instance-id, local-hostname, public-hostname, iam/security-credentials/role-name

  - There is NO policy in metadata

- Security Group

  - Each EC2 instance must have at least one security group and can have more than one

## CloudFormation

```
---
AWSTemplateFormatVersion: "version date"

Description:
  String

Metadata:
  template metadata

Parameters:
  set of parameters

Mappings:
  set of mappings

Conditions:
  set of conditions

Transform:
  set of transforms

Resources:
  set of resources

Outputs:
  set of outputs
```


- A **stack** is a running, deployed **template**

- There is no limit for number of templates

- Limit **200 stacks** per account. Create a case with Support Center to increase limit

- Sections:

  - Parameters (optional):

    - Does not allow conditions
    - `Intrinsic functions` can not be used within the `Parameters` section

  - Mappings:

    - 2-level mapping => mapName, key1, key2

    - Use `Fn::FindInMap: [ MapName, TopLevelKey, SecondLevelKey ]` to get value

    ```
    Mappings:
      RegionMap:
        us-east-1:
          HVM64: "ami-0ff8a91507f77f867"
          HVMG2: "ami-0a584ac55a7631c0c"
        us-west-1:
          HVM64: "ami-0bdb828fd58c52235"
          HVMG2: "ami-066ee5fd4a9ef77f1"
        eu-west-1:
          HVM64: "ami-047bb4163c506cd98"
          HVMG2: "ami-31c2f645"
        ap-southeast-1:
          HVM64: "ami-08569b978cc4dfa10"
          HVMG2: "ami-0be9df32ae9f92309"
        ap-northeast-1:
          HVM64: "ami-06cd52961ce9f0d85"
          HVMG2: "ami-053cdd503598e4a9d"

    Resources:
      myEC2Instance:
        Type: "AWS::EC2::Instance"
        Properties:
          ImageId: !FindInMap
            - RegionMap
            - !Ref 'AWS::Region'
            - HVM64
          InstanceType: m1.small
    ```

  - Resources (required):
    - Limit **200** per template

    - **Resources** section is the only **required** section

  - Conditions:

    - Can be used in `Resources` and `Output` sections

  - Outputs:

    - Limit **60** per template

    - Can be exported to another stack using **EXPORT**

      - Export `Name` must be unique in a region

  - Metadata: for description, sometimes code to call later

  - Parameters

    - Type: String, Number, List<Number>, CommaDelimitedList, AWS-specific Parameter Types, SSM Parameter Types

- Intrinsic Functions

  - `Fn::Base64 : <valueToEncode>`: encodes value to base64

  - `Fn::FindInMap: [ MapName, TopLevelKey, SecondLevelKey ]`: gets map value

  - `Fn::Cidr : [ ipBlock, count, cidrBits ]`: returns an array of CIDR address blocks

  - `Fn::GetAtt: [ logicalNameOfResource, attributeName ]`: returns value of attribute from a resource in the template

  - `Fn::GetAZs: <region>`: returns array of AZ in a specified region

  - `Fn::ImportValue: <sharedValueToImport>`: gets output value exported by another stack

  - `Fn::Join: [ delimiter, [ comma-delimited list of values ] ]`: concatenate list of values into a single string, e.g. `!Join [ ":", [ a, b, c ] ]` produces `a:b:c`

  - `Fn::Split: [ delimiter, source string ]`: splits string to a list

  - `Fn::Select: [ index, listOfObjects ]`: returns value at the index of the list starting with 0, e.g. `!Select [ "1", [ "apples", "grapes", "oranges", "mangoes" ] ]` returns `"grapes"`

  - `Fn::Sub: [ string, [varName: varValue...] ]`: substitutes or replaces variable in string with variable value

    - `{ "Fn::Sub": [ "www.${Domain}", { "Domain": {"Ref" : "RootDomainName" }} ]}`

    - `!Sub 'arn:aws:ec2:${AWS::Region}:${AWS::AccountId}:vpc/${vpc}'`

  - `!Transform { "Name" : macro name, "Parameters" : {key : value, ... } }`: specifies a macro to perform custom processing on part of a stack template. Very powerful, can modify the template

  - `Ref: <logicalName>`: return the value of the specified *parameter* or *resource*

    - Parameter: return the value of the specified parameter

    - Resource: return the physical ID of the resource

- Condition functions:

  - `Fn::And: [condition, ...]`

  - `Fn::Equals: [value_1, value_2]`

  - `Fn::If: [condition_name, value_if_true, value_if_false]`

  - `Fn::Not: [condition]`

  - `Fn::Or: [condition, ...]`

- Pseudo Parameters

  - `AWS::AccountId`: retunes AWS Account ID of the account that the stack is being created

  - `AWS::NotificationARNs`: returns a list of notification ARNs for the current stack

  - `AWS::NoValue`: uses with `Fn::If`. Removes corresponding resource property when specified as a return value in `Fn::If`

  - `AWS::Partition`: returns the partition that the resources is in, e.g. `aws` for standard regions, `aws-cn` for china region, `aws-us-gov` for AWS GovCloud

  - `AWS::Region`: returnes region for the stack, e.g. `us-west-2`

  - `AWS::StackId`: returns stack's ID

  - `AWS::StackName`: return stack's name

  - `AWS::suffix`: returns the suffix for a domain, e.g. `amazonaws.com` or `amazonaws.com.cn`

- CLI

  - Use `cloudformation package` and `cloudformation deploy` to deploy SAM

  - `sam package` and `sam deploy` are syntatic sugar for above

- **AWS CloudFormation StackSets** extends the functionality of stacks by enabling you to create, update, or delete stacks across **multiple accounts and regions** with a single operation

- Python helper script

  - **cfn-init**: Use to retrieve and interpret resource metadata, install packages, create files, and start services

  - **cfn-signal**: Use to signal with a CreationPolicy or WaitCondition, so you can synchronize other resources in the stack when the prerequisite resource or application is ready

  - **cfn-get-metadata**: Use to retrieve metadata for a resource or path to a specific key

  - **cfn-hup**: Use to check for updates to metadata and execute custom hooks when changes are detected

## Serverless Application Model (SAM)

- A CloudFormation contains a section `Transform: 'AWS::Serverless-2016-10-31'`

- Predefined resource types:

  - `AWS::Serverless::Function`: creates a Lambda function

  - `AWS::Serverless::Api`: creates an API GateWay

  - `AWS::Serverless::Application`: embeds a serverless application

  - `AWS::Serverless::SimpleTable`: creates a DynamoDB table

  - `AWS::Serverless::LayerVersion`: Creates a Lambda LayerVersion that contains library or runtime code needed by a Lambda Function

- Can include Lambda code in the package by specifying `CodeUri`

- AWS SAM CLI lets you locally build, test, and debug serverless applications that are defined by AWS SAM templates. The CLI provides a Lambda-like execution environment locally

## Elastic Beanstalk

- Customize the environment and create resources using **.config** file extension that you place in a directory named **.ebextensions**

- One environment includes one and only one application **version**

- One application can have multiple environments

- To avoid losing a resource using EB, defining a resource externally and using environment variables to reference it

- At most 1000 versions across all applications in a region

- `Lifecycle Policy`:

  - to phase out old application versions

  - Based on time or space

- Worker environment

  - Defines periodic task in a file **cron.yaml**

- Use `env.yaml` (Environment Manifest) to configure environment in the root of application source bundle

- `.elasticbeanstalk/config.yml` is EB CLI config

## DynamoDB

- Is a key-value and document NoSQL database with flexible schemas

- Enables event-driven programming with DynamoDB streams

  - Stream has 24 hours of data retention

- Partition key length: **2 KB**

- Sort key length: **1 KB**

- Max item size is **400 KB**

- **256** tables per AWS Region.

- Data is divided in partitions. Choosing an evenly distributed **partition key** is important

- Has **two** types of primary keys:

  - Partition (Hash) key: a simple primary key composed of one attribute

  - Partition key & sort key: composite primary key composed of two attributes: hash key and sort key

- Reading Data

  - Scan:

    - retrieve an entire table and then filter out data (inefficient)

    - projection-expression : attributes to retrieve

    - filter-expression : further filter results (client-side)

    - Support **Parallel Scan**

  - Query:

    - Retrieve data based on primary key values

    - **Can query based on LSI and GSI**

    - Pagination support

    - projection-expression : attributes to retrieve

    - filter-expression : further filter results (client-side)

    - Return up to 1 MB of data

  - GetItem: returns a set of attributes for the item with the given primary key

    - Must specify the **entire primary key**

    - projection-expression : attributes to retrieve

  - BatchGetItem: returns the attributes of one or more items from one or more tables

    - Returns up to 100 items

    - Up to 16 MB of data

    - projection-expression : attributes to retrieve

- `Global Secondary Indexes`:

  - **20** GSI max per table

  - effectively a "new table"

  - query over the entire table, across all partitions

  - Support **eventual consistency** only

- `Local Secondary Indexes`:

  - **5** LSI max per table

  - Same partition as base table

  - **Primary key must be composite**

  - Defined at table creation time

  - query over a single partition

  - Support **eventual consistency** and **strong consistency**

- Read/Write Capavity Unit

  - **(1 RCU = 2 entually consistent reads = 1 strongly consistent read = 1/2 transactional read request) per second for 4 KB**

  - **(1 WCU = 1 write = 1/2 transactional write request) per second for 1 KB**

- DynamoDB Stream can send changes in table to Lambda

  - Store this information up to 24 hours

- Time To Live (TTL): automatically delete an item after an expiry date/time

- Concurrency

  - Conditional writes

    - Helps with concurrent access

    - No performance impact (may improve performance since reduce round-trip checking)

  - Optimisic Locking / Concurreny

  - Does NOT suppport item locking or pessimistic locking

- **LimitExceededException**

  - If you want to create more than one table with secondary indexes, you must do so sequentially. For example, you would create the first table and wait for it to become ACTIVE, create the next table and wait for it to become ACTIVE, and so on. If you try to concurrently create more than one table with a secondary index, DynamoDB returns a LimitExceededException

- **UpdateItem** actually performs an **upsert**

- **Atomic Counter** is not idempotent

  - **Idempotent** means calling the same operation multiple times will give the same result

- **ReturnConsumedCapacity**: to return the number of write capacity units consumed by any of these operations
  
  - *TOTAL* — returns the total number of write capacity units consumed

  - *INDEXES* — returns the total number of write capacity units consumed, with subtotals for the table and any secondary indexes that were affected by the operation.

  - *NONE* — no write capacity details are returned. (This is the default.)

- Throttle Mitigating

  - **Increase the amount of read or write capacity for your table to anticipate short-term spikes or bursts in read or write operations**. If you decide later you don't need the additional capacity, decrease it. Take note that Before deciding on how much to increase read or write capacity, consider the best practices in designing your partition keys

  - **Implement error retries and exponential backoff**. This technique uses progressively longer waits between retries for consecutive error responses to help improve an application's reliability. If you're using an AWS SDK, this logic is built‑in. If you're using another SDK, consider implementing it manually

  - **Distribute your read operations and write operations (write sharding aka adding suffixes to partition key) as evenly as possible across your table**. A "hot" partition can degrade the overall performance of your table

  - **Implement a caching solution, such as DynamoDB Accelerator (DAX) or Amazon ElastiCache**. DAX is a DynamoDB-compatible caching service that offers fast in‑memory performance for your application. If your workload is mostly read access to static data, query results can often be served more quickly from a well‑designed cache than from a database

## Cognito

- User Pool

  - User database (username or email/password)

  - User can sign-in via other identity providers such as Facebook, Google...

  - Multi-factor authentication (MFA) supported

  - Return a **JSON Web Tokens (JWT)**

  - Can integrated with API Gateway for authentication

- Identity Pool

  - Users can sign-in via Federated Identity providers such as Cognito User Pool, Facebook, Google or Amazon

  - After a successful authentication, returning a **STS token** to use as a temporary AWS credential

  - AWS credentical will allow access based on IAM permission of that credential

  - Support **unauthenticated identities**

  - This feature also supports Developer Authenticated Identities (Identity Pools), which lets you register and authenticate users via your own back-end authentication process

- User Pool vs Identity Pool

  - User Pool is mostly used to authenticate user. (Who is the user?)

  - Identity Pool is not only authenticating user, but also returning temporary access credentials (access key, secret key, and session token). It is done by contacting STS. (Who is the user and what can that user do?)

  - User Pool returns **JWT** while Identity Pool returns **STS**

- Custom identity broker

  - Only build one when your identity store is not compatible with SAML 2.0

- AppSync

  - Cross-device syncing of application-related user data

  - Offline capability

  - Requires Federated Identity Pool in Cognito

## Kinesis

- Kinesis Data Streams (KDS) is a massively scalable and durable **real-time** data streaming service

- Kinesis Stream needs capacity provision

- Kinesis Firehorse supports auto scaling

- Kinesis Stream:

  - Data retention is 1 day by default, max 7 days

  - Shards:

    - 1 MB/s at write per shard

    - 2 MB/s at read per shard

    - Recommends **Batching** with PutRecords to reduce cost and increase throughput

    - **Records are ordered per shard**

  - **ProvisionedThroughputExceeded**:

    - When sending more data which exceeds MB/s or TPS per shard

    - Makes sure there is not HOT shard (most data goes to a single shard due to bad partition key)

    - Solution:

      - Retries with backoff

      - Increase shards

      - Ensure partition key is uniform distributed

  - Kinesis Client Library (KCL)

    - Java library to read records from Kinesis Streams

    - Rule: each shard is be read by only once KCL instance

    - Records are read in order at the shard level

## Simple Queue Service (SQS)

- Serverless, autoscaling

- Supports Server-Side Encryption

- Max message size is **256 KB**

- SQS consumer can receive up to 10 messages per call

- Message retention period is between 1 minutes and 14 days

- To send more than 256 KB, use **Java SDK Extended Client** which sends large message to S3 and metadata to SQS

- **FIFO** features: deduplication, sequencing

  - **Message Group Id**:

    - Strict messages order within the same group Id

  - **Message Deduplication Id**:

    - By default, this must be specified when creating a message

    - Any messages with the same deduplication id will be received sucessfully but will NOT be delivered during the 5 minute deduplication interval

    - Content-Based Deduplication:

      - Automaticaly generates SHA-256 hash of message body as a **Message Deduplication Id**

      - Helps avoid duplicate message content in the queue
      
  - 300 messages per second per API action without patching

  - 3000 messages with batching

- **No limit** for how many messages can be stored in the queue

- Standard queue provides nearly unlimited throughput

- `ReceiveMessageWaitTimeSeconds` AKA **Long Polling**, max 20 seconds



- Important APIs: CreateQueue, DeleteQueue, **PurgeQueue**, SendMessage (batchable), ReceiveMessage, DeleteMessage (batchable), ChangeMessageVisibility (batchable)

- **Visibility Timeout**

  - Default: 30 seconds

  - Max: 12 hours

  - After a message is received, it will be invisible to other receivers. This is to prevent the same message from being processed multiple times

- ChangeMessageVisibility API: allows to change message visibility while processing a message. For example, if a message is taking too long to process, we can extend the message visibility timeout by calling this API

> When a message handler receive a queue message, that message will not be available to other handlers within **Default Visibility Timeout** period. If the handler is taking longer to process a certain message, it can call **ChangeMessageVisibility** API to extend the visiblity timeout. After the handler finishes processing the message, it will call **DeleteMessage** API to remove the message from the queue

> When a handler failed to process a message, that message goes back to the queue. We can set **Maximum Receives** threshold to specify failed messages will go into **Dead Letter Queue** after exceeding the receiving threshold. To turn on **Dead Letter Queue**, makes sure to enable **Redrive Policy** in the setting

## Simple Notification Service (SNS)

- Fan-out architecture pattern:

  - data -> SNS => SQS, Lambda, HTTP endpoint...

  - Push once in SNS, receive in many SQS

  - Basic of Pub/Sub model

- Up to **100,000** topics

- Up to **10,000,000** subscriptions per topic

- Subscribers can be:

  - Lambda

  - SQS

  - HTTP/HTTPS (with delivery retries)

  - Emails

  - SMS messages

  - Mobile Notifications

- Message text can be customized for each endpoint

## Simple Storage Service (S3)

- Size limit as of 2019

  - Multi-part upload is **5 TB**

    - Recommends using multipart upload if size > **100 MB**

    - Take advantages of retries if any part fails

  - A single PUT call is **5 GB**

- **100** buckets per account (can request AWS to increase the limit)

- Serverless, auto scaling

- S3 is a **global** service. <b style='color:red'>However</b>, each bucket is **regional** and it must have a **globally unique name**

- Unlimited storage for each bucket

- Bucket naming convention:

  - No UPPERCASE

  - No underscore

  - 3-63 characters (**us-east-1 allows 256 characters**)

  - not an IP

  - Begin with lowercase letter or number

  - Hyphen and period are ok. Double hyphen or period are invalid

  - For DNS bucket name, no period allowed

- Encryption for Objects:

  - SSE-S3: encrypts using keys handled & managed by AWS

    - Server side

    - Must set header: `x-amz-server-side-encryption": "AES256"`

  - SSE-KMS: encrypts using keys handled & managed by KMS

    - Server side

    - Must set header: `x-amz-server-side-encryption": "aws:kms"`

    - Use default key if not specifying `x-amz-server-side-encryption-aws-kms-key-id`

  - SSE-C: encrypts using keys handled & managed by Customer

    - Server side

    - **HTTPS** must be used

    - Encryption key must be in HTTPS header per request

  - Client Side Encryption

    - Client encrypts the data before sending to S3

- Bucket policy is JSON-based and similar to IAM permission policy

  - Applies at bucket level, NOT object-level (use ACL instead to manage permissions of S3 objects)

  - To allow access only for only requests that originate from specific webpages, use **aws:Referer**

- Can send notifications on changes to Lambda, SQS and SNS

- Pre-Signed URLs

  - SDK can generate Pre-Signed URLs for a S3 object

  - URL will be valid by specified time limit

  - Allows temporary access to a S3 object

  - Each URLs will have a single action permission: either GET or PUT

- Static HTML hosting

  - If we get HTTP response code 403 (Access Denied), we need to create a bucket policy allow public access

  - Allows CORS if there is cross-origin request between buckets

    - Route 53 alias to S3 website endpoint is the same as calling it directly so no CORS needed

    - A CORS configuration is an XML file that contains a series of rules within a `<CORSRule>`

- **Consistency Model**

  - Read after write consistency for PUTS of new objects

  - Eventual consistency for DELETES and PUTS for existing objects

  - Update to the same record is decided with latest timestamp

  - This is due to high availability model which replicate data across multiple servers

- Bucket URL

  - `<bucket-name>.s3-website-<region>.amazonaws.com`

- S3 is not part of the VPC

- **Cross-region replication** feature in S3, the following items should be met:

  - The source and destination buckets must have **versioning** enabled

  - The source and destination buckets must be in different AWS Regions

  - Amazon S3 must have permissions to replicate objects from that source bucket to the destination bucket on your behalf

- **S3 Transfer Acceleration** enables fast, easy, and secure transfers of files over long distances between your client and your Amazon S3 bucket

  - Transfer Acceleration leverages **Amazon CloudFront’s** globally distributed AWS Edge Locations. As data arrives at an AWS Edge Location, data is routed to your Amazon S3 bucket over an optimized network path

## Serverless Services

- Compute

  - Lambda

  - Lambda@Edge

  - Fargate

- Storage

  - Simple Storage Service

  - Elastic File System

- Data Stores

  - DynamoDB

  - Aurora Serverless

- API Proxy

  - API Gateway

- Application Integration

  - SNS

  - SQS

  - AppSync

  - EventBridge

- Orchestration

  - Step Functions

- Analytics

  - Kinesis

  - Athena

- Developer tooling

  - Framework: Serverless Application Model (SAM)

  - CI/CD: Codestar, CodePipeline, CodeBuild, CodeDeploy

  - Monitoring, Logging and Diagnostics: X-Ray, CloudWatch

  - Authoring and development: SAM Local, Cloud9

## Lambda

- Memory:

  - Assign memory to improve performance

  - Lambda allocates CPU power proportional to the memory

  - Max 3008 MB or 3 GB

- Timeout

  - Default: 3 seconds

  - Max: 900 seconds or 15 minutes

  - May impact cost

- Error Handling

  - Integrate with Dead Letter Queue, SNS and SQS

- Environment variables

  - No limit but total size must be under 4 KB

- Version

  - Lambda version can be specify in the function name, e.g. `HelloWorld:$LATEST` in ARN

  - API Gateway does NOT support Lambda version. See Alias

- Alias

  - Alias is useful to integrate with API Gateway such as each API stage can point to a different alias

  - Like version, we can specify Alias in Lambda ARN

  - **Can support Blue/Green versioning** in Alias - shift traffic between two versions based on assigned weights (%)

- Can integrate with various services using SDK

  - e.g. automatically call CodeDeploy to deploy a new source code stored in S3

- Asynchronous

  - Lambda sents the event to a queue

  - Lambda will try 3 times before sending the event to DLQ

- Deployment package

  - **50 MB** (zipped, for direct upload)

  - **250 MB** (unzipped, including layers)

  - /tmp directory **512 MB**

  - Can upload the zip file directly if under 50 MB, otherwise use S3

- **Execution Context**

  - This is a temporary runtime environment that initializes any external dependencies of your Lambda function code, such as database connections or HTTP endpoints
  
  - Take advantage of Execution Context reuse to improve the performance of your function

- Errors

  - **CodeStorageExceededException** (400): You have exceeded your maximum total code size per account

  - **InvalidParameterValueException** (400): One of the parameters in the request is invalid. For example, if you provided an IAM role for AWS Lambda to assume in the CreateFunction or the UpdateFunctionConfiguration API, that AWS Lambda is unable to assume you will get this exception

  - **ResourceConflictException** (409): The resource already exists

  - **ResourceNotFoundException** (404): The resource (for example, a Lambda function or access policy statement) specified in the request does not exist

  - **ServiceException** (500): The AWS Lambda service encountered an internal error

  - **TooManyRequestsException** (429): Request throughput limit exceeded

- Concurrency

  - Default limit: 1000 across all functions within a given region

  - `concurrent executions = (invocations per second) x (average execution duration in seconds)`

  - AWS Lambda will keep the unreserved concurrency pool at a minimum of **100** concurrent executions

  - For Lambda functions that process Kinesis or DynamoDB streams, **the number of shards is the unit of concurrency**. If your stream has 100 active shards, there will be at most 100 Lambda function invocations running concurrently. This is because Lambda processes each shard’s events in sequence and in batch. Remember that the Kinesis and Lambda integration is using a **poll-based event source**, which means that the number of shards is the unit of concurrency for the function. AWS Lambda polls the stream and invokes your Lambda function synchronously when it detects new stream records

- InvocationType (Invoke API)

  - `RequestResponse` (default): synchronous

  - `Event`: asynchronous - send the **event** to queue

  - `DryRun`: validate parameter values and verify the user or role has permission to invoke the function

- X-Ray environment variables

  - _X_AMZN_TRACE_ID

  - AWS_XRAY_CONTEXT_MISSING

  - AWS_XRAY_DAEMON_ADDRESS

- **5 Layers** which is a ZIP archive that contains library, custom runtime, and other dependencies  

- For Node.js, Python, and Ruby functions, you can develop your function code in the **Lambda console** as long as you keep your deployment package under **3 MB**

- VPC

  - You can put Lambda in a VPC to access AWS services in private subnets such as RDS

  - If Lambda needs to access public AWS services or resources on the Internet, create a **NAT**

  - Makes sure to configure the correct security group

## Step Functions

- Language: JSON

- Can be provisioned by CloudFormation

- Max input and output between each state is 32 KB

- States:

  - Pass
  - Task
  - Choice
  - Wait
  - Succeed
  - Fail
  - Parallel
  - Map

## Identity and Access Management (IAM)

- An explicit deny has higher priority than an explicit allow

- Dynamic Policy:

  - We can use policy variable `${aws:username}` to assign different resource for each user

  - `"Resource": ["arn:aws:s3:::my-company/home/${aws:username}/*"]`

- Policy

  - Effect: Allow or Deny

  - Action: Include a list of actions that the policy allows or denies (effect)

  - Resource: a list of resources to which the actions apply

  - Principle:

    - The `Principal` element specifies the user, account, service, or other entity that is allowed or denied access to a resource

    - Mostly for resource-based policy, which attached to services such as SQS, S3, KMS. These services have their own policy management system with same language as IAM

    - Must use ARN to specify a principal:

      - <code>"arn:aws:iam::**AWS-account-ID**:root"</code>

      - <code>"arn:aws:iam::**AWS-account-ID**:user/**user-name**"</code>

        - Cannot use a wildcard (*) to mean "all users"

    - Do not use the Principal element in policies that you attach to IAM users, groups and roles. In those cases, the principal is implicitly the user that the policy is attached to (for IAM users) or the user who assumes the role (for role access policies)

  - Condition: Specify the circumstances under which the policy grants permission

    - `"Condition": {"Bool": {"aws:MultiFactorAuthPresent": "true"}`

  - Managed Policies vs Inline Policies

    - Managed Policies are preferred

    - Inline policy is tightly coupled to user or role

  - ARN format

    - **arn:*partition*:*service*:*region*:*account-id*:*resource-id***

    - **arn:*partition*:*service*:*region*:*account-id*:*resource-type*/*resource-id***

    - **arn:*partition*:*service*:*region*:*account-id*:*resource-type*:*resource-id***

    - `https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html`

  - For **on-premises** programatic access, we must create credentials file at `~/.aws/credentials` with access keys (access key ID and secrec access key)

  - Sign-in page URL

    - `https://<account_id>.signin.aws.amazon.com/console/`

    - `https://<alias>.signin.aws.amazon.com/console/`

  - For applications running outside of AWS such as on-premises and third-party automation, it will need access keys, NOT IAM Role

## Security Token Service (STS)

- Global service

- Credentials are returned as Security Token, Access Key ID, Secret Access Key

- Supported Actions:

  - AssumeRole

    - First, add an IAM policy to allow assume a certain role

    - Second, create a role with `trust relationship`

      - `Trust relationship` defines who can assume the role as `Principle`. If principle is `root`, all IAM users under that account can assume that role

      - https://aws.amazon.com/premiumsupport/knowledge-center/iam-assume-role-cli/

      - Supports MFA

      - Supports Session Policy

  - AssumeRoleWithSAML

  - AssumeRoleWithWebIdentity

  - DecodeAuthorizationMessage

  - GetAccessKeyInfo

  - GetCallerIdentity

  - **GetFederationToken** returns temporary security credentials for a federated user

    - Does NOT support MFA

    - Supports Session Policy

  - **GetSessionToken** returns a set of temporary security credentials to an existing IAM user
  
    - Supports MFA

    - Does NOT support Session Policy

## Route 53

- Record type:

  - A: URL to IPv4

  - AAAA: URL to IPv6

  - CNAME: URL to URL

  - Alias (AWS specific): URL to AWS resource

    - Alias is created under an A type record

- Prefer Alias over CNAME for AWS resources (performance)

- Routing policy:

  - simple: no rule

  - failover: based on health of the instance

  - geolocation: based on user's location

  - geoproximity: based on proximity of resource and user's location

  - latency: chooses the lowest latency resource

  - weighted: is used for Blue/Green deployment to test a new version of application

## CodeCommit

- Notifications: CloudWatch Event Rules with SNS

  - Mostly for comments and merge/pull requests

- Triggers: Lambda or SNS

  - Primary for actions on a branch

- Data in AWS CodeCommit repositories is encrypted in transit and at rest

## CodeBuild

- Autoscaling up to 20 concurrent build

- `buildspec.yml` contains build instructions

- Can be reproduced locally for troubleshooting

- **Phases**: `install`, `pre_build`, `build`, `post_build`

```
version: 0.2

env:
  variables:
    JAVA_HOME: "/usr/lib/jvm/java-8-openjdk-amd64"
  parameter-store:
    LOGIN_PASSWORD: /CodeBuild/dockerLoginPassword

phases:
  install:
    commands:
      - echo Entered the install phase...
      - apt-get update -y
      - apt-get install -y maven
    finally:
      - echo This always runs even if the update or install command fails
  pre_build:
    commands:
      - echo Entered the pre_build phase...
      - docker login –u User –p $LOGIN_PASSWORD
    finally:
      - echo This always runs even if the login command fails
  build:
    commands:
      - echo Entered the build phase...
      - echo Build started on `date`
      - mvn install
    finally:
      - echo This always runs even if the install command fails
  post_build:
    commands:
      - echo Entered the post_build phase...
      - echo Build completed on `date`
artifacts:
  files:
    - target/messageUtil-1.0.jar
  discard-paths: yes
  secondary-artifacts:
    artifact1:
      files:
        - target/messageUtil-1.0.jar
      discard-paths: yes
    artifact2:
      files:
        - target/messageUtil-1.0.jar
      discard-paths: yes
cache:
  paths:
    - '/root/.m2/**/*'
```

## CodeDeploy

- CodeDeploy is a deployment service that automates application deployments to Amazon EC2 instances, on-premises instances, serverless Lambda functions, or Amazon ECS services.

- Each EC2 must run CodeDeploy agent

- Deployment choices:

  - Supports only **In-place** and **Blue/Green** for Application

  - Only deployments that use the **EC2/On-Premises** compute platform can use **in-place** deployments

  - **Blue/Green** does not work with On-Premises

  - Supports **All at once**, **Canary** and **Linear** for Lambda (Blue/Green only)

  - All AWS Lambda compute platform deployments are blue/green deployments

  - **AWS Lambda** compute platform deployments CANNOT use an **in-place** deployment type

- **CodeDeploy agent** is NOT required for deployments that use the Amazon ECS or AWS Lambda compute platform. Required only for EC2/On-Premises compute platform

- Supports EC2 Auto Scaling Group

- More flexible than Elastice Beanstalk

- Application revision: application code + appspec.yml file

- Each deployment app can have multiple deployment groups

  - We can group DEV, TEST and PROD to different deployment groups

  - Each deployment group can be used to deploy to EC2, ECS and Lambda

- Rollback

  - Re-deploy a previous good version to a failed instance (NOT A NEW INSTANCE)

- appspec.yml

  - files: This section is required only if you are copying files from your revision to locations on the instance during deployment. For instance, we need to copy `/index.html` in source directory to `/var/www/html`

  - hooks: Can include scripts for different stages of deployment. The stage order is:

    - ApplicationStop

    - DownloadBundle

    - BeforeInstall

    - AfterInstall

    - ApplicationStart

    - ValidateService

```
version: 0.0
os: linux
files:
  - source: /
    destination: /var/www/html/WordPress
hooks:
  BeforeInstall:
    - location: scripts/install_dependencies.sh
      timeout: 300
      runas: root
  AfterInstall:
    - location: scripts/change_permissions.sh
      timeout: 300
      runas: root
  ApplicationStart:
    - location: scripts/start_server.sh
    - location: scripts/create_test_db.sh
      timeout: 300
      runas: root
  ApplicationStop:
    - location: scripts/stop_server.sh
      timeout: 300
      runas: root
```

## CodePipeline

## CodeStar

## Key Management Service (KMS)

- Up to **4 KB** per call

- Supports Key Policy

- AWS KMS only supports **Symmetric Encryption**

- Use **CloudHSM** if you want to manage your own HSMs and have asymmetric encryption

- Using **envelop encryption** if data > 4 KB

  - **Data key** and **encrypted data key** is generated by KMS and sent to client

  - **Encrypted data key** is encrypted by CMK

  - Use API **GenerateDataKey**

  - Client encrypts plain text with **data key** with preferred encryption algorithm and remove **data key**

  - Cipher text and encrypted data key are sent together as encrypted package

  - To decrypt, we ask KMS to decrypt the **encrypted data key** and decrypt using above preferred encryption algorithm

- Advantages: user control and audit trail

## API Gateway

- Stage: dev, test or prod...

- Deployment: snapshot of resources and methods

- Integration steps:

  - Method Request

  - Integration Request

  - Integration Response

  - Method Response

- Usage Plan:

  - Generate API key for other user to use your APIs

  - Control request rate and quota for each API user

  - Associate with API stages

  - Ability to track usage for each API key

- Mapping Template:

  - Renaming parameter

  - Modifying body content

  - Add headers

  - Filter output results

  - Able to map JSON to XML

  - Can be added during **Integration** phases of Request and Response

  - Use `Velocity Template Language`

- Cognito User Pools

  - Only authentication, NOT authorization

  - User management can be fully managed

  - Return **JSON Web Token**

- Supports authentication using *request header* signing with AWS Signature Version 4

  - Sigv4 is created by user's access key (access key ID and secrec access key) and therefore is authorized by IAM

  - For both authentication and authorization (with IAM)

  - Simpler to implement than Lambda Authorizer and Cognito User Pool. However, must grant permission at IAM level, does not work with 3rd party and hard to manage users

  ```
  Authorization: AWS4-HMAC-SHA256
  Credential=AKIAIOSFODNN7EXAMPLE/20130524/us-east-1/s3/aws4_request,
  SignedHeaders=host;range;x-amz-date,
  Signature=fe5f80f77d5fa3beca038a248ff027d0445342fe2855ddc963176630326f1024
  ```

- Lambda Authorizer

  - Supports `token-based` or `request parameter-based`

  - For authentication but also returning authorization

  - Returns an **IAM policy** along with caller's principal identifier for authorization purpose

  - Flexible and customizable (e.g. allows OAuth/SAML/3rd-party authentication method)

  - There are two types of Lambda authorizers:

    - **A token-based Lambda authorizer** (also called a TOKEN authorizer) receives the caller's identity in a bearer token, such as a JSON Web Token (JWT) or an OAuth token

    - **A request parameter-based Lambda authorizer** (also called a REQUEST authorizer) receives the caller's identity in a combination of headers, query string parameters, stageVariables, and $context variables

- Integration type

  - For the Lambda **proxy** integration, the value is **AWS_PROXY**
  
    - For Lambda proxy integration, Lambda must return the result in JSON format

    - In Lambda proxy integration, API Gateway maps the entire client request to the input event parameter of the backend Lambda function as follows

    ```
    {
    "resource": "Resource path",
    "path": "Path parameter",
    "httpMethod": "Incoming request's method name"
    "headers": {String containing incoming request headers}
    "multiValueHeaders": {List of strings containing incoming request headers}
    "queryStringParameters": {query string parameters }
    "multiValueQueryStringParameters": {List of query string parameters}
    "pathParameters":  {path parameters}
    "stageVariables": {Applicable stage variables}
    "requestContext": {Request context, including authorizer-returned key-value pairs}
    "body": "A JSON string of the request payload."
    "isBase64Encoded": "A boolean flag to indicate if the applicable request payload is Base64-encode"
    }
    ```

  - For the Lambda **custom** integration and all other AWS integrations, it is **AWS**
  
  - **HTTP** is only used for HTTP custom integration where you need to specify how the incoming request data is mapped to the integration request and how the resulting integration response data is mapped to the method response

  - **HTTP_PROXY** is for simple setup, You only need to set the HTTP method and the HTTP endpoint URI, according to the backend requirements, if you are not concerned with content encoding or caching

- Canary deployment

  - Blue/green deployment

  - Allows a single stage to have 2 different versions running

- **Integration Timeout**

  - The range is from 50 milliseconds to 29 seconds for all integration types, including Lambda, Lambda proxy, HTTP, HTTP proxy, and AWS integrations

- **Cache Invalidation**

  - Use header **Cache-Control: max-age=0**

- All of the APIs created with Amazon API Gateway expose **HTTPS** endpoints only

- **Endpoint URL**

  - `https://{restapi_id}.execute-api.{region}.amazonaws.com/{stage_name}/`

- Available Metrics

  - Monitor the **IntegrationLatency** metrics to measure the responsiveness of the backend

  - Monitor the **Latency** metrics to measure the overall responsiveness of your API calls

  - Monitor the **CacheHitCount** and **CacheMissCount** metrics to optimize cache capacities to achieve a desired performance. CacheMissCount tracks the number of requests served from the backend in a given period, when API caching is enabled. On the other hand, CacheHitCount track the number of requests served from the API cache in a given period

## Elastic Container Service (ECS)

- **Task definition** are metadata in JSON form to tell ECS how to run a Docker container

  - Port mappings are specified as part of container definition in the task definition

- **Services** help define how many tasks should run and how they should be run

- Best practice: Assigns a separated IAM role for each task definition

  - A container can only retrieve credentials for the IAM role that is defined in the task definition to which it belongs; a container never has access to credentials that are intended for another container that belongs to another task

- Application Load Balancer

  - Allows containers to use dynamic host port mapping. For example, if we have multiple containers listening on port 80 on the same instance, dynamic host port mapping will map a different port for each container 8080 -> 80, 8081 -> 80, 8082 -> 80...

- Agent Configuration `/etc/ecs/ecs.config`

  - `ECS_CLUSTER`: Assigns EC2 to an cluster

  - `ECS_ENGINE_AUTH_DATA`: To pull images from private registries

  - `ECS_AVAILABLE_LOGGING_DRIVERS`: CloudWatch container logging

  - `ECS_ENABLE_TASK_IAM_ROLE`: Enable IAM roles for ECS tasks

- Task Placement Strategies

  - binpack: least available amount of CPU or memory

  - random: randomly

  - spread: evenly

- **Cluster queries** are expressions that enable you to group objects

- **A task placement constraint** is a rule that is considered during task placement

- **Container Instance IAM Role** only applies if you are using the EC2 launch type, NOT Fargate

- **A service-linked role** is a unique type of IAM role that is linked directly to Amazon ECS itself and not on the ECS task. Service-linked roles are **predefined** by Amazon ECS and include all the permissions that the service requires to call other AWS services on your behalf

## Elastic Container Registry (ECR)

- If there is an authorization issue, check IAM permission for `ecr:GetAuthorizationToken` for push and pull images

- Use command `aws ecr get-login --no-include-email` to login to ECR

## CloudFront

- Supports **HTTP/HTTPS** and **Adobe Real-Time Messaging Protocol (RTMP)**

- Has no instrinsic routing support. Instead, combines with Route 53

- HTTPS support: Between clients and CloudFront and CloudFront and backend

- Uses *Signed URLs* and *Signed Cookies* to serve private content

- HTTPS

  - Requiring HTTPS for Communication Between Viewers and CloudFront: **Viewer Protocol Policy**

    - `Redirect HTTP to HTTPS`, or `HTTPS Only`

  - Requiring HTTPS for Communication Between CloudFront and Your Custom Origin: **Origin Protocol Policy**

  - Only requires SSL certificate between the origin and CloudFront

- To restrict access to content that you serve from Amazon S3 buckets, you create CloudFront **signed URLs** or **signed cookies** to limit access to files in your Amazon S3 bucket, and then you create a special CloudFront user called an **origin access identity (OAI)** and associate it with your distribution. Then you configure permissions so that CloudFront can use the OAI to access and serve files to your users, but users can't use a direct URL to the S3 bucket to access a file there

  - Higher latency than **Lambda@Edge with Cognito**

## Elastic Block Storage (EBS)

- EBS volumes are locked at AZ level (probably due to performance & latency since EBS is network drive, not physical drive)

- To move EBS to another AZ, first take a snapshot and recreate it in the other AZ

- Encryption

  - All data encrypted between the instance and the volume

  - All snapshots are encrypted

  - All volumns created from the snapshot are encrypted

  - Minimal impact on latency and performance

- New volumes are raw block devices and do not contain any partition or file system. You need to login to the instance and then **format** the EBS volume with a file system, and then **mount** the volume for it to be usable

- Use the `sudo file -s device` command to list down the information about your volume, such as file system type

- Detach

  - You can detach an Amazon EBS volume from an instance explicitly or by terminating the instance. However, if the instance is running, you must first unmount the volume from the instance

  - If an EBS volume is the **root** device of an instance, you must **STOP** the instance before you can detach the volume. You can’t unmount the root volume on a running instance

  - Stop Instance = Forced Unmount

## X-Ray

- Helps understand dependencies in a microservices architecture

- Helps troubleshoot performance

- Helps pinpoint service issues and finds errors/exceptions

- Daemon listens on UDP port 2000

  - You only have to install the X-Ray daemon if you are using Elastic Beanstalk, ECS or EC2 instances

- Sampling: specifies the percent of requests send to X-ray which could help reduce cost

  - `Total Sampling Request = = reservoir size + ( (incoming requests per second - reservoir size) * fixed rate)`

  - The fixed rate is a decimal between 0 and 1.00 (100%)

- Segments: Each application/service will send them

  - Your application can record data about the work that it does itself in **segments** or work that uses downstream services and resources in **subsegments**

- Trace: a combination of segments to form end-to-end trace

- Annotations: key/value pair used to **index** traces and use with **filters**

- Metadata: key/value pair for extra information in the segment

- For Fargate:

  - Deploy X-Ray as a sidecar container

  - Provide the correct IAM task role to X-Ray container

  - Specify env var **AWS_XRAY_DAEMON_ADDRESS** - set the host and port of the X-Ray daemon listener

- Active Tracing is for Lambda only

- You can send segment documents directly to X-Ray by using the **PutTraceSegments** API

- X-Ray compiles and processes segment documents to generate queryable trace summaries and full traces that you can access by using the **GetTraceSummaries** and **BatchGetTraces** APIs, respectively

- A **subsegment** can contain additional details about a call to an AWS service, an external HTTP API, or an SQL database

- **GetTraceSummaries** retrieves IDs and annotations for traces available for a specified time frame using an optional filter

  ```
  POST /TraceSummaries HTTP/1.1
  Content-type: application/json

  {
    "EndTime": number,
    "FilterExpression": "string",
    "NextToken": "string",
    "Sampling": boolean,
    "SamplingStrategy": { 
      "Name": "string",
      "Value": number
    },
    "StartTime": number,
    "TimeRangeType": "string"
  }
  ```

## Relational Database Service (RDS)

- Supports **Failover** to a standby replica in another AZ

- Support Automated Backups

  - The current **backup retention pediod** is 35 days

- Amazon RDS supports using **Transparent Data Encryption (TDE)** to encrypt stored data on your DB instances running **Microsoft SQL Server**. TDE automatically encrypts data before it is written to storage, and automatically decrypts data when the data is read from storage

## CloudWatch

- **Alarms**

  - Send notifications or automatically make changes to the resources based on the rules you define

  - Watches over a metric for a period of time

  - Can be used to monitor a Custom Metric for autoscaling

  - States: *OK*, *INSUFFICIENT_DATA*, *ALARM*

  - High resolution: can only choose **10 seconds** or **30 seconds**

  - Settings

    - **Period** is the length of time to evaluate the metric or expression to create each individual data point for an alarm. It is expressed in seconds. If you choose one minute as the period, there is one data point every minute

    - **Evaluation Period** is the number of the most recent periods, or data points, to evaluate when determining alarm state

    - **Datapoints to Alarm** is the number of data points within the evaluation period that must be breaching to cause the alarm to go to the ALARM state. The breaching data points don't have to be consecutive, they just must all be within the last number of data points equal to **Evaluation Period**

- **Events**

  - When a change in environment happened, e.g., EC2 changes states from pending to running... Amazon CloudWatch Events help you to respond to state changes in your AWS resources

  - Event delievers a near real-time stream of system events and triggers if a event match the rule you define

  - Supports schedule (cron)

  - Can target/trigger a variety of services such as Lambda, SQS, SNS topics...

- **Metrics**

  - Collects and tracks metrics, which are variables you can measure for your resources and applications

  - EC2

    - Default monitoring: 5 minutes

    - Detailed monitoring: 1 minutes

  - Custom

    - Use **StorageResolution** API parameter to set resolution (default is **60 seconds**)

    - Regular/Standard resolution: 60 seconds

    - High resolution: 1 second

  - To push metric, uses **PutMetricData** API

  - Uses exponential backoff in case of throttle error

  - You can use Amazon CloudWatch Monitoring Scripts for Amazon Elastic Compute Cloud (Amazon EC2) Linux-based instances to produce and consume Amazon CloudWatch custom metrics

  - Is useful to report memory, swap, and disk space utilization metrics for a Linux instance

- Logs

  - Used to monitor, store and access log files from different services

  - Can export log files to S3

## ElastiCache

- Cache strategies

  - **Lazy Loading**: Loads data to cache when requests

    - Pros

      - Only requested data is cached

      - Node failures are not fatal

    - Cons

      - Cache miss penalty: 3 round trips (check cache, read data from database, write cache)

      - Stale data in cache. Mitigation: implements TTL to expire cache data

  - Write Through: adds or updates cache when database is updated

    - Pros

      - No stale data

      - Write penalty vs Read penalty (each write requires 2 call). Write is slightly slower

    - Cons

      - Missing data until it is added/updated in the DB. Mitigation: implements Lazy Loading as well

      - Cache churn - a lot of data will never be read

- Redis

  - More durable and powerful cache layer

  - Single-threaded server

- Memcached

  - Multi-threaded server

  - Recommends for large nodes with multiple cores or threads; simplest model possible; cache objects, such as a database

## Simple Work Flow (SWF)

- Task execution max duration is 1 year

## Virtual Private Network (VPC)

- 5 VPCs per Region

- 200 Subnets per VPC

- Flow Logs

  - Used for capture IP traffic going to and from network interfaces in the VPC

- NAT

  - In a **private subnet**, we need to have a **NAT** or **Internet Gateway** to communicate with public services such as S3 and access the Internet

- Route Table

  - Limit 200 per VPC

  - 50 routes per route table

- A VPC can only have one IGW attached at any given time

## Secrets Manager

- Key rotation

- Secrets can be shared across account

- Lambda integration

- Random key generator

## Others

- If you don't select a region, then us-east-1 (US Virginia) will be used by default, which happens to be the first region AWS started in and where most of the AWS services reside

- Usually, if there is a **throttle issue**, the solution will be **exponential backoff retries**

- **CLI**

  - **--page-size**: full dataset is received but behind the scene API will make smaller, more frequent calls to get all the data, helps avoid timeouts (default is 1000)

  - **Pagination**:

    - **--max-items**: max numbers of records returned by CLI. If there are more contents to retrieve, *NextToken** will be included

    - **--starting-token**: specifies the last received **NextToken** to keep on reading

- **Amazon Inspector** provides security assessments, mostly used with EC2

- OpsWorks: Chef and Puppet management

- **Amazon QuickSight** has an ML Insights feature which leverages AWS’s proven machine learning (ML) and natural language capabilities to help you gain deeper insights from your data. It provides interactice dashboards and visualization capabilities

- Working with **Server Certificates**

  - **AWS Certificate Manager (ACM)**

  - **IAM certificate store**

    - Use IAM as a certificate manager only when you must support HTTPS connections in a Region that is not supported by ACM

    - you cannot manage your certificates from the IAM Console

- **AWS Glacier**

  - Standard retrievals: within 3–5 hours. This is the default option

  - Bulk retrievals: within 5–12 hours

  - Expedited retrievals: within 1–5 minutes

- Billing

  - Use **consolidate billing** for multiple accounts

- **App Sync**

  - **AWS AppSync** is quite similar with **Amazon Cognito Sync** which is also a service for synchronizing application data across devices. It enables user data like app preferences or game state to be synchronized as well however, the key difference is that, it also extends these capabilities by allowing multiple users to synchronize and collaborate in real time on shared data

- **Athena** can handle far more complex ad-hoc queries on data in Amazon S3, this entails an increase in operating cost as compared to **S3 Select**

- In general, API actions stick to the **PascalCase** style with the first letter of every word capitalized. **However**, CLI actions use **hyphens** with lower case

