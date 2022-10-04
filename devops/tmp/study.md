## Exam tips

- Ignore "You do not have to do anything" answer. It will always be wrong.

- "You want to focus on the simplest, most technically correct answer."

- We wanna automate all aspects of our environment. We wanna use that infrastructure as code mentality. **Manual step isn't going to be the right answer**. We wanna avoide **manual steps** in any of the scenarios that were given on the exam.

- When routing **portions of users**, always choose Amazon Route 53.

- When you see a question about reporting on API calls or logs, that's likely to be `AWS CloudTrail`.

- Use `Roles` everywhere you can. Avoid permanent security credentials.

- If you see the phrase **Network Instrusion Detection**, you should think of `Amazon GuardDuty`.

- Focus on IAM solution (maybe).

- Automation is key.
  - The solution must Optimize cost.
  - The solution must automatically apply to new instances and new regions.

----------

## Review

- CloudFormation Helper Scripts
- Lambda SAM Framework
- EventBridge 
  - Event bus

----------

## Domain 1 - SDLC Automation
## Domain 2 - Configuration Management and Infrastructure as Code
## Domain 3 - Monitoring and Logging
## Domain 4 - Policies and Standards Automation
## Domain 5 - Incident and Event Response
## Domain 6 - High Availability, Fault Tolerance, and Disaster Recovery

----------

## CodeCommit

- CodeCommit `Notification` vs `Trigger`
  - Notifications should be used for literal notification and not for taking action based on them.
  - Triggers are supposed to initiate action. So, if I need to invoke some service based on this event on which trigger is based, I would do that and hence the option to integrate Lambda service. In a way to add automation after codecommit events.
  - Triggers are more limited in scope: Push to existing branch, create branch or tag, delete branch or tag.

- `Code Commit` can trigger Lambda directly.

- `Code Commit` supports IAM policies only.

- CodeCommit's AWS manage policy `AWSCodeCommitPowerUser` allow users access to all the functionality of CodeCommit but it does NOT allow them to delete or create repositories.

----------

## CodeBuild

- (IMPORTANT!!!) `buildspec.yml`
  - https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html#build-spec-ref-syntax

- Validating AWS CodeCommit Pull Requests with AWS CodeBuild and AWS Lambda: https://github.com/aws-samples/aws-codecommit-pull-request-aws-codebuild

- Lambda vs. CodeBuild
  - Lambda has a timeout of 15 minutes
  - CodeBuild can have between 5 minutes and 8 hours for a timeout.
  
- `CodeBuild Triggers` allow you to schedule automated builds every hour, day, week or custom time.  

- `CodeBuild` NamespaceType can add `BUILD_ID` into artifact path: https://docs.aws.amazon.com/codebuild/latest/APIReference/API_ProjectArtifacts.html#CodeBuild-Type-ProjectArtifacts-namespaceType

----------

## CodeDeploy

- `appspec` file can be YML or JSON.

- List of lifecycle event hooks (IMPORTANT!!!): https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file-structure-hooks.html#reference-appspec-file-structure-hooks-list

- CodeDeploy's different deployment modes: https://docs.aws.amazon.com/codedeploy/latest/userguide/deployment-configurations.html

- Deployment groups in CodeDeploy are defined by `TAGS`, not by the registration of instance IDs.

- Deployment group can have `Alarms` in advanced options.

----------

## CodeStart

- The main file is `template.yml`.

----------

## CodePipeline

- `Continuous Delivery` phase requires manual approval step. `Continuous Integration` and `Continuous Deployment` are fully automated.
  - https://aws.amazon.com/devops/continuous-integration/

----------

## CloudFormation

- CloudFormation does not detect a new file has been uploaded to `S3` unless one of these parameters change: - S3Bucket - S3Key (filename) - S3ObjectVersion

- `WaitCondtion` vs `CreationPolicy`
  - `WaitCondition` is a resource.
  - `CreationPolicy` is an attribute.
  - The `CreationPolicy` is recommended for `EC2` and `Auto Scaling`.

- We cannot do a rolling update to `Auto Scaling` unless we are utililzing `CloudFormation`. (`UpdatePolicy` attribute under `AutoScalingGroup` resource)

- In CloudFormation, updates to all resources are open by default but `Stack Policy` changes to DENY for all resources once created.
The `Stack Policy` is the IAM style policy statement which governs what can be changed and who can change it.

