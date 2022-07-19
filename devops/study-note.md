### Code Commit

#### Overview

- Repository similar to GitHub and BitBucket.

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

#### Use notifications and triggers and CloudWatch events to set up automation between CodeCommit and other services (Lambda, SNS, SQS...)

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

- CodeBuild can sent notification to SNS using CodeBuild triggers.
