# Terraform Cheat Sheet

## Core Commands

### Initialization and Planning
```bash
# Initialize Terraform (download providers, modules)
terraform init

# Initialize with specific backend config
terraform init -backend-config="bucket=my-tf-state"

# Upgrade providers
terraform init -upgrade

# Validate configuration syntax
terraform validate

# Format configuration files
terraform fmt

# Recursively format all files
terraform fmt -recursive

# Create execution plan
terraform plan

# Create plan with specific var file
terraform plan -var-file="prod.tfvars"

# Save plan to file
terraform plan -out=tfplan

# Show saved plan
terraform show tfplan
```

### Apply and Destroy
```bash
# Apply changes
terraform apply

# Apply specific plan
terraform apply tfplan

# Apply with auto-approve (skip confirmation)
terraform apply -auto-approve

# Apply with specific variables
terraform apply -var="instance_type=t3.micro"

# Destroy infrastructure
terraform destroy

# Destroy with auto-approve
terraform destroy -auto-approve

# Destroy specific resource
terraform destroy -target=aws_instance.example
```

### State Management
```bash
# Show current state
terraform show

# List resources in state
terraform state list

# Show specific resource
terraform state show aws_instance.example

# Remove resource from state
terraform state rm aws_instance.example

# Import existing resource
terraform import aws_instance.example i-1234567890abcdef0

# Move resource in state
terraform state mv aws_instance.old aws_instance.new

# Refresh state
terraform refresh

# Pull remote state
terraform state pull

# Push local state to remote
terraform state push
```

### Workspace Management
```bash
# List workspaces
terraform workspace list

# Create new workspace
terraform workspace new dev

# Select workspace
terraform workspace select prod

# Show current workspace
terraform workspace show

# Delete workspace
terraform workspace delete dev
```

## Configuration Syntax

### Provider Configuration
```hcl
# AWS Provider
provider "aws" {
  region  = "us-west-2"
  profile = "default"
  
  default_tags {
    tags = {
      Environment = "production"
      Owner       = "devops-team"
    }
  }
}

# Multiple provider instances
provider "aws" {
  alias  = "east"
  region = "us-east-1"
}

# Required providers block
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "prod/terraform.tfstate"
    region = "us-west-2"
  }
}
```

### Variables
```hcl
# Variable definitions
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}

variable "availability_zones" {
  description = "List of availability zones"
  type        = list(string)
  default     = ["us-west-2a", "us-west-2b"]
}

variable "tags" {
  description = "Resource tags"
  type        = map(string)
  default     = {}
}

variable "instance_count" {
  description = "Number of instances"
  type        = number
  default     = 1
  
  validation {
    condition     = var.instance_count > 0 && var.instance_count <= 10
    error_message = "Instance count must be between 1 and 10."
  }
}

# Complex variable types
variable "database_config" {
  description = "Database configuration"
  type = object({
    engine         = string
    engine_version = string
    instance_class = string
    allocated_storage = number
  })
  
  default = {
    engine         = "mysql"
    engine_version = "8.0"
    instance_class = "db.t3.micro"
    allocated_storage = 20
  }
}
```

### Outputs
```hcl
# Basic outputs
output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.example.id
}

output "instance_public_ip" {
  description = "Public IP of the instance"
  value       = aws_instance.example.public_ip
  sensitive   = false
}

# Complex outputs
output "database_endpoint" {
  description = "RDS instance endpoint"
  value       = aws_db_instance.database.endpoint
  sensitive   = true
}

output "vpc_info" {
  description = "VPC information"
  value = {
    vpc_id     = aws_vpc.main.id
    cidr_block = aws_vpc.main.cidr_block
    subnets    = aws_subnet.private[*].id
  }
}
```

### Data Sources
```hcl
# Get latest AMI
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

# Get availability zones
data "aws_availability_zones" "available" {
  state = "available"
}

# Get current caller identity
data "aws_caller_identity" "current" {}

# Get existing VPC
data "aws_vpc" "existing" {
  filter {
    name   = "tag:Name"
    values = ["production-vpc"]
  }
}
```

### Resources
```hcl
# EC2 Instance
resource "aws_instance" "web" {
  ami                    = data.aws_ami.amazon_linux.id
  instance_type          = var.instance_type
  key_name              = aws_key_pair.deployer.key_name
  vpc_security_group_ids = [aws_security_group.web.id]
  subnet_id             = aws_subnet.public[0].id
  
  user_data = file("${path.module}/userdata.sh")
  
  root_block_device {
    volume_type = "gp3"
    volume_size = 20
    encrypted   = true
  }
  
  tags = merge(var.tags, {
    Name = "web-server-${count.index + 1}"
  })
  
  lifecycle {
    prevent_destroy = true
    ignore_changes  = [ami]
  }
}

# Security Group
resource "aws_security_group" "web" {
  name        = "web-sg"
  description = "Security group for web servers"
  vpc_id      = aws_vpc.main.id
  
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = var.tags
}
```

## Advanced Features

### Loops and Conditionals

