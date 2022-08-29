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

- CodeBuild can sent notifications to SNS using CodeBuild triggers.

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

- List of lifecycle event hooks: https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file-structure-hooks.html#appspec-hooks-server
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

#### Best practices and use cases

- https://docs.aws.amazon.com/codepipeline/latest/userguide/best-practices.html

#### Invoke Lambda function

- https://docs.aws.amazon.com/codepipeline/latest/userguide/actions-invoke-lambda-function.html

#### Create a pipeline with CloudFormation

- https://docs.aws.amazon.com/codepipeline/latest/userguide/tutorials-cloudformation.html

#### Exercise - best practices to set up CodePipeline for production deployment

- https://github.com/aws-samples/codepipeline-nested-cfn

#### Reading - Implementing GitFlow Using AWS CodePipeline, AWS CodeCommit, AWS CodeBuild, and AWS CodeDeploy

- https://aws.amazon.com/blogs/devops/implementing-gitflow-using-aws-codepipeline-aws-codecommit-aws-codebuild-and-aws-codedeploy/

### CodeStar

The main file is `template.yml`. Use it to update your project:

- https://docs.aws.amazon.com/codestar/latest/userguide/templates.html#update-project
