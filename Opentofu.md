# OpenTofu Cheat Sheet

## What is OpenTofu?

OpenTofu is an open-source Infrastructure as Code (IaC) tool and a fork of Terraform. It's fully compatible with Terraform configurations and provides additional features while remaining community-driven and truly open-source.

## Core Commands

### Initialization and Planning
```bash
# Initialize OpenTofu (download providers, modules)
tofu init

# Initialize with specific backend config
tofu init -backend-config="bucket=my-tf-state"

# Upgrade providers
tofu init -upgrade

# Validate configuration syntax
tofu validate

# Format configuration files
tofu fmt

# Recursively format all files
tofu fmt -recursive

# Create execution plan
tofu plan

# Create plan with specific var file
tofu plan -var-file="prod.tfvars"

# Save plan to file
tofu plan -out=tfplan

# Show saved plan
tofu show tfplan

# Generate and show plan in JSON format
tofu plan -json
```

### Apply and Destroy
```bash
# Apply changes
tofu apply

# Apply specific plan
tofu apply tfplan

# Apply with auto-approve (skip confirmation)
tofu apply -auto-approve

# Apply with specific variables
tofu apply -var="instance_type=t3.micro"

# Destroy infrastructure
tofu destroy

# Destroy with auto-approve
tofu destroy -auto-approve

# Destroy specific resource
tofu destroy -target=aws_instance.example
```

### State Management
```bash
# Show current state
tofu show

# List resources in state
tofu state list

# Show specific resource
tofu state show aws_instance.example

# Remove resource from state
tofu state rm aws_instance.example

# Import existing resource
tofu import aws_instance.example i-1234567890abcdef0

# Move resource in state
tofu state mv aws_instance.old aws_instance.new

# Refresh state
tofu refresh

# Pull remote state
tofu state pull

# Push local state to remote
tofu state push

# Replace resource (OpenTofu enhanced)
tofu apply -replace=aws_instance.example
```

### Workspace Management
```bash
# List workspaces
tofu workspace list

# Create new workspace
tofu workspace new dev

# Select workspace
tofu workspace select prod

# Show current workspace
tofu workspace show

# Delete workspace
tofu workspace delete dev
```

## OpenTofu-Specific Features

### Enhanced State Backend Support
OpenTofu provides improved backend support and additional backend types:

```hcl
# Enhanced S3 backend with better error handling
terraform {
  backend "s3" {
    bucket         = "my-opentofu-state-bucket"
    key            = "prod/opentofu.tfstate"
    region         = "us-west-2"
    encrypt        = true
    dynamodb_table = "opentofu-state-lock"
    
    # OpenTofu enhanced features
    force_path_style = true
    skip_region_validation = true
  }
}

# Local backend with encryption (OpenTofu feature)
terraform {
  backend "local" {
    path = "terraform.tfstate"
    
    # Encrypt local state files
    encryption {
      method = "aes_gcm"
      keys {
        key_provider = "pbkdf2"
        pbkdf2 {
          passphrase = "your-secure-passphrase"
        }
      }
    }
  }
}
```

### State Encryption
OpenTofu provides native state encryption capabilities:

```hcl
terraform {
  # State encryption configuration
  encryption {
    # Method can be: unencrypted, aes_gcm
    method = "aes_gcm"
    
    keys {
      key_provider = "pbkdf2"
      pbkdf2 {
        passphrase = var.state_passphrase
      }
    }
  }
  
  # Can also use key providers
  encryption {
    method = "aes_gcm"
    keys {
      key_provider = "aws_kms"
      aws_kms {
        kms_key_id = "alias/opentofu-state-key"
        region     = "us-west-2"
      }
    }
  }
}
```

### Enhanced Provider Configuration
```hcl
terraform {
  required_version = ">= 1.6.0"
  
  required_providers {
    aws = {
      source  = "opentofu/aws"
      version = "~> 5.0"
    }
    # OpenTofu registry providers
    custom = {
      source  = "registry.opentofu.org/myorg/custom"
      version = "~> 1.0"
    }
  }
}
```

