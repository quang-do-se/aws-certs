# Exam tips

Ignore "You do not have to do anything" answer. It will always be wrong.

"You want to focus on the simplest, most technically correct answer."

We wanna automate all aspects of our environment. We wanna use that infrastructure as code mentality. **Manual step isn't going to be the right answer**. We wanna avoide **manual steps** in any of the scenarios that were given on the exam.

When routing **portions of users**, always choose Amazon Route 53.

----------

# Domain 1

CodeBuild
  - buildspec.yml

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

# Domain 2

`WaitCondtion` vs `CreationPolicy`
  - `WaitCondition` is a resource.
  - `CreationPolicy` is an attribute.

We cannot do a rolling update to `Auto Scaling` unless we are utililzing `CloudFormation`. (`UpdatePolicy` attribute under `AutoScalingGroup` resource)

Updates to all resources are open by default but `Stack Policy` changes to DENY for all resources once created.

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
