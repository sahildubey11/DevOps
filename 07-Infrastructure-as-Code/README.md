# Infrastructure as Code (IaC)

## What is Infrastructure as Code?

Infrastructure as Code (IaC) is the practice of managing and provisioning infrastructure through machine-readable definition files, rather than manual configuration or interactive tools.

## Benefits of IaC

### 1. Version Control
- Track infrastructure changes in Git
- Review history and audit trail
- Rollback to previous versions
- Branch and merge infrastructure changes

### 2. Consistency
- Same configuration every time
- Eliminate configuration drift
- No manual errors
- Reproducible environments

### 3. Automation
- Automated provisioning
- Faster deployment
- Self-service infrastructure
- CI/CD integration

### 4. Documentation
- Code documents infrastructure
- Always up-to-date
- Easier onboarding
- Knowledge sharing

### 5. Cost Management
- Easily destroy resources
- Test environments on-demand
- Resource optimization
- Track infrastructure costs

### 6. Disaster Recovery
- Rebuild infrastructure quickly
- Infrastructure backup in Git
- Multi-region deployment
- Business continuity

## IaC Tools

### Terraform (HashiCorp)
- Multi-cloud support
- Declarative HCL language
- State management
- Large provider ecosystem

### CloudFormation (AWS)
- AWS-specific
- JSON or YAML templates
- Native AWS integration
- No additional tooling needed

### Pulumi
- Use programming languages (TypeScript, Python, Go, C#)
- Multi-cloud
- State management
- Modern development workflows

### Ansible
- Configuration management + provisioning
- YAML-based
- Agentless
- Wide module support

## Terraform

### Installation

```bash
# macOS
brew tap hashicorp/tap
brew install hashicorp/tap/terraform

# Linux
wget https://releases.hashicorp.com/terraform/1.5.0/terraform_1.5.0_linux_amd64.zip
unzip terraform_1.5.0_linux_amd64.zip
sudo mv terraform /usr/local/bin/

# Verify
terraform version
```

### Core Concepts

#### Providers
Plugins that interact with cloud providers, SaaS providers, and APIs.

```hcl
# AWS Provider
provider "aws" {
  region = "us-east-1"
  profile = "default"
}

# Azure Provider
provider "azurerm" {
  features {}
}

# Google Cloud Provider
provider "google" {
  project = "my-project"
  region  = "us-central1"
}
```

#### Resources
Infrastructure components (VMs, networks, storage, etc.).

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  tags = {
    Name = "WebServer"
  }
}
```

#### Variables
Parameterize your configurations.

```hcl
# variables.tf
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

variable "environment" {
  description = "Environment name"
  type        = string
}

variable "instance_count" {
  description = "Number of instances"
  type        = number
  default     = 1
}

variable "enable_monitoring" {
  description = "Enable monitoring"
  type        = bool
  default     = true
}

# Using variables
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = var.instance_type
  count         = var.instance_count
  
  tags = {
    Name = "${var.environment}-web"
  }
}
```

#### Outputs
Extract information from resources.

```hcl
output "instance_ip" {
  description = "Public IP of instance"
  value       = aws_instance.web.public_ip
}

output "instance_id" {
  value = aws_instance.web.id
}
```

#### Data Sources
Query existing infrastructure.

```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
}
```

#### Modules
Reusable Terraform configurations.

```hcl
# modules/vpc/main.tf
variable "cidr_block" {
  type = string
}

resource "aws_vpc" "main" {
  cidr_block = var.cidr_block
  
  tags = {
    Name = "main-vpc"
  }
}

output "vpc_id" {
  value = aws_vpc.main.id
}

# Using the module
module "network" {
  source     = "./modules/vpc"
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public" {
  vpc_id     = module.network.vpc_id
  cidr_block = "10.0.1.0/24"
}
```

### Terraform Workflow

```bash
# Initialize working directory
terraform init

# Validate configuration
terraform validate

# Format code
terraform fmt

# Plan changes
terraform plan
terraform plan -out=tfplan

# Apply changes
terraform apply
terraform apply tfplan
terraform apply -auto-approve

