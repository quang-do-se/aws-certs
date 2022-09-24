### Code Commit

#### Overview

- A Git repository similar to GitHub and BitBucket.

- It's private, fully managed and highly available.

- Code is only in AWS Cloud acount (increased security and compliance).

- It is secure (encrypted, access control, etc...)

- It can be integrated with Jenkins, CodeBuild and other CI tools.

#### Useful links

- Using identity-based policies (IAM Policies) for CodeCommit: https://docs.aws.amazon.com/codecommit/latest/userguide/auth-and-access-control-iam-identity-based-access-control.html

- Refining Access to Branches in AWS CodeCommit: https://aws.amazon.com/blogs/devops/refining-access-to-branches-in-aws-codecommit/

- Determining whether a request is allowed or denied within an account: https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html#policy-eval-denyallow
  - An explicit deny has higher priority than an explicit allow.

#### Securing the Repository and Branches

- Configure an IAM policy to limit pushes and merges to a branch: https://docs.aws.amazon.com/codecommit/latest/userguide/how-to-conditional-branch.html#how-to-conditional-branch-create-policy

- Inline Policy to deny making changes in `main` and `prod` branches:

``` javascript
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": [
                "codecommit:GitPush",
                "codecommit:DeleteBranch",
                "codecommit:PutFile",
                "codecommit:MergeBranchesByFastForward",
                "codecommit:MergeBranchesBySquash",
                "codecommit:MergeBranchesByThreeWay",
                "codecommit:MergePullRequestByFastForward",
                "codecommit:MergePullRequestBySquash",
                "codecommit:MergePullRequestByThreeWay"
            ],
            "Resource": "arn:aws:codecommit:us-east-2:111111111111:MyDemoRepo",
            "Condition": {
                "StringEqualsIfExists": {
                    "codecommit:References": [
                        "refs/heads/main", 
                        "refs/heads/prod"
                     ]
                },
                "Null": {
                    "codecommit:References": "false"
                }
            }
        }
    ]
}

```

#### Use notifications, triggers or CloudWatch events to set up automation between CodeCommit and other services (Lambda, SNS, SQS...)

#### Create an AWS CodeCommit trigger for an AWS Lambda function 

Ref: https://docs.aws.amazon.com/codecommit/latest/userguide/how-to-notify-lambda.html

`boto3` is a Python client library for AWS.

----------

### CodeBuild

#### Overview

- Fully managed build service.

- Alternative to other build tools such as Jenkins, GitHub Actions.

- Leverages Docker under the hood for reproducible builds and it's ephemeral.

- Build instructios are defined in `buildspec.yml`.

- It's secure: 
  - Integration with KMS for encryption of build artifacts.
  - IAM for build permissions.
  - VPC for network security. 
  - CloudTrail for API calls logging.
  
- It can get the source code from GitHub, CodeCommit, CodePipeline, S3...

- The logs of the build can be sent to CloudWatch Logs or S3.

- The build can be monitored usind CloudWatch Metrics and get statistics from them.
  
- We can use CloudWatch Events to detect failed builds and trigger notifications.

- We can use CloudWatch Alarms to notify us if there are too many failures.

- CloudWatch Events and Lambda can be used as a blue between CodeBuild and other services.

- CodeBuild can sent notifications to SNS using `CodeBuild triggers`.

#### Lambda vs. CodeBuild

- Lambda has a timeout of 15 minutes
- CodeBuild can have between 5 minutes and 8 hours for a timeout.

#### Reference: 

- Buildspec: https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html
- Samples: https://docs.aws.amazon.com/codebuild/latest/userguide/use-case-based-samples.html
- Restricting access to Systems Manager parameters using IAM policies: https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-paramstore-access.html
- Environment variables in build environments: https://docs.aws.amazon.com/codebuild/latest/userguide/build-env-ref-env-vars.html
- AWS CodeBuild resources and operations: https://docs.aws.amazon.com/codebuild/latest/userguide/auth-and-access-control-iam-access-control-identity-based.html#arn-formats
- Validating AWS CodeCommit Pull Requests with AWS CodeBuild and AWS Lambda: https://aws.amazon.com/blogs/devops/validating-aws-codecommit-pull-requests-with-aws-codebuild-and-aws-lambda/
- Automating your API testing with AWS CodeBuild, AWS CodePipeline, and Postman: https://aws.amazon.com/blogs/devops/automating-your-api-testing-with-aws-codebuild-aws-codepipeline-and-postman/