- (IMPORTANT!!!) UPDATE_ROLLBACK_FAILED in CloudFormation: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/troubleshooting.html#troubleshooting-errors-update-rollback-failed
  - https://aws.amazon.com/blogs/devops/continue-rolling-back-an-update-for-aws-cloudformation-stacks-in-the-update_rollback_failed-state/

- (IMPORTANT!!!) https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-update-behaviors.html#update-replacement
  - For example, `AWS::RDS::DBInstance`'s `Port` attribute has the update requirement of `Replacement`. That means when the `Port` value is changed, the database will be replaced. The database will have to be restored from backups.
  - https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-rds-dbinstance.html#cfn-rds-dbinstance-port

- Troubleshooting "Delete `CloudFormation` stack fails"
  - Some resources must be empty before they can be deleted. For example, you must delete all objects in an Amazon S3 bucket or remove all instances in an Amazon EC2 security group before you can delete the bucket or security group.

- `SAM` templates are an extension of `CloudFormation` templates and are written in YAML.

- CloudFormation `custom resources` allow you to extend CloudFOrmation to do things it could not normally do.
  - Develop `custom resources` backed by `AWS Lambda` to add intelligence to CloudFormation.

- In CloudFormation, as your infrastructure grows, common patterns can emerge in which you declare the same components in multiple templates. You can separate out these common components and create dedicated templates for them. Then use the resource in your template to reference other templates, creating `nested stacks`.

- CloudFormation's cross-stack reference: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/walkthrough-crossstackref.html

- In `CloudFormation`, `Intrinsic functions` can not be used within the `Parameters` section.

- In `CloudFormation`, to protect resources against update and deletion, use:
``` yaml
"DeletionPolicy" : "Retain",
"UpdateReplacePolicy" : "Retain"
```

- Use `Change Set` to preview the effects of the changes before executing.

----------

## SQS

- SQS size limit is 256KB
  - Files must be uploaded to S3 and a reference to them should be sent to SQS.

----------

## KMS

- AWS KMS can encrypt data only up to 4 KB in size.

----------

## ElasticBeanstalk

- (IMPORTANT!!!) Deployment methods: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.deploy-existing-version.html

- (IMPORTANT!!!) Option Precedence, from highest to lowest: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/command-options.html#configuration-options-precedence
  - `Settings applied directly to the environment` - Settings specified during a create environment or update environment operation on the Elastic Beanstalk API by any client, including the Elastic Beanstalk console, EB CLI, AWS CLI, and SDKs. The Elastic Beanstalk console and EB CLI also apply recommended values for some options that apply at this level unless overridden.
  - `Saved Configurations` - Settings for any options that are not applied directly to the environment are loaded from a saved configuration, if specified.
  - `Configuration Files (.ebextensions)` - Settings for any options that are not applied directly to the environment, and also not specified in a saved configuration, are loaded from configuration files in the .ebextensions folder at the root of the application source bundle.
    - Configuration files are executed in alphabetical order. For example, .ebextensions/01run.config is executed before .ebextensions/02do.config.
  - `Default Values` - If a configuration option has a default value, it only applies when the option is not set at any of the above levels.


- We cannot use `Elastic Beanstalk` to migrate our applications. We cannot import or export environments with `Elastic Beanstalk`.
We can't bring in resources that aren't previously created utilizing the Elastic Beanstalk service.

- `commands`: You can use the `commands` key to execute commands on the EC2 instance. 
  - The commands run **BEFORE** the application and web server are set up and the application version file is extracted.
- `container_commands`: You can use the `container_commands` key to execute commands that affect your application source code. 
  - Container commands run **AFTER** the application and web server have been set up and the application version archive has been extracted, but **BEFORE** the application version is deployed.

- When an instance is launched, Elastic Beanstalk runs `commands`, `prebuild`, `Buildfile`, `container_commands` `predeploy`, and `postdeploy`, in this order.
  - (IMPORTANT!!!) https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/platforms-linux-extend.html#platforms-linux-extend.workflow

----------

## IAM

- Control access to resource based on `tags` with `aws:ResourceTag/<tag-key>` in IAM pocily:
  - https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/control-access-with-tags.html
  - https://docs.aws.amazon.com/IAM/latest/UserGuide/access_iam-tags.html
  - `ec2:ResourceTag/<tag-key>` will work for EC2 instance
  - User and Role can also have tags and can be accessed with `"${aws:PrincipalTag/<tag-key>}"`

