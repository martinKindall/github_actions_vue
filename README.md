# Github actions and Vue 3 on AWS S3

This is a toy example on how to build a minimal CI/CD pipeline for a Vue project deployed in AWS S3.

## Github OIDC and AWS IAM

You must allow Github OIDC as authentication method in the AWS IAM console in order to assume roles from github actions.
More information [here](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services) and [there](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-idp_oidc.html).

## IAM Role Trust Policy

Every role in IAM has a __trust relationship__, which specifies the entities that are allowed to assume the role.
The following policy is used for the current actions role. Note that the wildcard used after _repository_ allows the github action to assume on any branch of this repository.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::<aws-account-number>:oidc-provider/token.actions.githubusercontent.com"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringLike": {
                    "token.actions.githubusercontent.com:sub": "repo:<org-or-username>/<repository>:*"
                },
                "StringEquals": {
                    "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
                }
            }
        }
    ]
}
```

## IAM Policy

The S3 bucket has a read only IAM policy like this

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::<bucket-name>/*"
        }
    ]
}
```

The IAM role specified in __role-to-assume__ has only the following permissions policy:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:PutObjectAcl",
                "s3:GetObject",
                "s3:GetObjectAcl",
                "s3:DeleteObject",
                "s3:ListMultipartUploadParts",
                "s3:AbortMultipartUpload",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::<bucket-name>/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::<bucket-name>"
            ]
        }
    ]
}
```

It is important to follow the [_least privilege permissions principle_](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege), hence use a dedicated IAM user just for this task and only with the minimal permissions.