#### Good points

- It's better to have Docker login early in `pre_build` phase so we don't have to waste time building an image if the login fails. Fail fast is better.

- Tip: Go to Support Center to get **Account Number** or **Account Id**

- Intergration with CodeBuild can be implemented with CloudWatch, EventBridge or CodePipeline.

----------

### CodeDeploy

#### How it works

- Each EC2 Machine (or On Premise machine) must be running the CodeDeploy Agent.

- The agent is continuously polling CodeDeploy for work to do.

- CodeDeploy sends `appspec.yml` file.

- Application is pulled from GitHub or S3.

- EC2 will run the deployment instructions.

- CodeDeploy Agent will report of success/failure of deployment on the instance.

#### Features

- EC2 instances are grouped by deployment group (dev/test/prod).

- Lots of flexibility to define any kind of deployments.

- CodeDeploy can be chained into CodePipeline and use artifacts from there.

- CodeDeploy can re-use existing setup tools, works with any application, and even have integration with auto scaling.

- Note: Blue/Green only works with EC2 instances (not on premise).

- Support for AWS Lambda deploymens as well as EC2 and ECS.

- CodeDeploy does NOT provision resources. We have to provision all the instances in advance.

#### CodeDeploy and EC2

- EC2 will need IAM role to access S3
- Install CodeDeploy agent on EC2
- Add tags so CodeDeploy knows which EC2 instance to deploy

#### Application -> Deployment Group -> Deployment

- We can create Deployment Group for each environment (separated by EC2 tags)

#### Deployment configuration

- We can specify the percentage of healthy instances that must be available during the deployment.

#### Blue/green deployment

- Need EC2 Auto Scaling group

#### Hooks

- List of lifecycle event hooks (IMPORTANT!!!): https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file-structure-hooks.html#reference-appspec-file-structure-hooks-list

- Environment variable availability for hooks: https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file-structure-hooks.html#reference-appspec-file-structure-environment-variable-availability

#### Monitoring deployments with Amazon CloudWatch EventBridge

- https://docs.aws.amazon.com/codedeploy/latest/userguide/monitoring-cloudwatch-events.html
- https://aws.amazon.com/blogs/devops/view-aws-codedeploy-logs-in-amazon-cloudwatch-console/
  - Need to install the CloudWatch Logs agent on EC2 instance
- Monitoring deployments with Amazon SNS event notifications: https://docs.aws.amazon.com/codedeploy/latest/userguide/monitoring-sns-event-notifications.html
  - CloudWatch EventBridge is external to CodeDeploy (more flexible, more options); WHILE Code Deploy's Triggers would directly interact only with Amazon SNS.
  
#### Roll back

- https://docs.aws.amazon.com/codedeploy/latest/userguide/deployments-rollback-and-redeploy.html

#### Advanced options for a deployment group

- https://docs.aws.amazon.com/codedeploy/latest/userguide/deployment-groups-configure-advanced-options.html


#### Working with on-premises instances

- https://docs.aws.amazon.com/codedeploy/latest/userguide/instances-on-premises.html
- https://docs.aws.amazon.com/codedeploy/latest/userguide/on-premises-instances-register.html

#### Lambda Deployment Configurations

- https://docs.aws.amazon.com/codedeploy/latest/userguide/deployment-configurations.html#deployment-configuration-lambda

- AppSpec hooks for Lambda: https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file-structure-hooks.html#appspec-hooks-lambda

----------

### CodePipeline

#### Overview

- Continuous delivery
- Visual workflow

- Pipeline -> Stages -> Actions
  - Each pipeline can have zero to many stages (sequential)
  - Each stage can have zero to many action groups (sequential)
  - Each action group can have zero to many actions (parallel)
    - In code, use `runOrder` value to control action order

- Pipeline structure: https://docs.aws.amazon.com/codepipeline/latest/userguide/reference-pipeline-structure.html

- Monitoring CodePipeline events: https://docs.aws.amazon.com/codepipeline/latest/userguide/detect-state-changes-cloudwatch-events.html#detect-state-events-pipeline