- `Users` are permanent set of credentials.
- `Roles` are temporary credentials and NOT something we need to rotate because it happens automatically behind the scenes.

- Cannot apply restriction conditions on root account.

- Always clean up IAM users for those who left.


----------

## Route 53

- We cannot set timed cutovers with `Route 53`.

- (IMPORTANT!!!) Route 53 routing policy: https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html
  - Simple routing policy
  - Failover routing policy (important for multiple AWS Regions and Disaster Recovery)
  - Geolocation routing policy
  - Geoproximity routing policy
  - Latency routing policy (important for multiple AWS Regions and Disaster Recovery)
  - IP-based routing policy
  - Multivalue answer routing policy
  - Weighted routing policy 

----------

## OpsWork

- Support Chef and Puppet

- OpsWorks layers:
  - Load Balancer layer
  - Application Server layer
  - Database layer

- It has stack manager, layers manager, instances manager, apps manager, monitoring tool...

- (IMPORTANT!!!) Need to know `OpsWorks Stacks LifeCycle Events` for exam: https://docs.aws.amazon.com/opsworks/latest/userguide/workingcookbook-events.html
  - `Configure` event applies to all instances when one of them goes online or offline. This could be used to configure all the instances at once and make them aware of each other.
  - `Setup`, `Deploy`, `Undeploy`, `Shutdown` are instant specific and they can be run individually.

- OpsWork automatic instance scaling options:
  - With automatic `load-based` scaling, you can set thresholds for CPU, memory, or load to define when additional instances will be started. 
  - With automatic `time-based` scaling, you can define at what time of the day instances will be started and stopped.
  
----------

## Load Balancer

- Application Load Balancer weighted target groups: https://aws.amazon.com/blogs/devops/blue-green-deployments-with-application-load-balancer/
  - https://aws.amazon.com/blogs/aws/new-application-load-balancer-simplifies-deployment-with-weighted-target-groups/

- You can't delete a `Load Balancer` if deletion protection is enabled.

----------

## EC2

- EC2 placement groups: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html#placement-groups-spread
  - `Cluster Placement Groups` are recommended for applications that benefit from low network latency, high network throughput, or both.
  - `Partition Placement Groups` help reduce the likelihood of correlated hardware failures for your application.
  - `Spread Placement Groups` are recommended for applications that have a small number of critical instances that should be kept separate from each other.

- `Security Group`
  - Stateful
  - For **Instance** level

- `Network Access Control List (NACLs)`
  - Stateless
  - For **Subnet** level

----------

## CloudWatch

- Metric Retention
  - Metric is kept for a period of 15 months
  - Less granularity overtime

- Cloudwatch Metrics are grouped into `namespaces`.

- We can create `metric filters` for a `log group` and then create `alarms` for those `metric filters`.

- Scenario: Monitor your estimated charges using CloudWatch: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/gs_monitor_estimated_charges_with_cloudwatch.html

----------

## Kinesis

- When the requirement is to process streaming data in real time, `Kinesis` must be strongly considered. "Real time" usually points to `Kinesis`.

- `Kinesis Data Streams` is a low latency streaming service in AWS Kinesis with the facility for ingesting at scale. On the other hand, `Kinesis Firehose` aims to serve as a data transfer service.

- The primary purpose of `Kinesis Firehose` focuses on loading streaming data to Amazon S3, Splunk, ElasticSearch, and RedShift. 

----------

## Trusted Advisor

- Global service - need to be in North Virginia region to create an `EventBridge` event.
  - Trusted Advisor itself cannot act as a trigger. It can be used with `EventBridge` events to create a trigger. 

- Use cases: https://github.com/aws/Trusted-Advisor-Tools
  - Can check low and high utilization EC2 
  - Integrate with `EventBridge`, `SSM Automation` and `Lambda`
  
- Can have `CloudWatch Alarms` for tracking service limit susage (Paid option)

- Can only refresh checks/reports every 5 minutes and need to be triggered by an API `refresh-trusted-advisor-check`.
  - Trusted Advisor checks are updated only **weekly**. Consequently, there must be a function to refresh the Trusted Advisor checks if these checks are to be used to evaluate **more frequently than weekly**.

