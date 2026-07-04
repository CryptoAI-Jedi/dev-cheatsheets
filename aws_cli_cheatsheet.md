# AWS CLI Cheatsheet

> Drive AWS from the terminal: EC2, S3, IAM, CloudWatch, and SSM. `--query` (JMESPath) and `--output table` turn JSON walls into answers.

---

## Table of Contents
- [Setup & Config](#setup--config)
- [EC2 — Instances](#ec2--instances)
- [EC2 — Key Pairs & Security Groups](#ec2--key-pairs--security-groups)
- [S3 — Buckets & Files](#s3--buckets--files)
- [IAM — Users & Roles](#iam--users--roles)
- [VPC & Networking](#vpc--networking)
- [CloudWatch — Logs & Metrics](#cloudwatch--logs--metrics)
- [ECS / EKS (Basics)](#ecs--eks-basics)
- [SSM — Parameter Store](#ssm--parameter-store)
- [Output Formatting](#output-formatting)
- [Tips & Gotchas](#tips--gotchas)

---

## SETUP & CONFIG

```bash
aws configure                              # Set access key, secret, region, output
aws configure list                         # Show current config
aws configure list-profiles               # List named profiles
aws configure --profile myprofile          # Configure named profile
AWS_PROFILE=myprofile aws s3 ls            # Use named profile inline
aws sts get-caller-identity                # Verify who you're authenticated as
```

---

## EC2 — INSTANCES

```bash
aws ec2 describe-instances                 # List all instances
aws ec2 describe-instances --filters "Name=instance-state-name,Values=running"
aws ec2 describe-instances \
  --query 'Reservations[*].Instances[*].[InstanceId,State.Name,PublicIpAddress]' \
  --output table
aws ec2 start-instances --instance-ids i-0abc123
aws ec2 stop-instances --instance-ids i-0abc123
aws ec2 reboot-instances --instance-ids i-0abc123
aws ec2 terminate-instances --instance-ids i-0abc123   # PERMANENT — see Gotchas
aws ec2 describe-instance-status --instance-ids i-0abc123
```

### Launch instance

```bash
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --count 1 \
  --instance-type t3.micro \
  --key-name my-key \
  --security-group-ids sg-0abc123 \
  --subnet-id subnet-0abc123
```

```text
💡 Destructive/expensive EC2 commands accept --dry-run — validates
permissions and syntax without doing anything.
```

---

## EC2 — KEY PAIRS & SECURITY GROUPS

```bash
aws ec2 describe-key-pairs                 # List key pairs
aws ec2 create-key-pair --key-name my-key \
  --query 'KeyMaterial' --output text > my-key.pem
chmod 600 my-key.pem                       # Or SSH will refuse it
aws ec2 delete-key-pair --key-name my-key
```

```bash
aws ec2 describe-security-groups           # List security groups
aws ec2 create-security-group --group-name my-sg \
  --description "My SG" --vpc-id vpc-0abc
aws ec2 authorize-security-group-ingress --group-id sg-0abc \
  --protocol tcp --port 22 --cidr 203.0.113.0/24   # YOUR IP range — not 0.0.0.0/0 (see Gotchas)
aws ec2 delete-security-group --group-id sg-0abc
```

---

## S3 — BUCKETS & FILES

```bash
aws s3 ls                                  # List all buckets
aws s3 ls s3://bucket-name                 # List bucket contents
aws s3 ls s3://bucket-name --recursive     # List all files recursively
aws s3 mb s3://my-new-bucket               # Create bucket
aws s3 rb s3://my-bucket --force           # Delete bucket (inc. contents)
```

```bash
aws s3 cp file.txt s3://bucket/path/       # Upload file
aws s3 cp s3://bucket/path/file.txt .      # Download file
aws s3 cp s3://bucket/ . --recursive       # Download entire bucket
aws s3 mv file.txt s3://bucket/            # Move/rename
aws s3 rm s3://bucket/file.txt             # Delete file
aws s3 rm s3://bucket/ --recursive         # Delete all files
aws s3 sync ./local s3://bucket/prefix     # Sync local → S3 (changed files only)
aws s3 sync s3://bucket/prefix ./local     # Sync S3 → local
aws s3 sync ./local s3://bucket/ --delete --dryrun   # Mirror preview — rsync rules apply
aws s3 presign s3://bucket/file.txt --expires-in 3600   # Temp URL (1hr)
```

---

## IAM — USERS & ROLES

```bash
aws iam list-users                         # List IAM users
aws iam get-user                           # Current user info
aws iam create-user --user-name alice
aws iam delete-user --user-name alice
```

```bash
aws iam list-roles                         # List IAM roles
aws iam list-attached-user-policies --user-name alice
aws iam attach-user-policy --user-name alice \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
aws iam detach-user-policy --user-name alice \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
```

```bash
aws iam create-access-key --user-name alice     # Shown ONCE — store securely
aws iam list-access-keys --user-name alice
aws iam delete-access-key --user-name alice --access-key-id AKIA...
```

---

## VPC & NETWORKING

```bash
aws ec2 describe-vpcs                      # List VPCs
aws ec2 describe-subnets                   # List subnets
aws ec2 describe-route-tables              # Routing tables
aws ec2 describe-internet-gateways         # Internet gateways
aws ec2 describe-nat-gateways              # NAT gateways
```

---

## CLOUDWATCH — LOGS & METRICS

```bash
aws logs describe-log-groups               # List log groups
aws logs describe-log-streams --log-group-name /aws/lambda/myfunction
aws logs get-log-events \
  --log-group-name /aws/ec2/myapp \
  --log-stream-name stream-name
aws logs tail /aws/lambda/myfunction --follow    # Stream live logs
```

```bash
aws cloudwatch list-metrics                # List available metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-0abc \
  --statistics Average \
  --period 300 \
  --start-time 2026-07-01T00:00:00Z \
  --end-time 2026-07-01T01:00:00Z
```

---

## ECS / EKS (BASICS)

```bash
aws ecs list-clusters                      # List ECS clusters
aws ecs list-services --cluster my-cluster # List services
aws ecs describe-services --cluster my-cluster --services my-service
aws ecs update-service --cluster my-cluster --service my-service \
  --force-new-deployment                   # Redeploy (pull fresh image)
```

```bash
aws eks list-clusters                      # List EKS clusters
aws eks update-kubeconfig --name my-cluster   # Add cluster to kubeconfig
```

---

## SSM — PARAMETER STORE

```bash
aws ssm get-parameter --name /my/param --with-decryption   # Get secret
aws ssm put-parameter --name /my/param --value "val" --type SecureString
aws ssm delete-parameter --name /my/param
aws ssm get-parameters-by-path --path /my/ --recursive --with-decryption
```

---

## OUTPUT FORMATTING

```text
--output table            → Human-readable table
--output json             → Full JSON (default)
--output text             → Tab-delimited plain text (script-friendly)
--query 'Key[*].Field'    → JMESPath filter
--no-cli-pager            → Don't drop into less (scripts)
```

### Useful query examples

```bash
--query 'Reservations[*].Instances[*].[InstanceId,PublicIpAddress]'
--query 'Buckets[*].Name'
--query 'Users[*].[UserName,UserId]'
```

```text
💡 --filters runs SERVER-side (less data transferred); --query filters
CLIENT-side (shapes the output). Use both: filter first, query second.
For heavier JSON surgery, pipe --output json into jq (see jq sheet).
```

---

## TIPS & GOTCHAS

- **`--cidr 0.0.0.0/0` on port 22 is an open invitation** — scope SSH ingress to your IP/CIDR; if access needs to be flexible, that's what SSM Session Manager or a tailnet is for.
- **stop ≠ terminate** — stopped instances keep their EBS volumes and can restart; terminated instances are gone (and non-root volumes may auto-delete). Read the command twice.
- **Access keys display exactly once** — `create-access-key` output is your only look at the secret. Rotation order: create new → deploy → verify → delete old.
- **`s3 sync --delete` is a mirror** — same rules as rsync `--delete` (see rsync sheet): dry-run first, always.
- **Presigned URLs are bearer tokens** — anyone holding the URL has the access until expiry. Short `--expires-in`, treat like a secret.
- **`dnf`-style history doesn't exist, but CloudTrail does** — "who terminated that instance" is answered in CloudTrail, not the CLI history.
- **Set `--region` or suffer** — resources "missing" are usually sitting in another region. `aws configure get region` when confused.

---
*Last Updated: 2026-07*
