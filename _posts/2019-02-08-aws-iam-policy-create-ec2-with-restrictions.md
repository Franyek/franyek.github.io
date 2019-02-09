---
layout: post
title: "AWS IAM Policy: EC2 instance creation with restrictions "
date: 2019-02-08
---

Used resources:
   *  [IAM Policy tags and variables](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_variables.html)
   *  [Example policies](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ExamplePolicies_EC2.html)
   *  [Restrict resource tags in IAM policy](https://aws.amazon.com/premiumsupport/knowledge-center/iam-policy-tags-restrict)
   *  [How to deploy an EC2 from CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-tutorial.html?shortFooter=true#tutorial-configure-security)

Example CLI commands for testing the instance deployment:
```
aws ec2 run-instances --security-group-ids <group-id> --count 1 --instance-type t2.micro --key-name devenv-key --query "Instances[0].InstanceId" --image-id <image-id>
aws ec2 run-instances --security-group-ids <group-id> --count 1 --instance-type t2.micro --key-name devenv-key --query "Instances[0].InstanceId" --image-id <image-id> --tag-specifications ResourceType=instance,Tags=[{Key=owner,Value=key2}] ResourceType=volume,Tags=[{Key=owner,Value=key2}]
aws ec2 run-instances --security-group-ids <group-id> --count 1 --instance-type t2.micro --key-name devenv-key --query "Instances[0].InstanceId" --image-id <image-id> --tag-specifications ResourceType=instance,Tags=[{Key=owner,Value=key1},{Key=name,Value=key1},{Key=unit,Value=key1},{Key=email,Value=key1}] ResourceType=volume,Tags=[{Key=owner,Value=key1},{Key=name,Value=key1},{Key=unit,Value=key1},{Key=email,Value=key1}]
aws ec2 run-instances --security-group-ids <group-id> --count 1 --instance-type t2.micro --key-name devenv-key --query "Instances[0].InstanceId" --image-id <image-id> --tag-specifications ResourceType=instance,Tags=[{Key=owner,Value=key1},{Key=name,Value=key1},{Key=unit,Value=key1}] ResourceType=volume,Tags=[{Key=owner,Value=key1},{Key=name,Value=key1},{Key=unit,Value=key1}]
aws ec2 run-instances --security-group-ids <group-id> --count 1 --instance-type t2.micro --key-name devenv-key --query "Instances[0].InstanceId" --image-id <image-id> --tag-specifications ResourceType=instance,Tags=[{Key=owner,Value=key1},{Key=unit,Value=key1}] ResourceType=volume,Tags=[{Key=owner,Value=key1},{Key=unit,Value=key1}]
```

How to decompose the error messages from CLI:
```
aws sts decode-authorization-message --encoded-message <msg>
```

Half ready policy:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowToDescribeAll",
            "Effect": "Allow",
            "Action": [
                "ec2:Describe*"
            ],
            "Resource": "*"
        },
        {
            "Sid": "AllowRunInstances",
            "Effect": "Allow",
            "Action": "ec2:RunInstances",
            "Resource": [
                "arn:aws:ec2:*::image/*",
                "arn:aws:ec2:*::snapshot/*",
                "arn:aws:ec2:*:*:subnet/*",
                "arn:aws:ec2:*:*:network-interface/*",
                "arn:aws:ec2:*:*:security-group/*",
                "arn:aws:ec2:*:*:key-pair/*"
            ]
        },
        {
            "Sid": "AllowRunInstancesWithRestrictions",
            "Effect": "Allow",
            "Action": [
                "ec2:CreateVolume",
                "ec2:RunInstances"
            ],
            "Resource": [
                "arn:aws:ec2:*:*:volume/*",
                "arn:aws:ec2:*:*:instance/*"
            ],
            "Condition": {
                "StringEquals": {
                    "aws:RequestTag/owner": "${aws:username}"
                },
                "ForAllValues:StringEquals": {
                    "aws:TagKeys": [
                        "Name",
                        "unit",
                        "email"
                    ]
                }
            }
        },
        {
            "Sid": "AllowCreateTagsOnlyLaunching",
            "Effect": "Allow",
            "Action": [
                "ec2:CreateTags"
            ],
            "Resource": [
                "arn:aws:ec2:*:*:volume/*",
                "arn:aws:ec2:*:*:instance/*"
            ],
            "Condition": {
                "StringEquals": {
                    "ec2:CreateAction": "RunInstances"
                }
            }
        }
    ]
}
```