- CodeBuild artifact is different from CodePipeline artifact
  - They both use S3

- CodeDeploy and actions could be for different regions: https://docs.aws.amazon.com/codepipeline/latest/userguide/actions-create-cross-region.html
  - We can have a central pipeline in one region that can deploy to mutliple regions.
  
#### Best practices and use cases

- https://docs.aws.amazon.com/codepipeline/latest/userguide/best-practices.html

#### Invoke Lambda function

- https://docs.aws.amazon.com/codepipeline/latest/userguide/actions-invoke-lambda-function.html

#### Create a pipeline with CloudFormation

- https://docs.aws.amazon.com/codepipeline/latest/userguide/tutorials-cloudformation.html

#### Exercise - best practices to set up CodePipeline for production deployment

- https://github.com/aws-samples/codepipeline-nested-cfn

#### Reading - Implementing GitFlow Using AWS CodePipeline, AWS CodeCommit, AWS CodeBuild, and AWS CodeDeploy (IMPORTANT!!!)

- https://aws.amazon.com/blogs/devops/implementing-gitflow-using-aws-codepipeline-aws-codecommit-aws-codebuild-and-aws-codedeploy/

### CodeStar

The main file is `template.yml`. Use it to update your project:

- https://docs.aws.amazon.com/codestar/latest/userguide/templates.html#update-project

----------

### Cloud Formation

#### Resource types

https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html

Form 2019: `AWS::aws-product-name::data-type-name`

Form 2022: `service-provider::service-name::data-type-name`

#### Intrinsic Functions

  - `Fn::Base64 : <valueToEncode>`: encodes value to base64
    - It can be used to pass `UserData` script to CloudFormation.
    - `UserData` script log is stored in `/var/log/cloud-init-output.log`.

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

  - `Fn::Ref <logicalName>` or `!Ref <logicalName`: return the value of the specified *parameter* or *resource*

    - Parameter: return the value of the specified parameter

    - Resource: return the physical ID of the resource

#### Condition functions:

  - `Fn::And: [condition, ...]`

  - `Fn::Equals: [value_1, value_2]`

  - `Fn::If: [condition_name, value_if_true, value_if_false]`

  - `Fn::Not: [condition]`

  - `Fn::Or: [condition, ...]`

#### Helper scripts

- **cfn-init**: Use to retrieve and interpret resource metadata, install packages, create files, and start services
  - Logs go to `/var/log/cfn-init.log`.
  - It's an alternative for `UserData`.

- **cfn-signal**: Use to signal with a CreationPolicy or WaitCondition, so you can synchronize other resources in the stack when the prerequisite resource or application is ready
  - Need `CreationPolicy` property

- **cfn-get-metadata**: Use to retrieve metadata for a resource or path to a specific key

- **cfn-hup**: Use to check for updates to metadata and execute custom hooks when changes are detected
  - For example, when EC2 instance's metadata is updated by CloudFormation, `cfn-init` is not run again and has to be triggered by `cfn-hup`. `cfn-init` is only when stack is freshly created. This allows us to keep the same EC2 instance and just update its metadata instead of deleting it and deploying a new one.

#### ChangeSets

- When you update a stack, you need to know what changes before it happens for greater confidence.
- ChangeSets won't say if the update will be successful.

#### Parameters

We can use Systems Manager's Parameters to centralize configuration values.

#### Public parameters

You can search for AMI Linux images's parameter path with: `aws ssm get-parameters-by-path --path /aws/service/ami-amazon-linux-latest  --query 'Parameters[].Name'`

#### Lambda

##### Inline

- Cannot have dependencies.
- Limit to 4000 characters.

##### Zip

- We can update zip file with a `index.py` and dependencies.
- We can use S3 versioning to reference unique version.

#### Stack Status Codes

- https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-describing-stacks.html#cli-stack-status-codes

#### `UPDATE_ROLLBACK_FAILED` is a very bad state

- Continue Rolling Back (DevOps blog): https://aws.amazon.com/blogs/devops/continue-rolling-back-an-update-for-aws-cloudformation-stacks-in-the-update_rollback_failed-state/
- Common errors: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/troubleshooting.html#troubleshooting-errors-update-rollback-failed

