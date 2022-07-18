### Code Commit

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