## Configuration Syntax (Compatible with Terraform)

### Provider Configuration
```hcl
# AWS Provider
provider "aws" {
  region  = "us-west-2"
  profile = "default"
  
  default_tags {
    tags = {
      Environment   = "production"
      Owner         = "devops-team"
      ManagedBy     = "OpenTofu"
      TofuVersion   = "1.6.0"
    }
  }
}

# Multiple provider instances
provider "aws" {
  alias  = "east"
  region = "us-east-1"
}
```

### Variables with Enhanced Validation
```hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
  
  # Enhanced validation in OpenTofu
  validation {
    condition = contains([
      "t3.micro", "t3.small", "t3.medium", 
      "t3.large", "t3.xlarge", "t3.2xlarge"
    ], var.instance_type)
    error_message = "Instance type must be a valid t3 instance type."
  }
  
  validation {
    condition     = can(regex("^t3\\.", var.instance_type))
    error_message = "Only t3 instance types are allowed for cost optimization."
  }
}

variable "environment" {
  description = "Environment name"
  type        = string
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

# Complex variable with nested validation
variable "database_config" {
  description = "Database configuration"
  type = object({
    engine            = string
    engine_version    = string
    instance_class    = string
    allocated_storage = number
    backup_retention  = number
  })
  
  validation {
    condition     = var.database_config.allocated_storage >= 20
    error_message = "Database storage must be at least 20GB."
  }
  
  validation {
    condition     = var.database_config.backup_retention <= 35
    error_message = "Backup retention cannot exceed 35 days."
  }
}
```

### Enhanced Outputs
```hcl
# Outputs with improved functionality
output "instance_details" {
  description = "Complete instance information"
  value = {
    id         = aws_instance.web.id
    public_ip  = aws_instance.web.public_ip
    private_ip = aws_instance.web.private_ip
    dns_name   = aws_instance.web.public_dns
  }
  
  # Enhanced output features
  sensitive = false
  depends_on = [aws_instance.web]
}

# Conditional outputs
output "load_balancer_dns" {
  description = "Load balancer DNS name"
  value       = var.create_load_balancer ? aws_lb.main[0].dns_name : null
}
```

### Advanced Data Sources
```hcl
# Enhanced data source with better error handling
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
  
  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
  
  # OpenTofu enhanced error handling
  lifecycle {
    postcondition {
      condition     = self.architecture == "x86_64"
      error_message = "AMI must be x86_64 architecture."
    }
  }
}

# Data source with complex filtering
data "aws_subnets" "private" {
  filter {
    name   = "vpc-id"
    values = [data.aws_vpc.main.id]
  }
  
  filter {
    name   = "tag:Type"
    values = ["Private"]
  }
  
  # Ensure we have subnets
  lifecycle {
    postcondition {
      condition     = length(self.ids) > 0
      error_message = "No private subnets found in the VPC."
    }
  }
}
```

## Advanced OpenTofu Features

### Enhanced For Expressions
```hcl
locals {
  # Advanced data transformation
  instance_configs = {
    for env in var.environments : env => {
      instance_type = env == "prod" ? "t3.large" : "t3.micro"
      min_size     = env == "prod" ? 2 : 1
      max_size     = env == "prod" ? 10 : 3
      
      # Conditional security groups
      security_groups = concat(
        [aws_security_group.base.id],
        env == "prod" ? [aws_security_group.prod_only.id] : []
      )
    }
  }
  
  # Complex subnet mapping
  subnet_mapping = {
    for idx, subnet in data.aws_subnets.all.ids : 
    format("subnet-%02d", idx + 1) => {
      id = subnet
      az = data.aws_subnet.details[subnet].availability_zone
      cidr = data.aws_subnet.details[subnet].cidr_block
    }
  }
}
```