#### Count
```hcl
resource "aws_instance" "web" {
  count = var.create_instance ? var.instance_count : 0
  
  ami           = data.aws_ami.amazon_linux.id
  instance_type = var.instance_type
  
  tags = {
    Name = "web-${count.index + 1}"
  }
}
```

#### For Each
```hcl
# For each with set
resource "aws_subnet" "private" {
  for_each = toset(var.private_subnet_cidrs)
  
  vpc_id     = aws_vpc.main.id
  cidr_block = each.value
  
  tags = {
    Name = "private-subnet-${each.key}"
  }
}

# For each with map
variable "environments" {
  type = map(object({
    instance_type = string
    min_size      = number
    max_size      = number
  }))
  
  default = {
    dev = {
      instance_type = "t3.micro"
      min_size      = 1
      max_size      = 2
    }
    prod = {
      instance_type = "t3.large"
      min_size      = 2
      max_size      = 10
    }
  }
}

resource "aws_autoscaling_group" "app" {
  for_each = var.environments
  
  name                = "asg-${each.key}"
  min_size           = each.value.min_size
  max_size           = each.value.max_size
  desired_capacity   = each.value.min_size
  
  launch_template {
    id      = aws_launch_template.app[each.key].id
    version = "$Latest"
  }
}
```

#### Dynamic Blocks
```hcl
resource "aws_security_group" "example" {
  name = "example"
  
  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}

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
```

### Built-in Functions
```hcl
# String functions
locals {
  server_name = upper("${var.environment}-server")
  subnet_name = format("%s-subnet-%02d", var.environment, 1)
  config_file = templatefile("${path.module}/config.tpl", {
    database_host = aws_db_instance.main.endpoint
    app_port      = var.app_port
  })
}

# Collection functions
locals {
  all_subnet_ids = concat(
    aws_subnet.public[*].id,
    aws_subnet.private[*].id
  )
  
  unique_azs = distinct([
    for subnet in aws_subnet.all : subnet.availability_zone
  ])
  
  instance_ips = {
    for instance in aws_instance.web :
    instance.tags.Name => instance.private_ip
  }
}

# Date and time
locals {
  timestamp = timestamp()
  formatted_date = formatdate("YYYY-MM-DD", timestamp())
}

# Type conversion
locals {
  port_string = tostring(var.app_port)
  cidr_list   = tolist(toset(var.cidr_blocks))
  tags_map    = tomap(var.default_tags)
}
```

### Modules

#### Module Structure
```
modules/
  vpc/
    main.tf
    variables.tf
    outputs.tf
    README.md
```

#### Module Definition (modules/vpc/main.tf)
```hcl
resource "aws_vpc" "main" {
  cidr_block           = var.cidr_block
  enable_dns_hostnames = var.enable_dns_hostnames
  enable_dns_support   = var.enable_dns_support
  
  tags = merge(var.tags, {
    Name = var.name
  })
}

resource "aws_subnet" "public" {
  count = length(var.public_subnets)
  
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnets[count.index]
  availability_zone       = var.azs[count.index]
  map_public_ip_on_launch = true
  
  tags = merge(var.tags, {
    Name = "${var.name}-public-${count.index + 1}"
    Type = "Public"
  })
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = merge(var.tags, {
    Name = "${var.name}-igw"
  })
}
```

#### Module Variables (modules/vpc/variables.tf)
```hcl
variable "name" {
  description = "Name of the VPC"
  type        = string
}

variable "cidr_block" {
  description = "CIDR block for VPC"
  type        = string
}

variable "public_subnets" {
  description = "List of public subnet CIDR blocks"
  type        = list(string)
  default     = []
}

variable "azs" {
  description = "List of availability zones"
  type        = list(string)
}

variable "tags" {
  description = "Tags to apply to resources"
  type        = map(string)
  default     = {}
}

variable "enable_dns_hostnames" {
  description = "Enable DNS hostnames in VPC"
  type        = bool
  default     = true
}

variable "enable_dns_support" {
  description = "Enable DNS support in VPC"
  type        = bool
  default     = true
}
```

#### Module Outputs (modules/vpc/outputs.tf)
```hcl
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}

output "vpc_cidr_block" {
  description = "CIDR block of VPC"
  value       = aws_vpc.main.cidr_block
}

output "public_subnet_ids" {
  description = "List of public subnet IDs"
  value       = aws_subnet.public[*].id
}

output "internet_gateway_id" {
  description = "ID of the Internet Gateway"
  value       = aws_internet_gateway.main.id
}
```

#### Using Module
```hcl
module "vpc" {
  source = "./modules/vpc"
  
  name         = "production"
  cidr_block   = "10.0.0.0/16"
  public_subnets = [
    "10.0.1.0/24",
    "10.0.2.0/24"
  ]
  azs = data.aws_availability_zones.available.names
  
  tags = {
    Environment = "production"
    Project     = "my-app"
  }
}

# Reference module outputs
resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"
  subnet_id     = module.vpc.public_subnet_ids[0]
}
```

## Variable Files