# Show current state
terraform show

# List resources in state
terraform state list

# Show specific resource
terraform state show aws_instance.web

# Destroy infrastructure
terraform destroy
terraform destroy -target=aws_instance.web
```

### Project Structure

```
terraform-project/
├── main.tf              # Main configuration
├── variables.tf         # Variable definitions
├── outputs.tf           # Output definitions
├── providers.tf         # Provider configuration
├── terraform.tfvars     # Variable values
├── versions.tf          # Version constraints
├── modules/             # Reusable modules
│   ├── vpc/
│   ├── ec2/
│   └── rds/
└── environments/        # Environment-specific configs
    ├── dev/
    ├── staging/
    └── production/
```

### Example: Complete AWS Infrastructure

```hcl
# providers.tf
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
    region = "us-east-1"
    encrypt = true
  }
}

provider "aws" {
  region = var.aws_region
}

# variables.tf
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "environment" {
  description = "Environment name"
  type        = string
  default     = "production"
}

variable "vpc_cidr" {
  description = "VPC CIDR block"
  type        = string
  default     = "10.0.0.0/16"
}

# main.tf
# VPC
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name        = "${var.environment}-vpc"
    Environment = var.environment
  }
}

# Subnet
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "${var.aws_region}a"
  map_public_ip_on_launch = true
  
  tags = {
    Name = "${var.environment}-public-subnet"
  }
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = {
    Name = "${var.environment}-igw"
  }
}

# Route Table
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
  
  tags = {
    Name = "${var.environment}-public-rt"
  }
}

resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}

# Security Group
resource "aws_security_group" "web" {
  name        = "${var.environment}-web-sg"
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
  
  tags = {
    Name = "${var.environment}-web-sg"
  }
}

# EC2 Instance
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.public.id
  
  vpc_security_group_ids = [aws_security_group.web.id]
  
  user_data = <<-EOF
              #!/bin/bash
              apt-get update
              apt-get install -y nginx
              systemctl start nginx
              systemctl enable nginx
              EOF
  
  tags = {
    Name        = "${var.environment}-web-server"
    Environment = var.environment
  }
}

data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
}

# outputs.tf
output "vpc_id" {
  description = "VPC ID"
  value       = aws_vpc.main.id
}

output "web_server_public_ip" {
  description = "Public IP of web server"
  value       = aws_instance.web.public_ip
}

output "web_server_url" {
  description = "URL of web server"
  value       = "http://${aws_instance.web.public_ip}"
}
```

### State Management

#### Remote State (S3 Backend)

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```

#### State Commands

```bash
# Show state
terraform show

# List resources
terraform state list

# Show resource
terraform state show aws_instance.web

# Move resource in state
terraform state mv aws_instance.web aws_instance.web_server

# Remove resource from state
terraform state rm aws_instance.web

# Pull remote state
terraform state pull

# Push local state
terraform state push

# Import existing resource
terraform import aws_instance.web i-1234567890abcdef0
```

### Workspaces

Manage multiple environments with same configuration.

```bash
# List workspaces
terraform workspace list

# Create workspace
terraform workspace new dev
terraform workspace new staging
terraform workspace new production

# Select workspace
terraform workspace select dev

# Show current workspace
terraform workspace show

# Delete workspace
terraform workspace delete dev
```

### Best Practices

#### 1. Use Modules
```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"
  
  name = "my-vpc"
  cidr = "10.0.0.0/16"
  
  azs             = ["us-east-1a", "us-east-1b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]
  
  enable_nat_gateway = true
}
```

#### 2. Use Variables and Locals

```hcl
locals {
  common_tags = {
    Environment = var.environment
    ManagedBy   = "Terraform"
    Project     = "MyApp"
  }
}

resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = var.instance_type
  
  tags = merge(
    local.common_tags,
    {
      Name = "web-server"
    }
  )
}
```

#### 3. Use Data Sources

```hcl
data "aws_availability_zones" "available" {
  state = "available"
}

resource "aws_subnet" "public" {
  count = length(data.aws_availability_zones.available.names)
  
  availability_zone = data.aws_availability_zones.available.names[count.index]
  cidr_block        = cidrsubnet(var.vpc_cidr, 8, count.index)
  vpc_id            = aws_vpc.main.id
}
```