#### Capabilities

- If you got `InsufficientCapabilitiesException`, you may need to add `CAPABILITY_IAM` or `CAPABILITY_NAMED_IAM` capability for your CloudFormation.

- If you have nested stacks, you will need `CAPABILITY_AUTO_EXPAND`.

#### Stack Policy

We can use stack policy to prevent updates to resources: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/protect-stack-resources.html

----------

### Elastic Beanstalk

#### Saved configurations

We can save non-default configurations then later import and deploy it to other regions quickly.

#### .ebextensions

- General options for all environments: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/command-options-general.html
- Option settings: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/ebextensions-optionsettings.html
- Option Precedence: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/command-options.html#configuration-options-precedence
  - Settings applied directly to the environment
  - Saved Configurations
  - Configuration Files (.ebextensions)
  - Default Values
- Container Commands vs Commands: https://stackoverflow.com/questions/35788499/what-is-difference-between-commands-and-container-commands-in-elasticbean-talk/40096352#40096352

#### Version

- Can have at most 1000 versions.
- Use `Application version lifecycle settings` to get rid of old versions.

#### Deployment Modes (IMPORTANT!!!)

- https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.deploy-existing-version.html

#### Swap Environment URLS

We can swap URLs between 2 environments. It is very useful for testing. It modifies Route 53 DNS configuration so it can take some time for the change to be in place.

### Lambda

CPU is allocated proportional to the memory configured.

Maximum timeout is 15 minutes.

Lambda is not good for long process (AWS Batch) or synchronize functions with one another (AWS Step Functions).

Lambda version is immutable. 

Lambda alias is good for A/B testing and can assign weight for each version.

#### SAM (Serverless Application Model)

There is CLI with template to build applications.

We can build and invoke lambda locally.

SAM uses Cloud Formation behind the scene. It also uses Code Deploy to deploy Lambda functions.

----------

### Step Functions

- It helps orchestrate complex workflows.
- It helps coordinate Lambda functions or AWS Batch jobs.

#### Must read (IMPORTANT!!!)

https://aws.amazon.com/step-functions/use-cases/

----------

### API Gateway

- https://aws.amazon.com/blogs/compute/introducing-amazon-api-gateway-private-endpoints/

- https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-private-apis.html

#### Lambda Proxy

- API points to Lambda alias. It allows blue/green testing while keeping the API as is. We can modify the weight of each version in Lambda alias.
- Request details will be available in the `event` of the handler function.

#### Mapping Templates

- We can map both response and request in API Gateway.

#### Throttling

- API Gateway Account-Level: 10,000 requests per second.
- Usage Plan: user-defined rate
- Lambda: 1,000 concurrent requests per second

#### Trigger other AWS services

- https://docs.aws.amazon.com/step-functions/latest/dg/tutorial-api-gateway.html

----------

### Elastic Container Service (ECS)

- EC2 instance is created with a configuration file `/etc/ecs/ecs.config`.

#### Task Definitions

- Task definitions are metadata in JSON form to tell ECS how to run a Docker container.

- It contains crucial information about:
  - Image Name
  - Port Binding for Container and Host
  - Memory and CPU required
  - Environment variables
  - Networking information

#### IAM Roles

For ECS, there are 2 roles we need to know about:
- EC2 instance role (allows the ECS agent on EC2 instance to do calls againsts ECS service)
  - This role is attached to EC2 instances.
- Task role (for Docker containers to do its operations with AWS services such as pushing data to S3)
- Service role (for auto scaling and load balancing)

#### Load Balancer and Security Group

- For dynamic port feature (multiple containers on the same EC2 instances), because we don't know what ports will be used on EC2 instances, we need to add an `inbound rules` for EC2 instances that allows all ports from Load Balancer's security group to EC2 instances.

- **NOTE**: Security Groups are STATEFUL.

#### Scaling

Scaling with ECS may be challenging since we need 2 autoscaling:
- Service Autoscaling (only scaling tasks)
- EC2 Instance Autoscaling

Using Fargate helps simplify autoscaling strategy.

`Capacity Provider` may help solve the problems with ECS autoscaling:
- ECS Cluster Auto Scaling Deep Drive: https://www.youtube.com/watch?v=Fb1EwgfLbZA

