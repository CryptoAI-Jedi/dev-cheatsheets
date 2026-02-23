# aws_cli_cheatsheet

# SETUP & CONFIG

---

aws configure                              → Set access key, secret, region, output
aws configure list                         → Show current config
aws configure list-profiles                → List named profiles
aws configure --profile myprofile          → Configure named profile
AWS_PROFILE=myprofile aws s3 ls           → Use named profile inline
aws sts get-caller-identity                → Verify who you're authenticated as

---

## EC2 — INSTANCES

aws ec2 describe-instances                 → List all instances
aws ec2 describe-instances --filters "Name=instance-state-name,Values=running"
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,[State.Name](http://state.name/),PublicIpAddress]' --output table
aws ec2 start-instances --instance-ids i-0abc123
aws ec2 stop-instances --instance-ids i-0abc123
aws ec2 reboot-instances --instance-ids i-0abc123
aws ec2 terminate-instances --instance-ids i-0abc123
aws ec2 describe-instance-status --instance-ids i-0abc123

# Launch instance

aws ec2 run-instances \
--image-id ami-0abcdef1234567890 \
--count 1 \
--instance-type t3.micro \
--key-name my-key \
--security-group-ids sg-0abc123 \
--subnet-id subnet-0abc123

---

## EC2 — KEY PAIRS & SECURITY GROUPS

aws ec2 describe-key-pairs                 → List key pairs
aws ec2 create-key-pair --key-name my-key --query 'KeyMaterial' --output text > my-key.pem
aws ec2 delete-key-pair --key-name my-key

aws ec2 describe-security-groups           → List security groups
aws ec2 create-security-group --group-name my-sg --description "My SG" --vpc-id vpc-0abc
aws ec2 authorize-security-group-ingress --group-id sg-0abc --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 delete-security-group --group-id sg-0abc

---

## S3 — BUCKETS & FILES

aws s3 ls                                  → List all buckets
aws s3 ls s3://bucket-name                 → List bucket contents
aws s3 ls s3://bucket-name --recursive     → List all files recursively
aws s3 mb s3://my-new-bucket               → Create bucket
aws s3 rb s3://my-bucket --force           → Delete bucket (inc. contents)

aws s3 cp file.txt s3://bucket/path/       → Upload file
aws s3 cp s3://bucket/path/file.txt .      → Download file
aws s3 cp s3://bucket/ . --recursive       → Download entire bucket
aws s3 mv file.txt s3://bucket/            → Move/rename
aws s3 rm s3://bucket/file.txt             → Delete file
aws s3 rm s3://bucket/ --recursive         → Delete all files
aws s3 sync ./local s3://bucket/prefix     → Sync local to S3
aws s3 sync s3://bucket/prefix ./local     → Sync S3 to local
aws s3 presign s3://bucket/file.txt --expires-in 3600  → Temp URL (1hr)

---

## IAM — USERS & ROLES

aws iam list-users                         → List IAM users
aws iam get-user                           → Current user info
aws iam create-user --user-name alice
aws iam delete-user --user-name alice

aws iam list-roles                         → List IAM roles
aws iam list-attached-user-policies --user-name alice
aws iam attach-user-policy --user-name alice --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
aws iam detach-user-policy --user-name alice --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess

aws iam create-access-key --user-name alice
aws iam delete-access-key --user-name alice --access-key-id AKIA...
aws iam list-access-keys --user-name alice

---

## VPC & NETWORKING

aws ec2 describe-vpcs                      → List VPCs
aws ec2 describe-subnets                   → List subnets
aws ec2 describe-route-tables             → Routing tables
aws ec2 describe-internet-gateways        → Internet gateways
aws ec2 describe-nat-gateways             → NAT gateways

---

## CLOUDWATCH — LOGS & METRICS

aws logs describe-log-groups               → List log groups
aws logs describe-log-streams --log-group-name /aws/lambda/myfunction
aws logs get-log-events \
--log-group-name /aws/ec2/myapp \
--log-stream-name stream-name
aws logs tail /aws/lambda/myfunction --follow    → Stream live logs
aws cloudwatch list-metrics                → List available metrics
aws cloudwatch get-metric-statistics \
--namespace AWS/EC2 \
--metric-name CPUUtilization \
--dimensions Name=InstanceId,Value=i-0abc \
--statistics Average \
--period 300 \
--start-time 2024-01-01T00:00:00Z \
--end-time 2024-01-01T01:00:00Z

---

## ECS / EKS (BASICS)

aws ecs list-clusters                      → List ECS clusters
aws ecs list-services --cluster my-cluster → List services
aws ecs describe-services --cluster my-cluster --services my-service
aws ecs update-service --cluster my-cluster --service my-service --force-new-deployment

aws eks list-clusters                      → List EKS clusters
aws eks update-kubeconfig --name my-cluster → Add cluster to kubeconfig

---

## SSM — PARAMETER STORE

aws ssm get-parameter --name /my/param --with-decryption   → Get secret
aws ssm put-parameter --name /my/param --value "val" --type SecureString
aws ssm delete-parameter --name /my/param
aws ssm get-parameters-by-path --path /my/ --recursive --with-decryption

---

## OUTPUT FORMATTING

- -output table → Human-readable table
--output json → Full JSON (default)
--output text → Tab-delimited plain text
--query 'Key[*].Field' → JMESPath filter

# Useful query examples

- -query 'Instances[*].[InstanceId,PublicIpAddress]'
--query 'Buckets[*].Name'
--query 'Users[*].[UserName,UserId]'