- `AWS Trusted Advisor` provides guidance for **FIVE** check categories:
  - Cost optimization
  - Performance
  - Security
  - Fault tolerance
  - Service limits

----------

## GuardDuty

- `GuardDuty` is a threat detection service that continuously monitors your AWS accounts, workloads for malicious activity and delivers detailed security findings for visibility and remediation.
- Gain insight of compromised credentials, unusual data access in Amazon S3, API calls from known malicious IP addresses.
- It detects threat with Machine Learning.
- Track:
  - `CloudTrail Logs`
  - `VPC Flow Logs`
  - `DNS Query Logs`

Data protection: at rest
  - Amazon S3 server side encryption
    - SSE-S3: encrypts using keys handled & managed by AWS
    - SSE-KMS: encrypts using keys handled & managed by KMS
    - SSE-C: encrypts using keys handled & managed by Customer
    - Can be enabled by default (manually)
  - Amazon EBS either server side or host
  - Amazon Glacier is by default
  - Amazon EFS supports encryption only thru AWS KMS

----------

## Secrets Manager

- `Secrets Manager` has additional charges compared to `Parameter Store`.

  
----------

## Auto Scaling Group (ASG)

- Template
  - `Launch Template` allows a combination of On-Demand and Spot instances.
  - We can create versions for Launch Template and inherit configurations from other templates as well.
  - It is the future of ASG. We should move away from `Launch Configuration`.

-  Schedule Actions
  - We can schedule in advance how ASG should behave.

- Dynamic Scaling Policy
  - Both `Target tracking scaling`, `Simple scaling`, `Step scaling` use `CloudWatch Alarm` behind the scene.

- Target Group
  - It takes care of health check and monitor instances.
  - It also controls attributes such as `Slow start`, `Load balancing algorithm` and `Stickiness`.

- (IMPORTANT!!!) Types of processes that can be suspended and resumed for an Auto Scaling group
  - https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-suspend-resume-processes.html?icmpid=docs_ec2as_help_panel#process-types
  
