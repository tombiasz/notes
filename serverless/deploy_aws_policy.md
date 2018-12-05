# deploy serverless app

Create `IAM` user for each service and generate `aws` `policy` with `yeoman`

## 1. Create dedicated IAM user

- Name: `serverless-deployer`
- Access Type: `Programmatic access`
- no permissions

## 2. Register new profile with aws cli

```
aws configure --profile serverless`

# validate
cat ~/.aws/config
cat ~/.aws/credentials
```

## 3. Create serverless app

```
serverless create --template hello-world
```

## 4. Create aws policy for your app

```
npm install -g yo generator-serverless-policy
yo serverless-policy
```

provided policy configuration:
```
? Your Serverless service name serverless-hello-world
? You can specify a specific stage, if you like: *
? You can specify a specific region, if you like: *
? Does your service rely on DynamoDB? No
? Is your service going to be using S3 buckets? No
```

:warning: service name must the same as `service` in `serverless.yml` file

`cat serverless-hello-world-_star_-_star_-policy.json`
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "cloudformation:List*",
        "cloudformation:Get*",
        "cloudformation:PreviewStackUpdate",
        "cloudformation:ValidateTemplate"
      ],
      "Resource": [
        "*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "cloudformation:CreateStack",
        "cloudformation:CreateUploadBucket",
        "cloudformation:DeleteStack",
        "cloudformation:Describe*",
        "cloudformation:UpdateStack"
      ],
      "Resource": [
        "arn:aws:cloudformation:*:*:stack/serverless-hello-world-*/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "lambda:Get*",
        "lambda:List*",
        "lambda:CreateFunction"
      ],
      "Resource": [
        "*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:CreateBucket",
        "s3:DeleteBucket",
        "s3:ListBucket",
        "s3:ListBucketVersions"
      ],
      "Resource": [
        "arn:aws:s3:::serverless-hello-world*serverlessdeploy*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:DeleteObject"
      ],
      "Resource": [
        "arn:aws:s3:::serverless-hello-world*serverlessdeploy*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "lambda:AddPermission",
        "lambda:CreateAlias",
        "lambda:DeleteFunction",
        "lambda:InvokeFunction",
        "lambda:PublishVersion",
        "lambda:RemovePermission",
        "lambda:Update*"
      ],
      "Resource": [
        "arn:aws:lambda:*:*:function:serverless-hello-world-*-*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "apigateway:GET",
        "apigateway:POST",
        "apigateway:PUT",
        "apigateway:DELETE"
      ],
      "Resource": [
        "arn:aws:apigateway:*::/restapis*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "iam:PassRole"
      ],
      "Resource": [
        "arn:aws:iam::*:role/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": "kinesis:*",
      "Resource": [
        "arn:aws:kinesis:*:*:stream/serverless-hello-world-*-*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "iam:GetRole",
        "iam:CreateRole",
        "iam:PutRolePolicy",
        "iam:DeleteRolePolicy",
        "iam:DeleteRole"
      ],
      "Resource": [
        "arn:aws:iam::*:role/serverless-hello-world-*-*-lambdaRole"
      ]
    },
    {
      "Effect": "Allow",
      "Action": "sqs:*",
      "Resource": [
        "arn:aws:sqs:*:*:serverless-hello-world-*-*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "cloudwatch:GetMetricStatistics"
      ],
      "Resource": [
        "*"
      ]
    },
    {
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:DeleteLogGroup"
      ],
      "Resource": [
        "arn:aws:logs:*:*:*"
      ],
      "Effect": "Allow"
    },
    {
      "Action": [
        "logs:PutLogEvents"
      ],
      "Resource": [
        "arn:aws:logs:*:*:*"
      ],
      "Effect": "Allow"
    },
    {
      "Effect": "Allow",
      "Action": [
        "logs:DescribeLogStreams",
        "logs:DescribeLogGroups",
        "logs:FilterLogEvents"
      ],
      "Resource": [
        "*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "events:Put*",
        "events:Remove*",
        "events:Delete*"
      ],
      "Resource": [
        "arn:aws:events:*:*:rule/serverless-hello-world-*-*"
      ]
    }
  ]
}
```

## 5 Add policy to user

Go to AWS console and create policy from generated json and attach it to the user

## 6 Deploy

```
AWS_PROFILE=serverless serverless deploy
```

:tada:

## Notes
1. You can also create a `IAM` user with `AdministratorAccess` policy but it's not secure
2. You don't have to use aws profiles

## Refs:
- https://github.com/dancrumb/generator-serverless-policy
- https://docs.aws.amazon.com/cli/latest/userguide/cli-multiple-profiles.html