#### 4. Use Remote State

Store state in S3, Azure Storage, or Terraform Cloud.

#### 5. Lock State

Use DynamoDB for state locking (prevents concurrent modifications).

#### 6. Use .gitignore

```gitignore
# .gitignore
.terraform/
*.tfstate
*.tfstate.backup
*.tfvars
.terraform.lock.hcl
```

#### 7. Pin Provider Versions

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

#### 8. Use terraform fmt and validate

```bash
terraform fmt -recursive
terraform validate
```

#### 9. Plan Before Apply

```bash
terraform plan -out=tfplan
terraform apply tfplan
```

#### 10. Use Meaningful Names

```hcl
# Good
resource "aws_instance" "web_server" {
  # ...
}

# Bad
resource "aws_instance" "i" {
  # ...
}
```

### Terraform Cloud

Features:
- Remote state management
- Collaborative workflows
- Version control integration
- Private module registry
- Sentinel policy as code
- Cost estimation

### Advanced Features

#### For Each

```hcl
variable "users" {
  type = set(string)
  default = ["alice", "bob", "charlie"]
}

resource "aws_iam_user" "users" {
  for_each = var.users
  name     = each.value
}
```

#### Count

```hcl
resource "aws_instance" "web" {
  count = 3
  
  ami           = var.ami_id
  instance_type = "t2.micro"
  
  tags = {
    Name = "web-${count.index}"
  }
}
```

#### Conditional Expressions

```hcl
resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = var.environment == "production" ? "t2.large" : "t2.micro"
}
```

#### Dynamic Blocks

```hcl
resource "aws_security_group" "web" {
  name = "web-sg"
  
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
```

## CloudFormation (AWS)

### Example Template

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Web Server Stack

Parameters:
  InstanceType:
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t2.small
      - t2.medium
  
  KeyName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: EC2 KeyPair

Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: MyVPC
  
  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.1.0/24
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: PublicSubnet
  
  InternetGateway:
    Type: AWS::EC2::InternetGateway
  
  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway
  
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: !Ref InstanceType
      KeyName: !Ref KeyName
      SubnetId: !Ref PublicSubnet
      Tags:
        - Key: Name
          Value: WebServer

Outputs:
  VPCId:
    Description: VPC ID
    Value: !Ref VPC
  
  WebServerPublicIP:
    Description: Public IP of web server
    Value: !GetAtt WebServer.PublicIp
```

### CloudFormation Commands

```bash
# Create stack
aws cloudformation create-stack \
  --stack-name my-stack \
  --template-body file://template.yaml \
  --parameters ParameterKey=InstanceType,ParameterValue=t2.small

# Update stack
aws cloudformation update-stack \
  --stack-name my-stack \
  --template-body file://template.yaml

# Delete stack
aws cloudformation delete-stack --stack-name my-stack

# Describe stack
aws cloudformation describe-stacks --stack-name my-stack

# List stacks
aws cloudformation list-stacks
```

## Comparison: Terraform vs CloudFormation

| Feature | Terraform | CloudFormation |
|---------|-----------|----------------|
| Cloud Support | Multi-cloud | AWS only |
| Language | HCL | JSON/YAML |
| State Management | External required | Managed by AWS |
| Preview Changes | terraform plan | Change sets |
| Module Registry | Public registry | Limited |
| Learning Curve | Medium | Easy (for AWS users) |
| Community | Large | AWS-focused |

## Resources

- Terraform Documentation: https://www.terraform.io/docs/
- Terraform Registry: https://registry.terraform.io/
- CloudFormation Docs: https://docs.aws.amazon.com/cloudformation/
- Pulumi: https://www.pulumi.com/docs/
- Infrastructure as Code Book: https://www.oreilly.com/library/view/infrastructure-as-code/9781098114664/

## Next Steps

Continue to [Monitoring and Logging](../08-Monitoring-and-Logging/) to learn about observability in DevOps.
