CodeBuild
  buildspec.yml

CodeDeploy hooks
  `appspec` file can be YML or JSON



Lambda time out after 15 minutes?

AWS Batch has a delay of an hour?

CloudFormation does not detect a new file has been uploaded to S3 unless one of these parameters change: - S3Bucket - S3Key (filename) - S3ObjectVersion

SQS size limit is 256KB
  - Files must be uploaded to S3 and a reference to them should be sent to SQS.

`Continuous Delivery` phase requires manual approval step. `Continuous Integration` and `Continuous Deployment` are fully automated.
  - https://aws.amazon.com/devops/continuous-integration/


To ensure that no security credentials are ever commited to the code repository, use `git-secrets` as a pre-commit hook. https://github.com/awslabs/git-secrets


Focus on IAM solution


`AWS CodeStart` and `template.yml`


`CodeBuild Triggers` allow you to schedule automated builds every hour, day, week or custom time.



# Exam tips

Ignore "You do not have to do anything" answer. It will always be wrong.

"You want to focus on the simplest, most technically correct answer."

We wanna automate all aspects of our environment. We wanna use that infrastructure as code mentality. **Manual step isn't going to be the right answer**. We wanna avoide **manual steps** in any of the scenarios that were given on the exam.
