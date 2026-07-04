# Terraform Cheatsheet

> Declarative infrastructure-as-code: describe resources in HCL, `plan` shows the diff, `apply` makes reality match. The state file is the source of truth; treat it accordingly.

---

## Table of Contents
- [Core Concepts](#core-concepts)
- [CLI Workflow](#cli-workflow)
- [State Management](#state-management)
- [Workspaces](#workspaces)
- [File Structure](#file-structure)
- [Providers](#providers)
- [Variables](#variables)
- [Outputs](#outputs)
- [Common Resources (AWS)](#common-resources-aws)
- [Data Sources](#data-sources)
- [Expressions & Functions](#expressions--functions)
- [Remote State Backend (S3)](#remote-state-backend-s3)
- [Modules](#modules)
- [Debugging](#debugging)
- [Tips & Gotchas](#tips--gotchas)

---

## CORE CONCEPTS

```text
Provider       → Plugin for cloud/service (AWS, GCP, Azure, etc.)
Resource       → Infrastructure object to manage (EC2, S3, VPC, etc.)
Data Source    → Read-only query of existing infrastructure
Module         → Reusable group of resources
State          → terraform.tfstate file tracking real-world resources
Plan           → Preview of changes before applying
Workspace      → Isolated state environments (dev/staging/prod)
Backend        → Where state is stored (local, S3, Terraform Cloud)
```

---

## CLI WORKFLOW

```bash
terraform init                     # Initialize project, download providers
terraform init -upgrade            # Upgrade provider versions
terraform fmt                      # Auto-format .tf files
terraform validate                 # Check syntax & config validity
terraform plan                     # Preview changes (dry run)
terraform plan -out=tfplan         # Save plan to file
terraform apply                    # Apply changes (prompts confirmation)
terraform apply tfplan             # Apply EXACTLY the saved plan
terraform apply -auto-approve      # Apply without prompt (CI only)
terraform destroy                  # Destroy all managed resources
terraform output                   # Show output values
terraform output output_name       # Show specific output value
```

---

## STATE MANAGEMENT

```bash
terraform show                       # Human-readable state
terraform state list                 # List all resources in state
terraform state show resource.name   # Details of specific resource
terraform state mv old new           # Rename resource in state (no destroy/recreate)
terraform state rm resource.name     # Remove from state (resource keeps existing)
terraform state pull                 # Download remote state to stdout
terraform import resource.name id    # Adopt existing infra into state
```

---

## WORKSPACES

```bash
terraform workspace list           # List workspaces
terraform workspace show           # Current workspace
terraform workspace new staging    # Create new workspace
terraform workspace select staging # Switch workspace
terraform workspace delete staging # Delete workspace
```

---

## FILE STRUCTURE

```text
main.tf           → Primary resources
variables.tf      → Input variable declarations
outputs.tf        → Output value declarations
providers.tf      → Provider configuration
terraform.tfvars  → Variable values (gitignore if secrets)
backend.tf        → Remote state config
.terraform.lock.hcl → Provider version lock — COMMIT this
```

---

## PROVIDERS

```hcl
# providers.tf
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
```

---

## VARIABLES

```hcl
# variables.tf
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
```

```hcl
# terraform.tfvars
instance_type = "t3.small"
aws_region    = "us-east-1"
```

```hcl
# Reference in code
resource "aws_instance" "web" {
  instance_type = var.instance_type
}
```

```bash
# Override at CLI
terraform apply -var="instance_type=t3.large"
terraform apply -var-file="prod.tfvars"
```

---

## OUTPUTS

```hcl
# outputs.tf
output "instance_ip" {
  description = "Public IP of EC2 instance"
  value       = aws_instance.web.public_ip
}

output "bucket_arn" {
  value     = aws_s3_bucket.my_bucket.arn
  sensitive = true          # Hides value in CLI output (NOT in state — see Gotchas)
}
```

---

## COMMON RESOURCES (AWS)

```hcl
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
  tags                 = { Name = "main-vpc" }
}

# Subnet
resource "aws_subnet" "public" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-east-1a"
}
```

---

## DATA SOURCES

```hcl
# Look up existing AMI
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]    # Canonical

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-*-22.04-amd64-server-*"]
  }
}

resource "aws_instance" "web" {
  ami = data.aws_ami.ubuntu.id
}
```

---

## EXPRESSIONS & FUNCTIONS

```hcl
# String interpolation
name = "web-${var.env}"

# Conditionals
instance_type = var.env == "prod" ? "t3.large" : "t3.micro"

# for_each — keyed instances (prefer over count for named things)
resource "aws_iam_user" "users" {
  for_each = toset(["alice", "bob", "carol"])
  name     = each.key
}

# count — numbered instances
resource "aws_instance" "web" {
  count         = 3
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
  tags          = { Name = "web-${count.index}" }
}
```

```hcl
# Common functions
length(var.list)
join(",", var.list)
split(",", "a,b,c")
toset(["a", "b", "a"])
lookup(var.map, "key", "default")
file("path/to/file")
templatefile("tmpl.tpl", { var = "val" })
cidrsubnet("10.0.0.0/16", 8, 1)      # "10.0.1.0/24"
```

---

## REMOTE STATE BACKEND (S3)

```hcl
# backend.tf
terraform {
  backend "s3" {
    bucket         = "my-tf-state-bucket"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "tf-state-lock"      # Enable state locking
    encrypt        = true
  }
}
```

---

## MODULES

```hcl
# Local module
module "vpc" {
  source     = "./modules/vpc"
  cidr_block = "10.0.0.0/16"
}

# Registry module
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"
  cidr    = "10.0.0.0/16"
}

# Reference module output
resource "aws_instance" "web" {
  subnet_id = module.vpc.public_subnet_id
}
```

---

## DEBUGGING

```bash
TF_LOG=DEBUG terraform apply          # Full debug logging
TF_LOG=INFO terraform plan            # Info-level logging
TF_LOG_PATH=tf.log terraform apply    # Write logs to file
terraform console                     # Interactive expression tester
terraform graph | dot -Tpng > graph.png   # Dependency graph (needs graphviz)
```

---

## TIPS & GOTCHAS

- **State contains secrets in plaintext** — `sensitive = true` only masks CLI output; passwords, keys, and connection strings sit readable in tfstate. Remote backend with encryption + tight bucket access is non-negotiable; never commit tfstate.
- **Never hand-edit tfstate** — use `state mv`/`state rm`/`import`. A corrupted state file means Terraform's model of reality is wrong, and `apply` acts on the model.
- **Read the plan like a contract** — the summary line (`X to add, Y to change, Z to destroy`) hides which ones; a rename or immutable-field change shows as destroy-and-recreate. `-out=tfplan` then `apply tfplan` guarantees you apply exactly what you reviewed.
- **Commit `.terraform.lock.hcl`** — it pins provider versions across machines/CI; without it, `~> 5.0` resolves differently next month and plans drift.
- **`for_each` over `count` for named resources** — count is positional: removing item 2 of 5 shifts indexes and recreates items 3-5. for_each keys by name; removals touch only the removed.
- **`-auto-approve` belongs in CI pipelines only** — interactively it deletes the one moment you'd have caught the destroy.
- **Manual console changes = drift** — Terraform will revert them on the next apply. If a change should stay, make it in code (or `import` it); infra has one source of truth or none.
- **`state rm` forgets, it doesn't delete** — the resource lives on, unmanaged and unbilled-for by nobody. Pair with documentation or an import elsewhere.

---
*Last Updated: 2026-07*