- `Detach` and `Standby` are different.
  - Detach instance will remove an instance from ASG. (https://docs.aws.amazon.com/autoscaling/ec2/userguide/detach-instance-asg.html)
    - ASG will scale out if the minimum capacity is the same.
    - The instance can be attached to a different ASG.
  - Put an instance into `StandBy` state. (https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-enter-exit-standby.html)
    - Instances that are on `Standby` are still part of the ASG, but they do not actively handle load balancer traffic.
    - This feature helps you stop and start the instances or reboot them without worrying about ASG terminating the instances as part of its health checks or during scale-in events.

- (IMPORTANT!!!) ASG Lifecycle Hooks 
  - https://docs.aws.amazon.com/autoscaling/ec2/userguide/lifecycle-hooks-overview.html
  - https://docs.aws.amazon.com/autoscaling/ec2/userguide/warm-pool-instance-lifecycle.html#lifecycle-state-transitions
  - https://github.com/aws-samples/aws-lambda-lifecycle-hooks-function

- Termination Policies
  - https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-termination-policies.html#default-termination-policy

- Scale-in Protection
  - https://aws.amazon.com/about-aws/whats-new/2015/12/protect-instances-from-termination-by-auto-scaling/
    - For example, EC2 -> SQS. Before an EC2 instance process a message, call scale-in protection API.

- CloudFormation Update
  - https://aws.amazon.com/premiumsupport/knowledge-center/auto-scaling-group-rolling-updates/

- CodeDeploy
  - https://docs.aws.amazon.com/codedeploy/latest/userguide/integrations-aws-auto-scaling.html

- Which health checks can an Auto Scaling group use to determine the health of its instances?
  - Instances are assumed to be healthy unless Amazon EC2 Auto Scaling receives notification that they are unhealthy, which would come from `EC2`, `Elastic Load Balancing`, or `custom health checks`.
  - When it determines that an `InService` instance is unhealthy, it terminates that instance and launches a new one.

- `RetainResource`
  - If a `CloudFormation` stack has failed because of one resource, you can set the `RetainResource` parameter for the offending resource, and then delete the stack. This will bypass the problem resource and will allow the other resources, and ultimately the stack, to be deleted.

----------

## AWS Security Hub 

- When you enable `AWS Security Hub`, it begins to consume, aggregate, organize, and prioritize findings from AWS services that you have enabled, such as `Amazon GuardDuty`, `Amazon Inspector`, and `Amazon Macie`.

- `AWS Security Hub` is used to provide a comprehensive view of security alerts and security posture across your AWS accounts. With Security Hub, you have a single place that aggregates, organizes, and prioritizes your security alerts, or findings, from multiple AWS services, such as `Amazon GuardDuty`, `Amazon Inspector`, `Amazon Macie`, `AWS Identity and Access Management (IAM) Access Analyzer`, `AWS Systems Manager`, and `AWS Firewall Manager`, as well as from AWS Partner Network (APN) solutions.

----------

## Aurora

- Amazon Aurora can have up to 15 replicas.

----------

## EventBridge

- `EventBridge` vs. `SNS`: https://medium.com/awesome-cloud/aws-difference-between-amazon-eventbridge-and-amazon-sns-comparison-aws-eventbridge-vs-aws-sns-46708bf5313

- Events Rules will match incoming events and route them to the target.
  - An `event` indicates a change in an environment. A `rule` matches incoming events and sends them to `targets` for processing.

- Message size: 256 KB

----------

## SNS

- Support **FAN-OUT** pattern.

- Message size: 256 KB

- An `EC2` instance cannot subscribe to an `SNS` message. A `Lambda` function can subscribe to an `SNS` message.

----------

## DynamoDB

- `DynamoDB` is the only option that supports multi-region replication and multi-master writes, and it does this using `Global Tables`.
  - https://aws.amazon.com/dynamodb/global-tables/

- `Point-in-time recovery` is not an archival solution because it retains the data for only **35** days. 

- `DynamoDB Streams` is integrated with `AWS Lambda` (only option) to create triggers - pieces of code that automatically respond to events in DynamoDB Streams.
  - Automatically Archive Items to S3 Using DynamoDB Time to Live (TTL) with AWS Lambda and Amazon Kinesis Firehose: https://aws.amazon.com/blogs/database/automatically-archive-items-to-s3-using-dynamodb-time-to-live-with-aws-lambda-and-amazon-kinesis-firehose/

----------

## AWS Config

- Audit and ensure resource compliance over time

- Remediate noncompliant resources with `CloudWatch Events` or native integration with `AWS Systems Manager Automation` in Rules' settings.

- (IMPORTANT!!!) Event Types in Config: https://docs.aws.amazon.com/config/latest/developerguide/monitor-config-with-cloudwatchevents.html#create-cloudwatch-events-rule-for-awsconfig
  - **Config Configuration Item Change** vs. **Config Rules Compliance Change**

- SNS Topic is used for the whole Config (for operational insights). There is no SNS notification at Rule level. You should use `CloudWatch Events` for Rules.

- You can develop custom rules by associate each custom rule with an AWS Lambda function.

----------

## Elastic Container Service

- There is no option to create **Notifications** within the ECS cluster options. That notification functionality can be used with `EventBridge` (CloudWatch Events).

----------

## Simple Storage Service (S3)

- S3 `Lifecycle` policy can include a `filter` rule to delete objects with certain tags, key prefix or a combination of both.

----------

## Service Quotas

- Go to Service Quotas console and select a quota then create a CloudWatch Alarm.
  - https://docs.aws.amazon.com/servicequotas/latest/userguide/configure-cloudwatch.html

----------

## Misc.

- To ensure that no security credentials are ever commited to the code repository, use `git-secrets` as a pre-commit hook. https://github.com/awslabs/git-secrets

- For distributed load testing:
  - It's better to use `Fargate` and `ECS` to scale for each test scenario
  - https://aws.amazon.com/solutions/implementations/distributed-load-testing-on-aws/

- `EBS` vs. `EFS` vs. `S3`
  - `EBS` is a high-performance per-instance block storage system designed to act as storage for a single EC2 instance (most of the time).
  - `EFS` is a highly scalable file storage system designed to provide flexible storage for **multiple EC2 instances**.
  - `S3` is an object storage system, designed to provide archiving and data control options and to interface with other services **beyond** EC2. It’s also useful for storing static html pages and shared storage for applications.

- AWS Batch has a delay of an hour?

- The AWS CLI credentials and configuration settings take precedence in the following order: https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-precedence

- An `STS` Token expires after 1 hour.