#### Workshop and tutorials (IMPORTANT!!!)

- https://ecsworkshop.com/
- https://docs.aws.amazon.com/codepipeline/latest/userguide/tutorials-ecs-ecr-codedeploy.html#tutorials-ecs-ecr-codedeploy-imagerepository
- https://docs.aws.amazon.com/codepipeline/latest/userguide/ecs-cd-pipeline.html
- https://aws.amazon.com/blogs/devops/build-a-continuous-delivery-pipeline-for-your-container-images-with-amazon-ecr-as-source/

----------

### OpsWorks

- It's an all integrated solution for Chef Cookbooks.
- It has stack manager, layers manager, instances manager, apps manager, monitoring tool...

- (IMPORTANT!!!) Need to know `OpsWorks Stacks LifeCycle Events` for exam: https://docs.aws.amazon.com/opsworks/latest/userguide/workingcookbook-events.html
  - `Configure` event applies to all instances when one of them goes online or offline. This could be used to configure all the instances at once and make them aware of each other.
  - `Setup`, `Deploy`, `Undeploy`, `Shutdown` are instant specific and they can be run individually.

----------

### CloudTrail

- Use CloudTrail to track API events. It can store logs in S3 bucket or CloudWatch log group.

- A `trail` can logs events in all AWS Regions and store those events in S3 Bucket.
  - https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-create-and-update-a-trail.html

- Log integrity (in case of hacking): https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-log-file-validation-intro.html

- Receiving CloudTrail log files from multiple accounts: https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-receive-logs-from-multiple-accounts.html
  - Need to set up S3 bucket policy for other accounts to put logs there

----------

### Kinesis

#### Kinesis Data Streams

- Producer limits:
  - 1MB/s or 1000 messages/s at write PER SHARD.
  - Otherwise, you will get `ProvisionedThroughputException`.

- Consumer Classis limits:
  - 2MB/s at read PER SHARD across all consumers.
  - 5 API calls per second PER SHARD across all consumers.
  - For example, if we have 3 different applications and each consume 1MB/s PER SHARD, then we may get some throttling.

- Data Retension:
  - 24 hours date retentions by default.
  - Can be extended to 7 days.

- What can be Producers?
  - Kinesis SDK
  - CloudWatch Logs (important)
  - Kinesis Producer Library (KPL), Kinesis Agent, Spark, Log4J Appenders, FLume, Kafka Connect, NiFi...

- What can be Consumers?
  - Kinesis SDK
  - Kinesis Firehose
  - AWS Lambda
  - Kinesis Client Library (KCL), Kinesis Connector Library, Spark, Log4J Appenders, FLume, Kafka Connect...

- Kinesis Client Library (KCL):
  - Use DynamoDB to track other workers and share the work amongst shards.
  - Great for reading in a distributed manner.
  - Cannot have more KCL applications than SHARDS in your stream.

#### Kinesis Data Firehose

- Fully managed service, no admnistration
- Near real time
- Load data into Redshift, S3, ElastricSearch, Splunk
- Automatic scaling
- Data transformation through AWS Lambda (e.g. CSV -> JSON)

- What can send to Kinesis Data Firehose?
  - Kinesis Producer Library (KPL)
  - Kinesis Agent
  - Kinesis Data Streams
  - CloudWatch Logs & Events
  - IoT rules actions

- Where can Kinesis Data Firehose deliver to?
  - Amazon S3
  - Redshift
  - ElasticSearch
  - Splunk

#### Kinesis Data Streams vs Firehose

- Streams
  - Going to write custom code (producer / consumer)
  - Real time (~200 ms latency for classic)
  - Must manage scaling (shard splitting / merging)
  - Data Storage for 1 to 7 days, replay capability, multi consumers
  - Use with Lambda to insert data in real-time to ElasticSearch (for example)

- Firehose
  - Fully managed, send to S3, Splunk, Redshift, ElasticSearch
  - Serverless data transformations with Lambda
  - Near real time (lowest buffer time is 1 minute)
  - Automated Scaling
  - No data storage

----------

### CloudWatch

#### Metrics

- `Detailed Monitoring` = every 1 minute
  - We have to pay for `Detailed Monitoring`.

