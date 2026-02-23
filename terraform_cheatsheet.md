# terraform_cheatsheet

# CORE CONCEPTS

---

Provider       → Plugin for cloud/service (AWS, GCP, Azure, etc.)
Resource       → Infrastructure object to manage (EC2, S3, VPC, etc.)
Data Source    → Read-only query of existing infrastructure
Module         → Reusable group of resources
State          → terraform.tfstate file tracking real-world resources
Plan           → Preview of changes before applying
Workspace      → Isolated state environments (dev/staging/prod)
Backend        → Where state is stored (local, S3, Terraform Cloud)

---

## CLI WORKFLOW

terraform init                     → Initialize project, download providers
terraform init -upgrade            → Upgrade provider versions
terraform fmt                      → Auto-format .tf files
terraform validate                 → Check syntax & config validity
terraform plan                     → Preview changes (dry run)
terraform plan -out=tfplan         → Save plan to file
terraform apply                    → Apply changes (prompts confirmation)
terraform apply -auto-approve      → Apply without confirmation prompt
terraform apply tfplan             → Apply from saved plan file
terraform destroy                  → Destroy all managed resources
terraform destroy -auto-approve    → Destroy without confirmation
terraform refresh                  → Sync state with real infrastructure
terraform output                   → Show output values
terraform output output_name       → Show specific output value

---

## STATE MANAGEMENT

terraform show                     → Human-readable state
terraform state list               → List all resources in state
terraform state show [resource.name](http://resource.name/) → Details of specific resource
terraform state mv old new         → Rename resource in state
terraform state rm [resource.name](http://resource.name/)   → Remove resource from state (no destroy)
terraform state pull               → Download remote state to stdout
terraform state push               → Upload local state to remote backend
terraform import [resource.name](http://resource.name/) id  → Import existing resource into state

---

## WORKSPACES

terraform workspace list           → List workspaces
terraform workspace show           → Current workspace
terraform workspace new staging    → Create new workspace
terraform workspace select staging → Switch workspace
terraform workspace delete staging → Delete workspace

---

## FILE STRUCTURE (BEST PRACTICE)

[main.tf](http://main.tf/)           → Primary resources
[variables.tf](http://variables.tf/)      → Input variable declarations
[outputs.tf](http://outputs.tf/)        → Output value declarations
[providers.tf](http://providers.tf/)      → Provider configuration
terraform.tfvars  → Variable values (gitignore secrets)
[backend.tf](http://backend.tf/)        → Remote state config

---

## PROVIDERS

# [providers.tf](http://providers.tf/)

terraform {
required_providers {
aws = {
source  = "hashicorp/aws"
version = "~> 5.0"
}
}
required_version = ">= 1.5.0"
}

provider "aws" {
region = var.aws_region
}

---

## VARIABLES

# [variables.tf](http://variables.tf/)

variable "instance_type" {
description = "EC2 instance type"
type        = string
default     = "t3.micro"
}

variable "allowed_cidrs" {
type    = list(string)
default = ["10.0.0.0/8"]
}

variable "tags" {
type = map(string)
default = {
env = "dev"
}
}

# terraform.tfvars

instance_type = "t3.small"
aws_region    = "us-east-1"

# Reference in code

resource "aws_instance" "web" {
instance_type = var.instance_type
}

# Override at CLI

terraform apply -var="instance_type=t3.large"
terraform apply -var-file="prod.tfvars"

---

## OUTPUTS

# [outputs.tf](http://outputs.tf/)

output "instance_ip" {
description = "Public IP of EC2 instance"
value       = aws_instance.web.public_ip
}

output "bucket_arn" {
value     = aws_s3_bucket.my_bucket.arn
sensitive = true          → Hides value in CLI output
}

---

## COMMON RESOURCES (AWS)

# EC2 Instance

resource "aws_instance" "web" {
ami           = "ami-0abcdef1234567890"
instance_type = var.instance_type
key_name      = aws_key_pair.my_key.key_name
tags = {
Name = "web-server"
Env  = "dev"
}
}

# S3 Bucket

resource "aws_s3_bucket" "my_bucket" {
bucket = "my-unique-bucket-name"
}

# Security Group

resource "aws_security_group" "web_sg" {
name = "web-sg"
ingress {
from_port   = 80
to_port     = 80
protocol    = "tcp"
cidr_blocks = ["0.0.0.0/0"]
}
egress {
from_port   = 0
to_port     = 0
protocol    = "-1"
cidr_blocks = ["0.0.0.0/0"]
}
}

# VPC

resource "aws_vpc" "main" {
cidr_block           = "10.0.0.0/16"
enable_dns_hostnames = true
tags = { Name = "main-vpc" }
}

# Subnet

resource "aws_subnet" "public" {
vpc_id            = aws_vpc.main.id
cidr_block        = "10.0.1.0/24"
availability_zone = "us-east-1a"
}

---

## DATA SOURCES

# Look up existing AMI

data "aws_ami" "ubuntu" {
most_recent = true
owners      = ["099720109477"]    → Canonical
filter {
name   = "name"
values = ["ubuntu/images/hvm-ssd/ubuntu-*-22.04-amd64-server-*"]
}
}

resource "aws_instance" "web" {
ami = data.aws_ami.ubuntu.id
}

---

## EXPRESSIONS & FUNCTIONS

# String interpolation

name = "web-${var.env}"

# Conditionals

instance_type = var.env == "prod" ? "t3.large" : "t3.micro"

# Loops

resource "aws_iam_user" "users" {
for_each = toset(["alice", "bob", "carol"])
name     = each.key
}

# count

resource "aws_instance" "web" {
count         = 3
ami           = data.aws_ami.ubuntu.id
instance_type = "t3.micro"
tags = { Name = "web-${count.index}" }
}

# Common functions

length(var.list)
join(",", var.list)
split(",", "a,b,c")
toset(["a","b","a"])
lookup(var.map, "key", "default")
file("path/to/file")
templatefile("tmpl.tpl", { var = "val" })
cidrsubnet("10.0.0.0/16", 8, 1)      → "10.0.1.0/24"

---

## REMOTE STATE BACKEND (S3)

# [backend.tf](http://backend.tf/)

terraform {
backend "s3" {
bucket         = "my-tf-state-bucket"
key            = "prod/terraform.tfstate"
region         = "us-east-1"
dynamodb_table = "tf-state-lock"      → Enable state locking
encrypt        = true
}
}

---

## MODULES

# Call a module

module "vpc" {
source     = "./modules/vpc"           → Local module
cidr_block = "10.0.0.0/16"
}

module "vpc" {
source  = "terraform-aws-modules/vpc/aws"   → Registry module
version = "5.0.0"
cidr    = "10.0.0.0/16"
}

# Reference module output

resource "aws_instance" "web" {
subnet_id = module.vpc.public_subnet_id
}

---

## DEBUGGING

TF_LOG=DEBUG terraform apply          → Full debug logging
TF_LOG=INFO terraform plan            → Info-level logging
TF_LOG_PATH=tf.log terraform apply    → Write logs to file
terraform console                     → Interactive expression tester