### Improved Error Handling and Validation
```hcl
resource "aws_instance" "web" {
  count = var.instance_count
  
  ami           = data.aws_ami.amazon_linux.id
  instance_type = var.instance_type
  subnet_id     = element(data.aws_subnets.private.ids, count.index)
  
  # Enhanced lifecycle with postconditions
  lifecycle {
    precondition {
      condition     = var.instance_count <= length(data.aws_subnets.private.ids)
      error_message = "Instance count cannot exceed available private subnets."
    }
    
    postcondition {
      condition     = self.instance_state == "running"
      error_message = "Instance failed to reach running state."
    }
  }
  
  tags = {
    Name = "web-server-${count.index + 1}"
  }
}
```

### Enhanced Module System
```hcl
# Module with improved validation
module "vpc" {
  source = "registry.opentofu.org/modules/vpc"
  
  name             = var.vpc_name
  cidr             = var.vpc_cidr
  azs              = data.aws_availability_zones.available.names
  private_subnets  = var.private_subnet_cidrs
  public_subnets   = var.public_subnet_cidrs
  
  # Enhanced module features
  enable_nat_gateway = true
  enable_vpn_gateway = false
  
  # Module-level validation
  lifecycle {
    precondition {
      condition = length(var.private_subnet_cidrs) == length(var.public_subnet_cidrs)
      error_message = "Number of private and public subnets must match."
    }
  }
}
```

## OpenTofu Testing Framework

### Test Configuration
```hcl
# tests/vpc_test.tftest.hcl
variables {
  vpc_name = "test-vpc"
  vpc_cidr = "10.0.0.0/16"
  private_subnet_cidrs = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnet_cidrs  = ["10.0.101.0/24", "10.0.102.0/24"]
}

run "vpc_creation" {
  command = plan
  
  assert {
    condition     = module.vpc.vpc_cidr_block == "10.0.0.0/16"
    error_message = "VPC CIDR block did not match expected value"
  }
  
  assert {
    condition     = length(module.vpc.private_subnet_ids) == 2
    error_message = "Expected 2 private subnets"
  }
}

run "vpc_validation" {
  command = apply
  
  assert {
    condition     = can(regex("^vpc-", module.vpc.vpc_id))
    error_message = "VPC ID does not have expected format"
  }
}
```

### Running Tests
```bash
# Run tests
tofu test

# Run specific test file
tofu test tests/vpc_test.tftest.hcl

# Run tests with verbose output
tofu test -verbose

# Run tests for specific configuration
tofu test -var-file="test.tfvars"
```

## Migration from Terraform

### Migration Commands
```bash
# Convert Terraform state to OpenTofu
tofu init -migrate-state

# Verify state migration
tofu plan

# Update provider sources in configuration
# Change from: source = "hashicorp/aws"
# To: source = "opentofu/aws"
```

### Compatibility Script
```bash
#!/bin/bash
# migrate-to-opentofu.sh

# Backup existing state
cp terraform.tfstate terraform.tfstate.backup

# Update provider sources
find . -name "*.tf" -exec sed -i 's/hashicorp\//opentofu\//g' {} \;

# Initialize with OpenTofu
tofu init -migrate-state

# Verify migration
tofu plan

echo "Migration completed. Review plan output before applying."
```

## State Encryption Examples

### File-based Encryption
```hcl
terraform {
  encryption {
    method = "aes_gcm"
    
    keys {
      key_provider = "pbkdf2"
      pbkdf2 {
        passphrase = var.encryption_passphrase
      }
    }
  }
}

variable "encryption_passphrase" {
  description = "Passphrase for state encryption"
  type        = string
  sensitive   = true
}
```

### Cloud KMS Encryption
```hcl
terraform {
  encryption {
    method = "aes_gcm"
    
    # AWS KMS
    keys {
      key_provider = "aws_kms"
      aws_kms {
        kms_key_id = "alias/opentofu-state"
        region     = "us-west-2"
      }
    }
    
    # Alternative: GCP KMS
    # keys {
    #   key_provider = "gcp_kms"
    #   gcp_kms {
    #     kms_encryption_key = "projects/my-project/locations/global/keyRings/my-ring/cryptoKeys/my-key"
    #   }
    # }
  }
}
```

## Best Practices for OpenTofu