- `Basic Monitoring` = every 5 minutes

- Metric Retention
  - Up to 15 months
  - Less granularity overtime
  - Reference: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch_concepts.html#metrics-retention
  
- Custom metrics
  - Standard resolution (default) = one-minute granularity
  - High resolution = one-second granularity
  - You can have up to 10 dimensions in one metric

#### Alarms

- An Alarm can only have 1 metric.
- It can send notification, execute auto scaling or EC2 actions.

#### Logs

- Log Groups -> Log Streams

- On EC2 instance, we can install Unified CloudWatch Agent to collect more metrics such as memory usage: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html

- We can create `metric filters` for a log group and then create `alarms` for those `metric filters`.

- Real-time processing of Log Data with Subscriptions: https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/Subscriptions.html
  - It delivered to other services such as an `Amazon Kinesis stream`, an `Amazon Kinesis Data Firehose stream`, or `AWS Lambda` for custom processing, analysis, or loading to other systems. 
  
- There are 2 ways to send logs to S3: 
  - Using AWS Event with Lambda function
  - Using `Subscription filters`

#### EventBridge and S3 (IMPORTANT!!!)

- We can use `CloudTrail` event to intercept custom events.

- In S3 properties, we can create `event notification` to send message to Lambda, SNS topic, or SQS queue.
  - EventBridge can have `Object Level` or `Bucket Level` Operations for S3. However for Object Level Operations to work with EventBridge, we need to have `trail(s)` in Cloud Trail that configured to received those events.
  - S3 events do not give us `Bucket Level` Operations.

#### Dashboard

- If the exam asks "how can you correlate date?", CloudWatch Dashboard should be your answer. (IMPORTANT!!!)

----------

### X-Ray

- Distributed tracing, debugging
- https://aws.amazon.com/blogs/devops/using-amazon-cloudwatch-and-amazon-sns-to-notify-when-aws-x-ray-detects-elevated-levels-of-latency-errors-and-faults-in-your-application/

----------

### Amazon ElasticSearch

- No serverless offering

- ElasticSearch + Kibana + Logstash
  - ElasticSearch: Provide search and indexing capability
  - Kibana: Provide real-time dashboards on top of the data sits in ES
  - Logstash: Log ingestion mechanism

----------

### Tagging strategies

- https://docs.aws.amazon.com/general/latest/gr/aws_tagging.html

----------

### System Manager

- Patching Automation


#### AWS Resource Groups

You can use resource groups to organize your AWS resources.

#### Automation

- Can be used to automate complex tasks
- Build Golden AMI (IMPORTANT!!!): https://d1.awsstatic.com/whitepapers/aws-building-ami-factory-process-using-ec2-ssm-marketplace-and-service-catalog.pdf

----------

### EC2 Instance Compliance

#### AWS Config

- Audit and ensure resource compliance over time
- Remediate noncompliant resources with `CloudWatch Events` or native integration with `AWS Systems Manager Automation`

#### AWS Inspector

