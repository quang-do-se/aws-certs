# Exam tips

Ignore "You do not have to do anything" answer. It will always be wrong.

"You want to focus on the simplest, most technically correct answer."

We wanna automate all aspects of our environment. We wanna use that infrastructure as code mentality. **Manual step isn't going to be the right answer**. We wanna avoide **manual steps** in any of the scenarios that were given on the exam.

When routing **portions of users**, always choose Amazon Route 53.

When you see a question about reporting on API calls or logs, that's likely to be `AWS CloudTrail`.

Use `Roles` everywhere you can. Avoid permanent security credentials.

If you see the phrase **Network Instrusion Detection**, you should think of `Amazon GuardDuty`.

----------

# Review

- CloudFormation Helper Scripts
- Lambda SAM Framework
- EventBridge 
  - Event bus

----------

# Domain 1

CodeBuild
  - buildspec.yml
  - Validating AWS CodeCommit Pull Requests with AWS CodeBuild and AWS Lambda: https://github.com/aws-samples/aws-codecommit-pull-request-aws-codebuild
  
CodeDeploy hooks
  - `appspec` file can be YML or JSON.
  - List of lifecycle event hooks (IMPORTANT!!!): https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file-structure-hooks.html#reference-appspec-file-structure-hooks-list

CodeDeploy's different deployment modes: https://docs.aws.amazon.com/codedeploy/latest/userguide/deployment-configurations.html

Lambda time out after 15 minutes?

AWS Batch has a delay of an hour?

CloudFormation does not detect a new file has been uploaded to S3 unless one of these parameters change: - S3Bucket - S3Key (filename) - S3ObjectVersion

SQS size limit is 256KB
  - Files must be uploaded to S3 and a reference to them should be sent to SQS.

`Continuous Delivery` phase requires manual approval step. `Continuous Integration` and `Continuous Deployment` are fully automated.
  - https://aws.amazon.com/devops/continuous-integration/


To ensure that no security credentials are ever commited to the code repository, use `git-secrets` as a pre-commit hook. https://github.com/awslabs/git-secrets

CodeCommit's AWS manage policy `AWSCodeCommitPowerUser` allow users access to all the functionality of CodeCommit but it does NOT allow them to delete or create repositories.

Focus on IAM solution


`AWS CodeStart` and `template.yml`


`CodeBuild Triggers` allow you to schedule automated builds every hour, day, week or custom time.


(IMPORTANT!!!) https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.deploy-existing-version.html

----------

# Domain 2

`WaitCondtion` vs `CreationPolicy`
  - `WaitCondition` is a resource.
  - `CreationPolicy` is an attribute.
  - The `CreationPolicy` is recommended for `EC2` and `Auto Scaling`.

We cannot do a rolling update to `Auto Scaling` unless we are utililzing `CloudFormation`. (`UpdatePolicy` attribute under `AutoScalingGroup` resource)

In CloudFormation, Updates to all resources are open by default but `Stack Policy` changes to DENY for all resources once created. The `Stack Policy` is the IAM style policy statement which governs what can be changed and who can change it.

(IMPORTANT!!!) UPDATE_ROLLBACK_FAILED in CloudFormation: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/troubleshooting.html#troubleshooting-errors-update-rollback-failed
  - https://aws.amazon.com/blogs/devops/continue-rolling-back-an-update-for-aws-cloudformation-stacks-in-the-update_rollback_failed-state/

Empty S3 can always be rolled back.

Application Load Balancer weighted target groups: https://aws.amazon.com/blogs/devops/blue-green-deployments-with-application-load-balancer/
  - https://aws.amazon.com/blogs/aws/new-application-load-balancer-simplifies-deployment-with-weighted-target-groups/
  
We cannot set timed cutovers with `Route 53`.

We cannot use `Elastic Beanstalk` to migrate our applications. We can't import or export environments with `Elastic Beanstalk`.
We can't bring in resources that aren't previously created utilizing the Elastic Beanstalk service.

OpsWorks layers:
  - Load Balancer layer
  - Application Server layer
  - Database layer

OpsWork automatic instance scaling options:
  - With automatic load-based scaling, you can set thresholds for CPU, memory, or load to define when additional instances will be started. 
  - With automatic time-based scaling, you can define at what time of the day instances will be started and stopped.

`SAM` templates are an extension of `CloudFormation` templates and are written in YAML.

----------

# Domain 3

- Metric Retention
  - Metric is kept for a period of 15 months
  - Less granularity overtime

CloudWatch Events Rules will match incoming events and route them to the target.
  - An `event` indicates a change in an environment. A `rule` matches incoming events and sends them to `targets` for processing.
  
Cloudwatch Metrics are grouped into namespaces.

When the requirement is to process streaming data in real time, `Kinesis` must be strongly considered. "Real time" usually points to `Kinesis`.

`Kinesis Data Streams` is a low latency streaming service in AWS Kinesis with the facility for ingesting at scale. On the other hand, `Kinesis Firehose` aims to serve as a data transfer service.

The primary purpose of `Kinesis Firehose` focuses on loading streaming data to Amazon S3, Splunk, ElasticSearch, and RedShift. 

----------

# Domain 4 - Policies and Standards Automation

Optimize cost through automation.

`IAM`
  - Users are permanent set of credentials
  - Roles are temporary credentials and NOT something we need to rotate because it happens automatically behind the scenes.
  - Cannot apply restriction conditions on root account
  - Clean up IAM users for those who left

`Security Group`
  - Stateful
  - For **Instance** level

`Network Access Control List (NACLs)`
  - Stateless
  - For **Subnet** level

Data protection: at rest
  - Amazon S3 server side encryption
    - SSE-S3: encrypts using keys handled & managed by AWS
    - SSE-KMS: encrypts using keys handled & managed by KMS
    - SSE-C: encrypts using keys handled & managed by Customer
    - Can be enabled by default (manually)
  - Amazon EBS either server side or host
  - Amazon Glacier is by default
  - Amazon EFS supports encryption only thru AWS KMS

`GuardDuty`:
  - CloudTrail events
  - VPC Flow Logs
  - DNS Query Logs

`Secrets Manager` has additional charges compared to `Parameter Store`.

`AWS Trusted Advisor` provides guidance to help you:
  - Reduce cost
  - Increase performance
  - Improve security
  
----------

# Domain 5 - Incident and Event Response

You can't delete a `Load Balancer` if deletion protection is enabled.

Auto Scaling `detach` and `standby` mode are different.

Route 53 routing policy: https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html

Amazon Aurora can have up to 15 replicas.

When you enable `AWS Security Hub`, it begins to consume, aggregate, organize, and prioritize findings from AWS services that you have enabled, such as `Amazon GuardDuty`, `Amazon Inspector`, and `Amazon Macie`.

If a `CloudFormation` stack has failed because of one resource, you can set the `RetainResource` parameter for the offending resource, and then delete the stack. This will bypass the problem resource and will allow the other resources, and ultimately the stack, to be deleted.


`EventBridge` vs. `SNS`: https://medium.com/awesome-cloud/aws-difference-between-amazon-eventbridge-and-amazon-sns-comparison-aws-eventbridge-vs-aws-sns-46708bf5313

----------

# Domain 6 - High Availability, Fault Tolerance, and Disaster Recovery
