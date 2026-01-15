# Terraform Complete Course: Basic to Advanced

A comprehensive guide to mastering Terraform for infrastructure automation, covering everything from fundamental concepts to advanced enterprise patterns. This course includes hands-on examples, best practices, and real-world interview questions.

---

## 📚 Table of Contents

### Part 1: Terraform Fundamentals
1. [Introduction to Infrastructure as Code](#1-introduction-to-infrastructure-as-code)
2. [Getting Started with Terraform](#2-getting-started-with-terraform)
3. [Terraform Configuration Language (HCL)](#3-terraform-configuration-language-hcl)
4. [Core Terraform Commands](#4-core-terraform-commands)
5. [Understanding Terraform State](#5-understanding-terraform-state)
6. [Variables and Outputs](#6-variables-and-outputs)

### Part 2: Intermediate Concepts
7. [Terraform Providers](#7-terraform-providers)
8. [Resource Dependencies](#8-resource-dependencies)
9. [Data Sources](#9-data-sources)
10. [Terraform Modules](#10-terraform-modules)
11. [Remote State Management](#11-remote-state-management)
12. [Workspaces](#12-workspaces)

### Part 3: Advanced Topics
13. [Advanced Module Patterns](#13-advanced-module-patterns)
14. [Dynamic Blocks and Expressions](#14-dynamic-blocks-and-expressions)
15. [Terraform Functions](#15-terraform-functions)
16. [Provisioners and Local-exec](#16-provisioners-and-local-exec)
17. [State Locking and Backend Configuration](#17-state-locking-and-backend-configuration)
18. [Terraform Cloud and Enterprise](#18-terraform-cloud-and-enterprise)

### Part 4: Best Practices & Production
19. [Code Organization and Structure](#19-code-organization-and-structure)
20. [Security Best Practices](#20-security-best-practices)
21. [Testing Terraform Code](#21-testing-terraform-code)
22. [CI/CD Integration](#22-cicd-integration)
23. [Disaster Recovery and Rollback](#23-disaster-recovery-and-rollback)
24. [Multi-Cloud Strategies](#24-multi-cloud-strategies)

### Part 5: Interview Preparation
25. [Common Interview Questions](#25-common-interview-questions)
26. [Scenario-Based Problems](#26-scenario-based-problems)
27. [Troubleshooting Guide](#27-troubleshooting-guide)

---

## Part 1: Terraform Fundamentals

### 1. Introduction to Infrastructure as Code

#### What is Infrastructure as Code (IaC)?
Infrastructure as Code is the practice of managing and provisioning infrastructure through machine-readable definition files rather than manual configuration or interactive tools.

**Key Benefits:**
- **Version Control**: Track infrastructure changes over time
- **Reproducibility**: Create identical environments consistently
- **Automation**: Reduce manual errors and deployment time
- **Documentation**: Code serves as living documentation
- **Collaboration**: Enable team-based infrastructure management

#### Why Terraform?
- **Provider Agnostic**: Works with AWS, Azure, GCP, and 100+ providers
- **Declarative Syntax**: Define desired state, Terraform handles implementation
- **State Management**: Tracks actual vs. desired infrastructure state
- **Plan Before Apply**: Preview changes before execution
- **Large Community**: Extensive modules and community support

---

### 2. Getting Started with Terraform

#### Installation
```bash
# macOS
brew tap hashicorp/tap
brew install hashicorp/tap/terraform

# Linux (Ubuntu/Debian)
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

# Windows (Chocolatey)
choco install terraform
```

#### Verify Installation
```bash
terraform version
# Output: Terraform v1.6.0
```

#### Your First Terraform Configuration

Create `main.tf`:
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

resource "aws_s3_bucket" "example" {
  bucket = "my-first-terraform-bucket-12345"
  
  tags = {
    Name        = "My First Bucket"
    Environment = "Dev"
  }
}
```

#### Initialize and Apply
```bash
# Initialize Terraform (downloads providers)
terraform init

# Format code
terraform fmt

# Validate configuration
terraform validate

# Preview changes
terraform plan

# Apply changes
terraform apply

# Destroy resources
terraform destroy
```

---

### 3. Terraform Configuration Language (HCL)

#### Basic Syntax Structure
```hcl
# Block type "resource_type" "local_name" {
#   argument1 = value1
#   argument2 = value2
#   
#   nested_block {
#     nested_argument = value
#   }
# }
```

#### Main Block Types

**1. Resource Blocks**
```hcl
resource "aws_instance" "web_server" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  tags = {
    Name = "WebServer"
  }
}
```

**2. Data Blocks**
```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
  
  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
  
  owners = ["099720109477"] # Canonical
}
```

**3. Variable Blocks**
```hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}
```

**4. Output Blocks**
```hcl
output "instance_ip" {
  description = "Public IP of EC2 instance"
  value       = aws_instance.web_server.public_ip
}
```

---

### 4. Core Terraform Commands

#### Essential Commands

```bash
# Initialize working directory
terraform init
  # Downloads providers
  # Initializes backend
  # Prepares modules

# Format code to HCL standards
terraform fmt
  # -recursive: Format subdirectories
  # -check: Check if formatting needed

# Validate syntax and configuration
terraform validate

# Create execution plan
terraform plan
  # -out=filename: Save plan to file
  # -var="key=value": Set variable
  # -var-file="vars.tfvars": Load variables from file

# Apply changes
terraform apply
  # -auto-approve: Skip confirmation
  # filename: Apply saved plan

# Destroy all resources
terraform destroy
  # -target=resource: Destroy specific resource
  # -auto-approve: Skip confirmation

# Show current state
terraform show

# List resources in state
terraform state list

# Show specific resource
terraform state show aws_instance.web_server

# Refresh state
terraform refresh

# Import existing resource
terraform import aws_instance.web i-1234567890abcdef0

# Output values
terraform output
  # output_name: Show specific output
```

#### Command Workflow
```bash
# Typical workflow
terraform init          # Initialize
terraform fmt           # Format code
terraform validate      # Check syntax
terraform plan          # Preview changes
terraform apply         # Apply changes
terraform output        # View outputs
```

---

### 5. Understanding Terraform State

#### What is Terraform State?
Terraform state is a mapping between your configuration files and real-world resources. It's stored in `terraform.tfstate` file.

**Key Concepts:**
- **State File**: JSON file tracking resource metadata
- **State Locking**: Prevents concurrent modifications
- **Remote State**: Shared state for team collaboration
- **State Backup**: Automatic backup created on changes

#### State File Structure
```json
{
  "version": 4,
  "terraform_version": "1.6.0",
  "serial": 1,
  "lineage": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "outputs": {},
  "resources": [
    {
      "mode": "managed",
      "type": "aws_s3_bucket",
      "name": "example",
      "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
      "instances": [
        {
          "schema_version": 0,
          "attributes": {
            "bucket": "my-terraform-bucket",
            "id": "my-terraform-bucket"
          }
        }
      ]
    }
  ]
}
```

#### State Management Commands
```bash
# List resources
terraform state list

# Show resource details
terraform state show aws_instance.web

# Move resource in state
terraform state mv aws_instance.old aws_instance.new

# Remove resource from state (doesn't delete resource)
terraform state rm aws_instance.web

# Pull remote state
terraform state pull

# Push local state to remote
terraform state push

# Replace provider in state
terraform state replace-provider hashicorp/aws registry.terraform.io/hashicorp/aws
```

#### State Best Practices
1. **Never edit state files manually**
2. **Always use remote state for teams**
3. **Enable state locking**
4. **Use state encryption**
5. **Backup state files regularly**
6. **Use workspaces for environments**

---

### 6. Variables and Outputs

#### Input Variables

**Declaration (variables.tf)**
```hcl
variable "region" {
  description = "AWS region for resources"
  type        = string
  default     = "us-east-1"
}

variable "instance_count" {
  description = "Number of instances to create"
  type        = number
  default     = 2
  
  validation {
    condition     = var.instance_count > 0 && var.instance_count < 10
    error_message = "Instance count must be between 1 and 9."
  }
}

variable "tags" {
  description = "Tags to apply to resources"
  type        = map(string)
  default     = {
    Environment = "Dev"
    Project     = "Demo"
  }
}

variable "availability_zones" {
  description = "List of AZs"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b"]
}

variable "enable_monitoring" {
  description = "Enable detailed monitoring"
  type        = bool
  default     = false
}
```

**Variable Types**
- `string`: Text values
- `number`: Numeric values
- `bool`: true/false
- `list(type)`: Ordered collection
- `map(type)`: Key-value pairs
- `set(type)`: Unordered unique values
- `object({...})`: Complex structure
- `tuple([...])`: Fixed-length collection

**Using Variables**
```hcl
resource "aws_instance" "web" {
  ami               = data.aws_ami.ubuntu.id
  instance_type     = var.instance_type
  count             = var.instance_count
  availability_zone = var.availability_zones[0]
  monitoring        = var.enable_monitoring
  
  tags = merge(
    var.tags,
    {
      Name = "web-server-${count.index + 1}"
    }
  )
}
```

**Setting Variables**

1. **Command Line**
```bash
terraform apply -var="region=us-west-2" -var="instance_count=3"
```

2. **Variable Files (terraform.tfvars)**
```hcl
region         = "us-west-2"
instance_count = 3
tags = {
  Environment = "Production"
  Team        = "DevOps"
}
```

3. **Environment Variables**
```bash
export TF_VAR_region="us-west-2"
export TF_VAR_instance_count=3
```

4. **Auto-loaded Files**
- `terraform.tfvars`
- `terraform.tfvars.json`
- `*.auto.tfvars`
- `*.auto.tfvars.json`

#### Output Values

**Declaration (outputs.tf)**
```hcl
output "instance_ids" {
  description = "IDs of EC2 instances"
  value       = aws_instance.web[*].id
}

output "instance_public_ips" {
  description = "Public IPs of instances"
  value       = aws_instance.web[*].public_ip
}

output "load_balancer_dns" {
  description = "DNS name of load balancer"
  value       = aws_lb.main.dns_name
}

output "sensitive_password" {
  description = "Database password"
  value       = random_password.db.result
  sensitive   = true
}
```

**Viewing Outputs**
```bash
# View all outputs
terraform output

# View specific output
terraform output instance_ids

# JSON format
terraform output -json

# Use in scripts
INSTANCE_IP=$(terraform output -raw instance_public_ips)
```

---

## Part 2: Intermediate Concepts

### 7. Terraform Providers

#### What are Providers?
Providers are plugins that interact with cloud providers, SaaS services, and other APIs.

**Popular Providers:**
- AWS (Amazon Web Services)
- Azure (Microsoft Azure)
- GCP (Google Cloud Platform)
- Kubernetes
- Docker
- GitHub
- Datadog
- And 3000+ more!

#### Provider Configuration

**Basic Provider Setup**
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region  = var.region
  profile = "default"
}
```

**Multiple Provider Instances**
```hcl
# Primary region provider
provider "aws" {
  alias  = "us_east"
  region = "us-east-1"
}

# Secondary region provider
provider "aws" {
  alias  = "us_west"
  region = "us-west-2"
}

# Use specific provider
resource "aws_s3_bucket" "east_bucket" {
  provider = aws.us_east
  bucket   = "my-east-bucket"
}

resource "aws_s3_bucket" "west_bucket" {
  provider = aws.us_west
  bucket   = "my-west-bucket"
}
```

**Multi-Cloud Example**
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

provider "azurerm" {
  features {}
}

provider "google" {
  project = "my-gcp-project"
  region  = "us-central1"
}
```

---

### 8. Resource Dependencies

#### Implicit Dependencies
Terraform automatically determines dependencies based on resource references.

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id  # Implicit dependency
  cidr_block = "10.0.1.0/24"
}

resource "aws_instance" "web" {
  ami           = "ami-12345"
  subnet_id     = aws_subnet.public.id  # Implicit dependency
  instance_type = "t2.micro"
}
```

**Dependency Graph:**
```
aws_vpc.main
  ↓
aws_subnet.public
  ↓
aws_instance.web
```

#### Explicit Dependencies
Use `depends_on` when dependencies aren't automatically inferred.

```hcl
resource "aws_iam_role" "example" {
  name = "example-role"
  
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "ec2.amazonaws.com"
      }
    }]
  })
}

resource "aws_iam_role_policy" "example" {
  role = aws_iam_role.example.name
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action   = ["s3:*"]
      Effect   = "Allow"
      Resource = "*"
    }]
  })
}

resource "aws_instance" "web" {
  ami           = "ami-12345"
  instance_type = "t2.micro"
  
  # Explicit dependency - wait for IAM policies to propagate
  depends_on = [
    aws_iam_role_policy.example
  ]
}
```

#### Visualizing Dependencies
```bash
# Generate dependency graph
terraform graph | dot -Tpng > graph.png

# Or use online viewer
terraform graph | pbcopy  # macOS
# Paste into: http://www.webgraphviz.com/
```

---

### 9. Data Sources

Data sources allow Terraform to fetch information from external sources.

#### Common AWS Data Sources

**1. AMI Lookup**
```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]  # Canonical
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
  
  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
}
```

**2. VPC Lookup**
```hcl
data "aws_vpc" "default" {
  default = true
}

data "aws_subnets" "default" {
  filter {
    name   = "vpc-id"
    values = [data.aws_vpc.default.id]
  }
}

output "vpc_cidr" {
  value = data.aws_vpc.default.cidr_block
}
```

**3. Availability Zones**
```hcl
data "aws_availability_zones" "available" {
  state = "available"
}

resource "aws_subnet" "example" {
  count             = 3
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(aws_vpc.main.cidr_block, 8, count.index)
  availability_zone = data.aws_availability_zones.available.names[count.index]
}
```

**4. Current AWS Account/Region**
```hcl
data "aws_caller_identity" "current" {}

data "aws_region" "current" {}

output "account_id" {
  value = data.aws_caller_identity.current.account_id
}

output "current_region" {
  value = data.aws_region.current.name
}
```

---

### 10. Terraform Modules

Modules are containers for multiple resources that are used together.

#### Creating a Module

**Module Structure:**
```
modules/
└── vpc/
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    └── README.md
```

**modules/vpc/main.tf:**
```hcl
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = merge(
    var.tags,
    {
      Name = "${var.project_name}-vpc"
    }
  )
}

resource "aws_subnet" "public" {
  count                   = length(var.public_subnet_cidrs)
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnet_cidrs[count.index]
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = true
  
  tags = merge(
    var.tags,
    {
      Name = "${var.project_name}-public-${count.index + 1}"
      Type = "Public"
    }
  )
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = merge(
    var.tags,
    {
      Name = "${var.project_name}-igw"
    }
  )
}
```

**modules/vpc/variables.tf:**
```hcl
variable "project_name" {
  description = "Project name for resource naming"
  type        = string
}

variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "public_subnet_cidrs" {
  description = "CIDR blocks for public subnets"
  type        = list(string)
}

variable "availability_zones" {
  description = "Availability zones for subnets"
  type        = list(string)
}

variable "tags" {
  description = "Tags to apply to all resources"
  type        = map(string)
  default     = {}
}
```

**modules/vpc/outputs.tf:**
```hcl
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}

output "vpc_cidr" {
  description = "CIDR block of the VPC"
  value       = aws_vpc.main.cidr_block
}

output "public_subnet_ids" {
  description = "IDs of public subnets"
  value       = aws_subnet.public[*].id
}

output "internet_gateway_id" {
  description = "ID of the Internet Gateway"
  value       = aws_internet_gateway.main.id
}
```

#### Using Modules

**main.tf:**
```hcl
module "vpc" {
  source = "./modules/vpc"
  
  project_name         = "my-app"
  vpc_cidr             = "10.0.0.0/16"
  public_subnet_cidrs  = ["10.0.1.0/24", "10.0.2.0/24"]
  availability_zones   = ["us-east-1a", "us-east-1b"]
  
  tags = {
    Environment = "Production"
    ManagedBy   = "Terraform"
  }
}

# Reference module outputs
resource "aws_instance" "web" {
  ami           = "ami-12345"
  instance_type = "t2.micro"
  subnet_id     = module.vpc.public_subnet_ids[0]
}

output "vpc_id" {
  value = module.vpc.vpc_id
}
```

#### Module Sources

**1. Local Path**
```hcl
module "vpc" {
  source = "./modules/vpc"
}
```

**2. Terraform Registry**
```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"
  
  name = "my-vpc"
  cidr = "10.0.0.0/16"
}
```

**3. GitHub**
```hcl
module "vpc" {
  source = "github.com/username/terraform-aws-vpc"
}

# Specific branch/tag
module "vpc" {
  source = "github.com/username/terraform-aws-vpc?ref=v1.0.0"
}
```

**4. Git**
```hcl
module "vpc" {
  source = "git::https://gitlab.com/username/terraform-aws-vpc.git"
}
```

---

### 11. Remote State Management

#### Why Remote State?
- **Collaboration**: Multiple team members can access shared state
- **State Locking**: Prevent concurrent modifications
- **Security**: Store state securely with encryption
- **Disaster Recovery**: Backup and version state

#### S3 Backend Configuration

**backend.tf:**
```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"
    
    # Optional: Use IAM role
    role_arn = "arn:aws:iam::123456789012:role/TerraformStateRole"
  }
}
```

**Create S3 Bucket and DynamoDB Table:**
```hcl
# Run this first to create backend resources
resource "aws_s3_bucket" "terraform_state" {
  bucket = "my-terraform-state-bucket"
  
  lifecycle {
    prevent_destroy = true
  }
}

resource "aws_s3_bucket_versioning" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-state-lock"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"
  
  attribute {
    name = "LockID"
    type = "S"
  }
}
```

#### Terraform Cloud Backend

```hcl
terraform {
  cloud {
    organization = "my-org"
    
    workspaces {
      name = "my-app-prod"
    }
  }
}
```

#### State Migration

```bash
# 1. Add backend configuration
# 2. Initialize with migration
terraform init -migrate-state

# 3. Verify
terraform state list
```

---

### 12. Workspaces

Workspaces allow managing multiple environments with the same configuration.

#### Workspace Commands

```bash
# List workspaces
terraform workspace list

# Create new workspace
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod

# Switch workspace
terraform workspace select dev

# Show current workspace
terraform workspace show

# Delete workspace
terraform workspace delete dev
```

#### Using Workspaces in Configuration

```hcl
locals {
  environment = terraform.workspace
  
  instance_counts = {
    dev     = 1
    staging = 2
    prod    = 5
  }
  
  instance_types = {
    dev     = "t2.micro"
    staging = "t2.small"
    prod    = "t2.large"
  }
}

resource "aws_instance" "app" {
  count         = local.instance_counts[local.environment]
  ami           = data.aws_ami.ubuntu.id
  instance_type = local.instance_types[local.environment]
  
  tags = {
    Name        = "${local.environment}-app-${count.index + 1}"
    Environment = local.environment
  }
}
```

---

## Part 3: Advanced Topics

### 13. Advanced Module Patterns

#### Conditional Resources

```hcl
variable "create_database" {
  type    = bool
  default = false
}

resource "aws_db_instance" "main" {
  count = var.create_database ? 1 : 0
  
  identifier     = "my-database"
  engine         = "mysql"
  instance_class = "db.t3.micro"
}
```

#### For_each with Modules

```hcl
variable "applications" {
  type = map(object({
    instance_type = string
    desired_count = number
  }))
  
  default = {
    web = {
      instance_type = "t2.micro"
      desired_count = 2
    }
    api = {
      instance_type = "t2.small"
      desired_count = 3
    }
  }
}

module "application" {
  source   = "./modules/application"
  for_each = var.applications
  
  name          = each.key
  instance_type = each.value.instance_type
  desired_count = each.value.desired_count
}
```

---

### 14. Dynamic Blocks and Expressions

#### Dynamic Blocks

```hcl
variable "ingress_rules" {
  type = list(object({
    from_port   = number
    to_port     = number
    protocol    = string
    cidr_blocks = list(string)
  }))
  
  default = [
    {
      from_port   = 80
      to_port     = 80
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    },
    {
      from_port   = 443
      to_port     = 443
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  ]
}

resource "aws_security_group" "web" {
  name   = "web-sg"
  vpc_id = aws_vpc.main.id
  
  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

#### Conditional Expressions

```hcl
# Ternary operator
instance_type = var.environment == "prod" ? "t2.large" : "t2.micro"

# Complex conditions
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = (
    var.environment == "prod" ? "t2.large" :
    var.environment == "staging" ? "t2.medium" :
    "t2.micro"
  )
  
  monitoring = var.environment == "prod" ? true : false
}
```

---

### 15. Terraform Functions

#### String Functions

```hcl
# upper/lower
output "uppercase" {
  value = upper("hello")  # "HELLO"
}

# format
output "formatted" {
  value = format("Hello, %s!", "World")  # "Hello, World!"
}

# join/split
output "joined" {
  value = join(",", ["a", "b", "c"])  # "a,b,c"
}

# replace
output "replaced" {
  value = replace("hello world", "world", "terraform")  # "hello terraform"
}
```

#### Collection Functions

```hcl
# length
output "list_length" {
  value = length(["a", "b", "c"])  # 3
}

# concat
output "concatenated" {
  value = concat(["a", "b"], ["c", "d"])  # ["a", "b", "c", "d"]
}

# merge
output "merged" {
  value = merge(
    {a = 1, b = 2},
    {c = 3, d = 4}
  )  # {a = 1, b = 2, c = 3, d = 4}
}

# flatten
output "flattened" {
  value = flatten([["a", "b"], ["c", "d"]])  # ["a", "b", "c", "d"]
}
```

#### IP Network Functions

```hcl
# cidrsubnet
output "subnets" {
  value = [
    cidrsubnet("10.0.0.0/16", 8, 0),  # "10.0.0.0/24"
    cidrsubnet("10.0.0.0/16", 8, 1),  # "10.0.1.0/24"
    cidrsubnet("10.0.0.0/16", 8, 2),  # "10.0.2.0/24"
  ]
}

# cidrhost
output "host_ip" {
  value = cidrhost("10.0.0.0/24", 5)  # "10.0.0.5"
}
```

---

### 16. Provisioners and Local-exec

**Note**: Provisioners are a last resort. Use native cloud init when possible.

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
  
  # File provisioner
  provisioner "file" {
    source      = "script.sh"
    destination = "/tmp/script.sh"
    
    connection {
      type        = "ssh"
      user        = "ubuntu"
      private_key = file("~/.ssh/id_rsa")
      host        = self.public_ip
    }
  }
  
  # Remote-exec provisioner
  provisioner "remote-exec" {
    inline = [
      "chmod +x /tmp/script.sh",
      "/tmp/script.sh"
    ]
    
    connection {
      type        = "ssh"
      user        = "ubuntu"
      private_key = file("~/.ssh/id_rsa")
      host        = self.public_ip
    }
  }
  
  # Local-exec provisioner
  provisioner "local-exec" {
    command = "echo ${self.private_ip} >> private_ips.txt"
  }
  
  # Null resource with local-exec
  provisioner "local-exec" {
    when    = destroy
    command = "echo 'Instance ${self.id} is being destroyed'"
  }
}
```

---

## Part 4: Best Practices & Production

### 19. Code Organization and Structure

#### Recommended Project Structure

```
terraform-project/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   ├── terraform.tfvars
│   │   └── backend.tf
│   ├── staging/
│   └── prod/
├── modules/
│   ├── vpc/
│   ├── ec2/
│   └── rds/
├── global/
│   ├── iam/
│   └── s3/
├── .gitignore
└── README.md
```

#### File Naming Conventions

- `main.tf` - Primary resources
- `variables.tf` - Input variable declarations
- `outputs.tf` - Output value declarations
- `providers.tf` - Provider configurations
- `backend.tf` - Backend configuration
- `locals.tf` - Local value declarations
- `data.tf` - Data source declarations
- `versions.tf` - Terraform and provider version constraints

---

### 20. Security Best Practices

#### 1. Never Commit Secrets

**.gitignore:**
```
# Terraform files
*.tfstate
*.tfstate.backup
.terraform/
.terraform.lock.hcl
crash.log
override.tf
override.tf.json

# Sensitive files
*.tfvars
*.auto.tfvars
**/.terraform/*
```

#### 2. Use AWS Secrets Manager/Parameter Store

```hcl
data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = "prod/db/password"
}

resource "aws_db_instance" "main" {
  identifier = "production-db"
  engine     = "mysql"
  password   = data.aws_secretsmanager_secret_version.db_password.secret_string
}
```

---

## Part 5: Interview Preparation

### 25. Common Interview Questions

#### Basic Level Questions

**Q1: What is Terraform and why use it?**

**Answer:** Terraform is an open-source Infrastructure as Code (IaC) tool by HashiCorp that allows you to define and provision infrastructure using declarative configuration files. Key benefits include:
- Version control for infrastructure
- Provider-agnostic (AWS, Azure, GCP, etc.)
- Declarative syntax - define desired state
- Plan before apply - preview changes
- State management for tracking resources
- Reusable modules for standardization

---

**Q2: Explain the Terraform workflow.**

**Answer:** The standard Terraform workflow consists of:

1. **Write**: Author infrastructure as code in `.tf` files
2. **Init**: `terraform init` - Initialize working directory, download providers
3. **Plan**: `terraform plan` - Preview changes before applying
4. **Apply**: `terraform apply` - Provision infrastructure
5. **Destroy**: `terraform destroy` - Remove infrastructure when needed

Additional commands:
- `terraform fmt` - Format code
- `terraform validate` - Validate syntax
- `terraform show` - Inspect current state

---

**Q3: What is Terraform state and why is it important?**

**Answer:** Terraform state is a JSON file (`terraform.tfstate`) that:
- Maps configuration to real-world resources
- Tracks resource metadata and dependencies
- Stores resource attributes for reference
- Enables performance optimization by caching
- Determines what changes need to be applied

**Why it's critical:**
- Without state, Terraform can't track existing resources
- State enables team collaboration
- Contains sensitive data - must be secured
- State locking prevents concurrent modifications

---

**Q4: Difference between `count` and `for_each`?**

**Answer:**

**Count:**
```hcl
resource "aws_instance" "server" {
  count         = 3
  ami           = "ami-12345"
  instance_type = "t2.micro"
  
  tags = {
    Name = "server-${count.index}"
  }
}
# Creates: aws_instance.server[0], server[1], server[2]
```

**For_each:**
```hcl
resource "aws_instance" "server" {
  for_each = toset(["web", "api", "db"])
  
  ami           = "ami-12345"
  instance_type = "t2.micro"
  
  tags = {
    Name = each.key
  }
}
# Creates: aws_instance.server["web"], server["api"], server["db"]
```

**Key differences:**
- `count` uses numeric indices; `for_each` uses keys
- Removing middle element with `count` causes recreation; `for_each` doesn't
- `for_each` is better for managing distinct resources

---

**Q5: What are Terraform modules and why use them?**

**Answer:** Modules are containers for multiple resources used together. They enable:

**Benefits:**
- **Reusability**: Write once, use multiple times
- **Organization**: Logical grouping of resources
- **Encapsulation**: Hide complexity
- **Consistency**: Enforce standards across teams
- **Versioning**: Track module changes

**Example:**
```hcl
module "vpc" {
  source = "./modules/vpc"
  
  vpc_cidr           = "10.0.0.0/16"
  availability_zones = ["us-east-1a", "us-east-1b"]
}

output "vpc_id" {
  value = module.vpc.vpc_id
}
```

---

#### Intermediate Level Questions

**Q6: Explain remote state and its benefits.**

**Answer:** Remote state stores the state file in a shared location instead of locally.

**Configuration:**
```hcl
terraform {
  backend "s3" {
    bucket         = "my-tf-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```

**Benefits:**
- **Collaboration**: Team members share same state
- **Security**: Centralized access control
- **Locking**: Prevents concurrent modifications
- **Backup**: Automatic versioning
- **Reliability**: No local state corruption

---

**Q7: How do you handle sensitive data in Terraform?**

**Answer:**

1. **Mark outputs as sensitive:**
```hcl
output "db_password" {
  value     = random_password.db.result
  sensitive = true
}
```

2. **Use external secret management:**
```hcl
data "aws_secretsmanager_secret_version" "api_key" {
  secret_id = "prod/api/key"
}
```

3. **Environment variables:**
```bash
export TF_VAR_db_password="secretpass"
```

4. **Never commit:**
- Add `*.tfvars` to `.gitignore`
- Use `.tfvars.example` as template
- Store secrets in AWS Secrets Manager/Vault

---

**Q8: What is the difference between implicit and explicit dependencies?**

**Answer:**

**Implicit Dependencies** - Automatically detected:
```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id  # Implicit dependency
  cidr_block = "10.0.1.0/24"
}
```

**Explicit Dependencies** - Manually specified:
```hcl
resource "aws_instance" "web" {
  ami           = "ami-12345"
  instance_type = "t2.micro"
  
  depends_on = [
    aws_iam_role_policy.example
  ]
}
```

**When to use explicit:**
- Hidden dependencies (API propagation delays)
- Ordering requirements not reflected in attributes
- Cross-module dependencies

---

**Q9: Explain workspaces and when to use them.**

**Answer:** Workspaces allow managing multiple instances of the same configuration.

```bash
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod
terraform workspace select dev
```

**Usage:**
```hcl
locals {
  env = terraform.workspace
  
  instance_count = {
    dev     = 1
    staging = 2
    prod    = 5
  }
}

resource "aws_instance" "app" {
  count         = local.instance_count[local.env]
  instance_type = local.env == "prod" ? "t2.large" : "t2.micro"
}
```

**When to use:**
- ✅ Same infrastructure, different sizes
- ✅ Temporary test environments
- ❌ Don't use for completely different configurations
- ❌ Different AWS accounts - use separate state files

---

**Q10: How do you upgrade Terraform providers safely?**

**Answer:**

1. **Check current versions:**
```bash
terraform version
terraform providers
```

2. **Review provider changelog:**
```bash
# Check for breaking changes
```

3. **Update version constraints:**
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"  # Update this
    }
  }
}
```

4. **Initialize and upgrade:**
```bash
terraform init -upgrade
```

5. **Test in non-prod first:**
```bash
terraform plan
terraform apply
```

6. **Use version pinning for production:**
```hcl
version = "5.25.0"  # Exact version
```

---

#### Advanced Level Questions

**Q11: Explain the Terraform import process and its limitations.**

**Answer:**

Terraform import brings existing infrastructure under Terraform management.

**Process:**

1. **Write resource configuration:**
```hcl
resource "aws_instance" "imported" {
  # Configuration must match existing resource
  ami           = "ami-12345"
  instance_type = "t2.micro"
}
```

2. **Import the resource:**
```bash
terraform import aws_instance.imported i-1234567890abcdef0
```

3. **Verify state:**
```bash
terraform state show aws_instance.imported
terraform plan  # Should show no changes
```

**Limitations:**
- Imports one resource at a time (no bulk import)
- Doesn't generate configuration - you must write it
- Resource must already exist
- Can't import into modules directly in older versions
- Some resources don't support import

**Tools to help:**
- `terraformer` - Bulk import tool
- `terracognita` - AWS infrastructure importer
- `terraform-import-github-organization` - GitHub importer

---

**Q12: How do you handle Terraform state file corruption?**

**Answer:**

**Prevention:**
1. Use remote state with versioning
2. Enable state locking
3. Never edit state manually
4. Use `terraform state` commands

**Recovery:**

1. **Use backup:**
```bash
cp terraform.tfstate.backup terraform.tfstate
```

2. **Pull from remote (if using remote state):**
```bash
terraform state pull > terraform.tfstate
```

3. **S3 versioning:**
```bash
aws s3api list-object-versions \
  --bucket my-tf-state \
  --prefix terraform.tfstate

aws s3api get-object \
  --bucket my-tf-state \
  --key terraform.tfstate \
  --version-id <version-id> \
  terraform.tfstate.recovered
```

4. **Rebuild state (last resort):**
```bash
# Remove corrupted state
rm terraform.tfstate

# Import each resource
terraform import aws_instance.web i-1234567890abcdef0
```

---

**Q13: Explain Terraform's graph and how it determines execution order.**

**Answer:**

Terraform builds a dependency graph to determine resource creation order.

**View the graph:**
```bash
terraform graph | dot -Tpng > graph.png
```

**How it works:**

1. **Parse configuration** - Read all `.tf` files
2. **Build graph** - Create nodes for each resource
3. **Determine dependencies** - Connect nodes based on references
4. **Topological sort** - Order execution
5. **Parallel execution** - Independent resources run concurrently

**Example:**
```hcl
resource "aws_vpc" "main" {}              # Node 1
resource "aws_subnet" "public" {           # Node 2 - depends on Node 1
  vpc_id = aws_vpc.main.id
}
resource "aws_instance" "web" {            # Node 3 - depends on Node 2
  subnet_id = aws_subnet.public.id
}
resource "aws_s3_bucket" "logs" {}        # Node 4 - independent, runs parallel
```

**Execution:**
- VPC and S3 bucket create in parallel
- Subnet waits for VPC
- Instance waits for Subnet

---

**Q14: How do you manage secrets rotation with Terraform?**

**Answer:**

**Strategy 1: Use random_password with keepers**
```hcl
resource "random_password" "db" {
  length  = 16
  special = true
  
  keepers = {
    rotation_time = "2024-01-01"  # Change this to rotate
  }
}

resource "aws_secretsmanager_secret_version" "db_password" {
  secret_id     = aws_secretsmanager_secret.db.id
  secret_string = random_password.db.result
}
```

**Strategy 2: External rotation with lifecycle ignore:**
```hcl
resource "aws_db_instance" "main" {
  password = data.aws_secretsmanager_secret_version.db.secret_string
  
  lifecycle {
    ignore_changes = [password]
  }
}
```

---

**Q15: Describe a blue-green deployment strategy with Terraform.**

**Answer:**

```hcl
variable "active_environment" {
  type    = string
  default = "blue"
}

resource "aws_instance" "blue" {
  count         = var.active_environment == "blue" ? 3 : 0
  ami           = var.blue_ami
  instance_type = "t2.micro"
  
  tags = {
    Environment = "blue"
  }
}

resource "aws_instance" "green" {
  count         = var.active_environment == "green" ? 3 : 0
  ami           = var.green_ami
  instance_type = "t2.micro"
  
  tags = {
    Environment = "green"
  }
}

resource "aws_lb_target_group_attachment" "active" {
  target_group_arn = aws_lb_target_group.main.arn
  target_id        = (
    var.active_environment == "blue" 
    ? aws_instance.blue[count.index].id 
    : aws_instance.green[count.index].id
  )
  count = 3
}
```

**Deployment Steps:**
1. Deploy green environment: `active_environment = "green"`
2. Test green environment
3. Switch traffic to green
4. Destroy blue: `terraform destroy -target=aws_instance.blue`

---

### 26. Scenario-Based Problems

#### Scenario 1: State Lock Issue

**Problem:** Team member's laptop crashed during `terraform apply`. State is now locked.

**Error:**
```
Error: Error acquiring the state lock
Lock ID: a1b2c3d4-1234-5678-abcd-ef1234567890
```

**Solution:**
```bash
# 1. Verify no one is actually running Terraform
# 2. Force unlock (use with caution)
terraform force-unlock a1b2c3d4-1234-5678-abcd-ef1234567890

# 3. Verify state integrity
terraform state list

# 4. If using DynamoDB, manually check/delete lock
aws dynamodb get-item \
  --table-name terraform-locks \
  --key '{"LockID": {"S": "my-state-bucket/terraform.tfstate"}}'
```

---

#### Scenario 2: Drift Detection

**Problem:** Someone manually modified infrastructure in AWS console.

**Solution:**
```bash
# 1. Detect drift
terraform plan
# Shows resources that changed outside Terraform

# 2. Options:
# A) Revert manual changes
terraform apply

# B) Import manual changes into config
# Update .tf files to match manual changes
terraform plan  # Should show no changes

# C) Refresh state to accept drift
terraform apply -refresh-only
```

---

#### Scenario 3: Migrating State

**Problem:** Need to migrate from local state to S3.

**Solution:**
```hcl
# 1. Create backend.tf
terraform {
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "prod/terraform.tfstate"
    region = "us-east-1"
    encrypt = true
    dynamodb_table = "terraform-locks"
  }
}
```

```bash
# 2. Initialize with migration
terraform init -migrate-state

# 3. Verify
aws s3 ls s3://my-terraform-state/prod/
terraform state list
```

---

#### Scenario 4: Circular Dependency

**Problem:**
```
Error: Cycle: aws_security_group.web, aws_security_group.db
```

**Cause:**
```hcl
resource "aws_security_group" "web" {
  ingress {
    security_groups = [aws_security_group.db.id]
  }
}

resource "aws_security_group" "db" {
  ingress {
    security_groups = [aws_security_group.web.id]
  }
}
```

**Solution:**
```hcl
# Use separate rules
resource "aws_security_group" "web" {
  name = "web-sg"
}

resource "aws_security_group" "db" {
  name = "db-sg"
}

resource "aws_security_group_rule" "web_to_db" {
  type                     = "ingress"
  security_group_id        = aws_security_group.db.id
  source_security_group_id = aws_security_group.web.id
  from_port                = 3306
  to_port                  = 3306
  protocol                 = "tcp"
}

resource "aws_security_group_rule" "db_to_web" {
  type                     = "ingress"
  security_group_id        = aws_security_group.web.id
  source_security_group_id = aws_security_group.db.id
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
}
```

---

### 27. Troubleshooting Guide

#### Common Errors and Solutions

**Error 1: Provider Configuration**
```
Error: Unsupported provider version
```

**Solution:**
```bash
terraform init -upgrade
rm .terraform.lock.hcl
terraform init
```

---

**Error 2: Resource Already Exists**
```
Error: resource already exists
```

**Solution:**
```bash
# Import existing resource
terraform import aws_instance.web i-1234567890abcdef0
```

---

**Error 3: State File Locked**
```
Error: Error acquiring the state lock
```

**Solution:**
```bash
terraform force-unlock <lock-id>
```

---

**Error 4: Invalid count argument**
```
Error: The "count" value depends on resource attributes
```

**Solution:** Use `-target` for two-step apply or refactor with `for_each`

---

**Error 5: Provider Plugin Crash**
```
Error: Plugin did not respond
```

**Solution:**
```bash
rm -rf .terraform/
terraform init
```

---

## Additional Resources

### Official Documentation
- [Terraform Documentation](https://www.terraform.io/docs)
- [Terraform Registry](https://registry.terraform.io/)
- [HashiCorp Learn](https://learn.hashicorp.com/terraform)

### Best Practice Guides
- [Google Cloud Terraform Best Practices](https://cloud.google.com/docs/terraform/best-practices)
- [AWS Well-Architected for Terraform](https://docs.aws.amazon.com/)
- [Terraform Style Guide](https://www.terraform.io/docs/language/syntax/style.html)

### Community
- [Terraform GitHub](https://github.com/hashicorp/terraform)
- [Terraform Community Forum](https://discuss.hashicorp.com/c/terraform-core)
- [r/Terraform](https://reddit.com/r/Terraform)

### Tools
- **tflint** - Linter for Terraform
- **terraform-docs** - Generate documentation
- **checkov** - Security scanner
- **terraform-graph** - Visualization tool
- **terraformer** - Import existing infrastructure
- **terragrunt** - Terraform wrapper for DRY configurations

---

## Hands-On Projects

### Project 1: Multi-Tier Web Application
Build a complete web application infrastructure:
- VPC with public/private subnets
- Auto Scaling Group with ALB
- RDS MySQL database
- S3 bucket for static assets
- CloudFront distribution
- Route53 DNS

### Project 2: Kubernetes Cluster
Provision EKS cluster with:
- Worker nodes in multiple AZs
- IAM roles and policies
- VPC and networking
- Helm provider setup

### Project 3: CI/CD Pipeline
Create infrastructure for:
- CodePipeline
- CodeBuild
- CodeDeploy
- ECR for Docker images
- ECS Fargate cluster

### Project 4: Multi-Cloud Setup
Deploy application across:
- AWS (primary)
- Azure (disaster recovery)
- GCP (analytics)
- Shared monitoring with Datadog

---

## Interview Tips

### Technical Preparation
1. **Practice writing Terraform code** - Don't just read
2. **Build a portfolio project** - Show it during interviews
3. **Understand state management deeply** - Most common topic
4. **Know your provider** - AWS/Azure/GCP specifics
5. **Study modules** - Creation and best practices

### Common Follow-up Questions
- "Walk me through a Terraform project you've built"
- "How do you handle multiple environments?"
- "Describe your Terraform CI/CD pipeline"
- "How do you manage Terraform in a team?"
- "What's your strategy for testing Terraform code?"

### Red Flags to Avoid
- Never editing state files manually
- Not using version control
- Storing secrets in code
- No state locking for teams
- Not using modules for reusability

---

## Certification Path

### HashiCorp Certified: Terraform Associate

**Exam Topics:**
1. Understand Infrastructure as Code (IaC) concepts
2. Understand Terraform's purpose
3. Understand Terraform basics
4. Use the Terraform CLI
5. Interact with Terraform modules
6. Navigate Terraform workflow
7. Implement and maintain state
8. Read, generate, and modify configuration
9. Understand Terraform Cloud capabilities

**Preparation:**
- Complete HashiCorp Learn tutorials
- Practice with real projects
- Review exam objectives
- Take practice tests
- Hands-on experience (3-6 months recommended)

**Exam Details:**
- 57 questions
- 60 minutes
- Multiple choice and multiple select
- $70.50 USD
- Valid for 2 years

---

## Contributing

Feel free to contribute to this course by:
- Opening issues for corrections
- Submitting pull requests for improvements
- Sharing additional interview questions
- Adding more practical examples

---

## License

MIT License - Free to use for learning purposes

---

## Contact

For questions or feedback:
- GitHub: [@thilakakula13](https://github.com/thilakakula13)
- LinkedIn: [Thilak Akula](https://www.linkedin.com/in/thilak-akula)

---

**⭐ If this course helped you, please star this repository!**

Happy Learning! 🚀