- Security Vulnerabilities scan from within the OS using the agent (
  - Need to manually install agent or assign the right role to allow Inspector to install agent on its own
- Or outside network scanning (no need for the agent)

#### AWS Systems Manager

- Run automations, patches, commands, inventory at scale

#### AWS Service Catalog

- Restrict how the EC2 instances can be launched to minimize configurations
- Helpful to onboard beginner AWS users

#### Other configuration management tools

- AWS tools: SSM, Opsworks, Ansible, Chef, Puppet, User Data
- Ensure the EC2 instances have proper configuration files

----------

### Health

AWS Health AWS_RISK_CREDENTIALS_EXPOSED remediation: https://github.com/aws/aws-health-tools/tree/master/automated-actions/AWS_RISK_CREDENTIALS_EXPOSED

----------

### Trusted Advisor

- Global service - need to be in North Virginia region to create an Event.
- Use cases: https://github.com/aws/Trusted-Advisor-Tools
  - Can check low and high utilization EC2 
  - Integrate with `EventBridge`, `SSM Automation` and `Lambda`
- Can have `CloudWatch Alarms` for tracking service limit susage (Paid option)
- Can only refresh every 5 minutes and need to be triggered by API `refresh-trusted-advisor-check`.

----------

### GuardDuty

- Detect threat with Machine Learning
- Continuously monitors for malicious activity and unauthorized behavior to protect your AWS accounts, workloads, and data stored in S3. 
- Use `CloudTrail Logs`, `VPC Flow Logs` and `DNS Logs`

----------

### Macie

- Analyze S3 for sensitive data such as Credit Cards, Personal Identifiable Information (PPI), Proctected Health Information (PHI), Private Keys.

----------

### AWS Secrets Manager

- Easily rotate, manage, and retrieve secrets throughout their lifecycle

----------

### AWS License Manager

- Manage license usage for Windows or Oracle...

----------

### Cost Allocation Tags - Billing

- We can slice and dice our cost by tags, allocate our cost to different cost centers.

----------

### AWS Data Protection (IMPORTANT!!!)

- https://www.udemy.com/course/aws-certified-devops-engineer-professional-hands-on/learn/lecture/16349516#overview

----------

### Auto Scaling Group

#### Launch Template

- `Launch Template` allows a combination of On-Demand and Spot instances.
- We can create versions for Launch Template and inherit configurations from other templates as well.
- It is the future of ASG. We should move away from `Launch Configuration`.

#### Schedule Actions

- We can schedule in advance how ASG should behave.

#### Dynamic Scaling Policy

- Both `Target tracking scaling`, `Simple scaling`, `Step scaling` use `CloudWatch Alarm` behind the scene.

#### Target Group

- It takes care of health check and monitor instances.
- It also controls attributes such as `Slow start`, `Load balancing algorithm` and `Stickiness`.

#### Suspend and resume a process for an Auto Scaling group (IMPORTANT!!!)

- https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-suspend-resume-processes.html?icmpid=docs_ec2as_help_panel
- Detach instance
- Set to StandBy
- Set scale-in protection

#### Lifecycle Hooks (IMPORTANT!!!)

- https://docs.aws.amazon.com/autoscaling/ec2/userguide/lifecycle-hooks-overview.html
- https://github.com/aws-samples/aws-lambda-lifecycle-hooks-function

#### Termination Policies

- https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-termination-policies.html#default-termination-policy

- https://aws.amazon.com/about-aws/whats-new/2015/12/protect-instances-from-termination-by-auto-scaling/
  - For example, EC2 -> SQS. Before an EC2 instance process a message, call scale-in protection API.

#### CloudFormation Update

- https://aws.amazon.com/premiumsupport/knowledge-center/auto-scaling-group-rolling-updates/

#### CodeDeploy

- https://docs.aws.amazon.com/codedeploy/latest/userguide/integrations-aws-auto-scaling.html

----------

### DynamoDB

- https://aws.amazon.com/blogs/database/choosing-the-right-dynamodb-partition-key/

#### Global Tables

- Replicating data between Regions and resolving update conflicts.
- Table must be empty and `DynamoDB Streams` are enabled.
  - `DynamoDB Streams` uses `Kinesis Stream` behind the scene. 2MB/s at read PER SHARD across all consumers.
  - No more than 2 processes should be reading from the same streams shard at the same time. Having more than 2 readers per hard can result in throttling.
  - Work around: Have 1 Lambda read from the stream and forward to SNS topics.

#### Patterns

- S3 Metadata Index: S3 metadata -> Lambda -> DynamoDB -> API search
- Elastic Search: DynamoDB Table -> DynamoDB Stream -> Lambda function -> Amazon Elastic Search
  - DynamoDB Table is good at retrieving items but bad with search (scan whole table, inefficient).
  - Elastic Search is good with searching items.

----------

### Multi AZ

- Services where Multi-AZ must be enabled manually: 
  - EFS, ELB, ASG, Beanstalk: assign AZ
  - RDS, ElastiCache: multi-AZ (synchronous standby DB for failovers)
  - Aurora: 
    - Data is stored automatically across multi-AZ
    - Can have multi-AZ for the DB itself (same as RDS)
  - ElasticSearch (managed service): multi master
  - Jenkins (self deployed): multi master
  
- Service where Multi-AZ is implicitly there:
  - S3 (except OneZone-Infrequent Access)
  - DynamoDB
  - All of AWS' proprietary, managed services
  
- Elastic Block Storage (EBS)
  - EBS is tied to a single AZ
  - How can you make EBS `multi-AZ`?
    - ASG with 1 min/max/desired
    - Lifecycle hooks for `Terminate`: make a snapshot of the EBS volume
    - Lifecycle hook for `Launch`: copy the snapshot, create an EBS, attach to instance
    - Good for exam question **How can we move EBS volume from one region to another?**

----------

### Multi Regions

- DynamoDB Global Tables (multi-way replication, enabled by Kinesis Streams)
- AWS Config Aggregator (multi region & multi account)
- RDS Cross Region Read REplicas (used for Read & Disaster Recovery)
- Aurora Global Database (one region is master, other is for Read & Disaster Recovery)
- EBS volumes snapshots, AMI, RDS snapshots can be copied to other regions
- VPC peering to allow private traffic between regions
- Route53 uses a global network of DNS servers
- S3 Cross Region Replication
- CloudFront for Global CDN at the Edge Locations
- Lambda@Edge for Global Lambda function at Edge Locations (A/B testing)

----------

### Multi Accounts

- https://aws.amazon.com/blogs/architecture/stream-amazon-cloudwatch-logs-to-a-centralized-account-for-audit-and-analysis/
- https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CrossAccountSubscriptions.html

----------

### CloudFormation `StackSets`

- Create, update, or delete stacks across multiple accounts and regions with a single operation

- Permissions needed: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-prereqs-self-managed.html

----------

### CodePipeline Multi Regions

- Hands-on: https://aws.amazon.com/blogs/devops/using-aws-codepipeline-to-perform-multi-region-deployments/

----------

### Disaster Recoverty

#### Objectives

- `RPO`: Recovery Point Objective
- `RTO`: Recovery Time Objective
- The smaller these are, the higher the cost

#### Strategies

- Backup and Restore
  - Very easy and cheap
  - High RPO and High RTO
- Pilot Light
  - Backup and replicate the critical core systems only
  - Manage costs
- Warm Standby
  - Full system is up and running, but at minimum size
  - More costly but reduce RPO & RTO
- Hot Site / Multi Site Approach
  - Full Production Scale is running on AWS and on-premises
  - Very low RTO, very expensive

#### Tips

- Backup
  - EBS Snapshots, RDS Automated backups/Snapshots, etc...
  - Regular pushes to S3 / S3 IA / Glacier, Lifecycle Policy, Cross Region Replication
  - From On-Premise: Snowball or Storage Gateway
  
- High Availability
  - Use Route53 to migrate DNS over from Region to Region
  - RSD Multi-AZ, ElastiCache Multi-AZ, EFS, S3
  - Site to Site VPN as a recovery from Direct Connect
  
- Replication
  - RDS Replication (Cross Region), AWS Aurora + Global Databases
  - Database replication from on-premise to RDS
  - Storage Gateway

- Automation
  - CloudFormation / Elastic Beanstalk to re-create a whole new environment
  - Recover / Reboot EC2 instances with CloudWatch if alarms fail
  - AWS Lambda functions for customized automations

- Chaos
  - Netflix has a "simian-army" randomly terminating EC2 (chaos testing in production)

#### Multi-Region Disaster Recovery Checklist

- Is my AMI copied? Is it stored in the parameter store?
- Is my CloudFormation StackSet working and tested to work in another region?
- What's my RPO and RTO and cost associated with it?
- Are Route53 Health Checks working correctly? Tied to a CW Alarm?
- How can I automate with CloudWatch Events to trigger some Lambda functions and perform a RDS Read Replication promotion?
- Is my data backed up? Where is my data living? How is it synchronized? How is it replicated?

#### Backups

- EFS Backup:
  - AWS Backup with EFS (frequency, when, retain time, lifecycle policy) - managed
  - EFS to EFS backup (maybe obsolete with AWS Backup): https://aws.amazon.com/solutions/implementations/efs-to-efs-backup-solution/
  - Multi-region idea: EFS -> S3 -> S3 CRR -> EFS
  
- Route53 Backup:
  - Use `ListResourceRecordSets` API for exports
  - Write your own scripts for imports into R53 or other DNS provider
  
- Elastice Beanstalk Backup:
  - Saved configurations using the EB CLI or AWS console.