### terraform.tfvars
```hcl
# Production variables
environment = "production"
region      = "us-west-2"

instance_type = "t3.large"
instance_count = 3

tags = {
  Environment = "production"
  Project     = "web-app"
  Owner       = "devops-team"
}

database_config = {
  engine         = "postgres"
  engine_version = "13.7"
  instance_class = "db.t3.medium"
  allocated_storage = 100
}

allowed_cidrs = [
  "10.0.0.0/8",
  "172.16.0.0/12"
]
```

### Environment-specific files
```bash
# dev.tfvars
environment = "development"
instance_type = "t3.micro"
instance_count = 1

# staging.tfvars  
environment = "staging"
instance_type = "t3.small"
instance_count = 2

# prod.tfvars
environment = "production"
instance_type = "t3.large"
instance_count = 5
```

## Remote State Backend

### S3 Backend
```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "prod/terraform.tfstate"
    region         = "us-west-2"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"
  }
}
```

### Remote State Data Source
```hcl
data "terraform_remote_state" "vpc" {
  backend = "s3"
  
  config = {
    bucket = "my-terraform-state-bucket"
    key    = "vpc/terraform.tfstate"
    region = "us-west-2"
  }
}

# Use remote state outputs
resource "aws_instance" "web" {
  subnet_id = data.terraform_remote_state.vpc.outputs.public_subnet_ids[0]
}
```

## Best Practices

### Resource Naming Convention
```hcl
locals {
  name_prefix = "${var.project}-${var.environment}"
  
  common_tags = {
    Project     = var.project
    Environment = var.environment
    ManagedBy   = "Terraform"
    CreatedDate = formatdate("YYYY-MM-DD", timestamp())
  }
}

resource "aws_instance" "web" {
  # Use consistent naming
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-web-server"
    Role = "WebServer"
  })
}
```

### Conditional Resources
```hcl
resource "aws_instance" "web" {
  count = var.environment == "production" ? 3 : 1
  
  ami           = data.aws_ami.amazon_linux.id
  instance_type = var.environment == "production" ? "t3.large" : "t3.micro"
  
  tags = {
    Name = "${var.environment}-web-${count.index + 1}"
  }
}
```

### Lifecycle Rules
```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = var.instance_type
  
  lifecycle {
    # Prevent accidental deletion
    prevent_destroy = true
    
    # Ignore changes to AMI (handled by separate process)
    ignore_changes = [ami]
    
    # Create new before destroying old
    create_before_destroy = true
  }
}
```

## Common Patterns

### Multi-Environment Setup
```hcl
# environments/prod/main.tf
module "infrastructure" {
  source = "../../modules/infrastructure"
  
  environment = "production"
  instance_type = "t3.large"
  min_size = 2
  max_size = 10
  
  database_instance_class = "db.r5.large"
  database_multi_az = true
}

# environments/dev/main.tf  
module "infrastructure" {
  source = "../../modules/infrastructure"
  
  environment = "development"
  instance_type = "t3.micro"
  min_size = 1
  max_size = 2
  
  database_instance_class = "db.t3.micro"
  database_multi_az = false
}
```

### Data Transformation
```hcl
# Transform list to map
locals {
  subnet_map = {
    for idx, subnet in var.subnets :
    "subnet-${idx}" => subnet
  }
  
  # Create security group rules from simple list
  ingress_rules = [
    for port in var.allowed_ports : {
      from_port   = port
      to_port     = port
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  ]
}
```

## Troubleshooting Commands

```bash
# Debug terraform execution
export TF_LOG=DEBUG
terraform plan

# Specific log levels
export TF_LOG=TRACE|DEBUG|INFO|WARN|ERROR

# Log to file
export TF_LOG_PATH=terraform.log

# Import existing resources
terraform import aws_instance.web i-1234567890abcdef0

# Unlock state (use carefully)
terraform force-unlock LOCK_ID

# Recover from corrupted state
terraform state pull > terraform.tfstate.backup
terraform state push terraform.tfstate.backup

# Graph dependencies
terraform graph | dot -Tsvg > graph.svg

# Console for testing expressions
terraform console
> var.environment
> local.name_prefix
> length(var.subnets)
```

## Security Best Practices

```hcl
# Use data sources for sensitive values
data "aws_ssm_parameter" "db_password" {
  name = "/myapp/${var.environment}/db_password"
}

resource "aws_db_instance" "main" {
  password = data.aws_ssm_parameter.db_password.value
  # Don't use: password = var.db_password
}

# Mark outputs as sensitive
output "database_password" {
  value     = aws_db_instance.main.password
  sensitive = true
}

# Use random passwords
resource "random_password" "db_password" {
  length  = 16
  special = true
}
```

## Useful Aliases

Add to your shell configuration:
```bash
alias tf='terraform'
alias tfa='terraform apply'
alias tfp='terraform plan'
alias tfd='terraform destroy'
alias tfs='terraform show'
alias tfo='terraform output'
alias tfi='terraform init'
alias tfv='terraform validate'
alias tff='terraform fmt'
```