### Project Structure
```
project/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   └── prod/
├── modules/
│   ├── vpc/
│   ├── compute/
│   └── database/
├── tests/
│   ├── vpc_test.tftest.hcl
│   └── compute_test.tftest.hcl
└── scripts/
    └── migrate-to-opentofu.sh
```

### Configuration Standards
```hcl
# Always specify OpenTofu version requirements
terraform {
  required_version = ">= 1.6.0, < 2.0.0"
  
  required_providers {
    aws = {
      source  = "opentofu/aws"
      version = "~> 5.0"
    }
  }
  
  # Use state encryption
  encryption {
    method = "aes_gcm"
    keys {
      key_provider = "pbkdf2"
      pbkdf2 {
        passphrase = var.state_passphrase
      }
    }
  }
}

# Standard locals for naming and tagging
locals {
  name_prefix = "${var.project}-${var.environment}"
  
  common_tags = {
    Project       = var.project
    Environment   = var.environment
    ManagedBy     = "OpenTofu"
    TofuVersion   = "1.6.0"
    LastModified  = timestamp()
  }
}
```

### Security Best Practices
```hcl
# Use data sources for sensitive values
data "aws_ssm_parameter" "db_password" {
  name = "/${var.project}/${var.environment}/db/password"
}

# Generate secure random passwords
resource "random_password" "admin_password" {
  length  = 32
  special = true
  
  lifecycle {
    ignore_changes = [length, special]
  }
}

# Store secrets securely
resource "aws_secretsmanager_secret" "app_secrets" {
  name = "${local.name_prefix}-app-secrets"
  
  # Enable automatic rotation
  rotation_rules {
    automatically_after_days = 30
  }
  
  tags = local.common_tags
}
```

## Troubleshooting OpenTofu

### Debug Commands
```bash
# Enable debug logging
export TF_LOG=DEBUG
export TF_LOG_PATH=opentofu.log

# Verify state integrity
tofu state list
tofu state show aws_instance.web

# Force state refresh
tofu refresh

# Validate configuration
tofu validate

# Check provider versions
tofu version
tofu providers

# Graph dependencies
tofu graph | dot -Tsvg > dependencies.svg
```

### Common Issues and Solutions

#### State Migration Issues
```bash
# If state migration fails
tofu init -reconfigure

# Force backend reconfiguration
tofu init -migrate-state -force-copy

# Manual state manipulation
tofu state pull > backup.tfstate
# Edit state file if necessary
tofu state push backup.tfstate
```

#### Provider Issues
```bash
# Clear provider cache
rm -rf .terraform
tofu init

# Force provider download
tofu init -upgrade

# Check provider compatibility
tofu providers schema -json
```

## Performance Optimization

### Parallel Execution
```bash
# Increase parallelism
tofu apply -parallelism=20

# Reduce parallelism for resource limits
tofu apply -parallelism=5
```

### Resource Targeting
```bash
# Apply specific resources only
tofu apply -target=module.vpc
tofu apply -target=aws_instance.web[0]

# Plan specific resources
tofu plan -target=module.database
```

## Useful Aliases for OpenTofu

```bash
# Add to your shell configuration
alias tf='tofu'
alias tfa='tofu apply'
alias tfp='tofu plan'
alias tfd='tofu destroy'
alias tfs='tofu show'
alias tfo='tofu output'
alias tfi='tofu init'
alias tfv='tofu validate'
alias tff='tofu fmt'
alias tft='tofu test'
alias tfw='tofu workspace'
```

## OpenTofu vs Terraform Comparison

| Feature | OpenTofu | Terraform |
|---------|----------|-----------|
| License | MPL 2.0 (Open Source) | BSL (Business Source) |
| State Encryption | Built-in | Third-party only |
| Testing Framework | Native support | Third-party tools |
| Community Governance | Open governance | HashiCorp controlled |
| Provider Registry | registry.opentofu.org | registry.terraform.io |
| Compatibility | Fully compatible | N/A |
| Cost | Free forever | Free tier + paid |
