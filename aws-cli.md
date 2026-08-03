# AWS CLI Cheat Sheet

## Setup & config
```bash
aws configure                          # set access key, secret, region, output format
aws configure --profile prod             # named profile
aws configure list                         # show current config
aws sts get-caller-identity                  # who am I / which account
export AWS_PROFILE=prod
export AWS_REGION=us-east-1
aws --version
```

## S3
```bash
aws s3 ls                              # list buckets
aws s3 ls s3://mybucket/                 # list objects in a bucket
aws s3 cp file.txt s3://mybucket/          # upload
aws s3 cp s3://mybucket/file.txt .           # download
aws s3 sync ./localdir s3://mybucket/dir       # sync directory (upload only changed files)
aws s3 sync s3://mybucket/dir ./localdir         # sync down
aws s3 rm s3://mybucket/file.txt
aws s3 rb s3://mybucket --force                    # remove bucket + all contents
aws s3 mb s3://mybucket                              # make bucket
aws s3api head-object --bucket mybucket --key file.txt   # metadata without downloading
aws s3 presign s3://mybucket/file.txt --expires-in 3600     # generate a temporary URL
```

## EC2
```bash
aws ec2 describe-instances
aws ec2 describe-instances --query "Reservations[].Instances[].[InstanceId,State.Name,PublicIpAddress]" --output table
aws ec2 start-instances --instance-ids i-0123456789
aws ec2 stop-instances --instance-ids i-0123456789
aws ec2 terminate-instances --instance-ids i-0123456789
aws ec2 describe-security-groups
aws ec2 describe-instance-status
aws ec2 create-tags --resources i-0123456789 --tags Key=Name,Value=web-1
```

## IAM
```bash
aws iam list-users
aws iam list-roles
aws iam get-user
aws iam list-attached-user-policies --user-name chris
aws iam create-access-key --user-name chris
aws iam attach-user-policy --user-name chris --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
aws iam simulate-principal-policy --policy-source-arn <arn> --action-names s3:GetObject
```

## Logs (CloudWatch)
```bash
aws logs describe-log-groups
aws logs tail /aws/lambda/myfunction --follow             # stream logs live
aws logs filter-log-events --log-group-name /aws/lambda/myfunction --filter-pattern "ERROR"
```

## Lambda
```bash
aws lambda list-functions
aws lambda invoke --function-name myfunc out.json
aws lambda update-function-code --function-name myfunc --zip-file fileb://function.zip
aws lambda get-function --function-name myfunc
```

## ECS / EKS
```bash
aws ecs list-clusters
aws ecs list-tasks --cluster mycluster
aws ecs describe-services --cluster mycluster --services myservice
aws eks list-clusters
aws eks update-kubeconfig --name mycluster --region us-east-1     # wire up kubectl for an EKS cluster
```

## Useful one-liners
```bash
aws sts get-caller-identity --query Account --output text                  # just the account ID
aws ec2 describe-instances --filters "Name=instance-state-name,Values=running" --query "Reservations[].Instances[].InstanceId" --output text
aws s3 ls --summarize --human-readable --recursive s3://mybucket/           # total size of a bucket
aws configure list-profiles                                                    # list all configured profiles
aws ec2 describe-regions --query "Regions[].RegionName" --output table            # list all regions
aws cloudformation describe-stacks --query "Stacks[].StackName" --output table       # list stacks
```
