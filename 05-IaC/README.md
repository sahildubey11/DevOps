# Infrastructure as Code (IaC)

## What is Infrastructure as Code?

Infrastructure as Code (IaC) is the practice of managing and provisioning infrastructure through machine-readable definition files, rather than physical hardware configuration or interactive configuration tools.

## Why IaC?

- **Version Control**: Track infrastructure changes like code
- **Consistency**: Eliminate configuration drift
- **Repeatability**: Deploy same infrastructure multiple times
- **Automation**: Reduce manual errors
- **Documentation**: Code serves as documentation
- **Speed**: Faster provisioning and deployment
- **Collaboration**: Team can review and collaborate

## IaC Tools

### 1. Terraform
- Multi-cloud support
- Declarative syntax
- State management
- Large provider ecosystem

### 2. Ansible
- Agentless
- YAML-based
- Configuration management
- Application deployment

### 3. CloudFormation (AWS)
- AWS-specific
- Native AWS integration
- JSON/YAML templates

### 4. Pulumi
- Real programming languages
- Multi-cloud
- State management

## Terraform

### Installation

```bash
# Ubuntu/Debian
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

# macOS
brew install terraform

# Verify installation
terraform version
```

### Basic Terraform Workflow

```bash
# Initialize Terraform
terraform init

# Validate configuration
terraform validate

# Plan changes
terraform plan

# Apply changes
terraform apply

# Destroy infrastructure
terraform destroy
```

### Basic Terraform Configuration

**main.tf**
```hcl
# Configure the AWS Provider
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "main-vpc"
  }
}

# Create Subnet
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true

  tags = {
    Name = "public-subnet"
  }
}

# Create Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "main-igw"
  }
}

# Create Security Group
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

  tags = {
    Name = "web-sg"
  }
}

# Create EC2 Instance
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.public.id
  
  vpc_security_group_ids = [aws_security_group.web.id]

  tags = {
    Name = "web-server"
  }
}
```

### Variables

**variables.tf**
```hcl
variable "region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

variable "environment" {
  description = "Environment name"
  type        = string
}

variable "allowed_ips" {
  description = "Allowed IP addresses"
  type        = list(string)
  default     = ["0.0.0.0/0"]
}

variable "tags" {
  description = "Common tags"
  type        = map(string)
  default     = {}
}
```

### Outputs

**outputs.tf**
```hcl
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}

output "instance_public_ip" {
  description = "Public IP of the instance"
  value       = aws_instance.web.public_ip
}

output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.web.id
}
```

### Terraform State

**backend.tf**
```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-lock"
  }
}
```

### Modules

**modules/vpc/main.tf**
```hcl
variable "cidr_block" {
  type = string
}

variable "name" {
  type = string
}

resource "aws_vpc" "this" {
  cidr_block = var.cidr_block

  tags = {
    Name = var.name
  }
}

output "vpc_id" {
  value = aws_vpc.this.id
}
```

**Using Module**
```hcl
module "vpc" {
  source     = "./modules/vpc"
  cidr_block = "10.0.0.0/16"
  name       = "production-vpc"
}

resource "aws_subnet" "public" {
  vpc_id     = module.vpc.vpc_id
  cidr_block = "10.0.1.0/24"
}
```

### Advanced Terraform

**Data Sources**
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

**Loops**
```hcl
# Count
resource "aws_instance" "web" {
  count         = 3
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name = "web-${count.index}"
  }
}

# For Each
resource "aws_instance" "web" {
  for_each = toset(["dev", "staging", "prod"])

  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name        = "web-${each.key}"
    Environment = each.key
  }
}
```

**Conditionals**
```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = var.environment == "prod" ? "t2.large" : "t2.micro"
}
```

## Ansible

### Installation

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install ansible

# macOS
brew install ansible

# Verify
ansible --version
```

### Inventory File

**inventory.ini**
```ini
[webservers]
web1.example.com
web2.example.com

[databases]
db1.example.com
db2.example.com

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

**inventory.yml**
```yaml
all:
  children:
    webservers:
      hosts:
        web1.example.com:
        web2.example.com:
    databases:
      hosts:
        db1.example.com:
        db2.example.com:
  vars:
    ansible_user: ubuntu
    ansible_ssh_private_key_file: ~/.ssh/id_rsa
```

### Ad-hoc Commands

```bash
# Ping all hosts
ansible all -m ping -i inventory.ini

# Run command
ansible webservers -m command -a "uptime"

# Copy file
ansible webservers -m copy -a "src=/local/file dest=/remote/file"

# Install package
ansible webservers -m apt -a "name=nginx state=present" --become

# Restart service
ansible webservers -m service -a "name=nginx state=restarted" --become

# Gather facts
ansible all -m setup
```

### Playbook

**playbook.yml**
```yaml
---
- name: Configure Web Servers
  hosts: webservers
  become: yes
  
  vars:
    nginx_port: 80
    app_name: myapp

  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600

    - name: Install Nginx
      apt:
        name: nginx
        state: present

    - name: Start and enable Nginx
      service:
        name: nginx
        state: started
        enabled: yes

    - name: Copy configuration file
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Restart Nginx

    - name: Copy application files
      copy:
        src: "{{ app_name }}/"
        dest: "/var/www/{{ app_name }}"

  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

### Run Playbook

```bash
# Run playbook
ansible-playbook -i inventory.ini playbook.yml

# Check mode (dry run)
ansible-playbook -i inventory.ini playbook.yml --check

# With extra variables
ansible-playbook -i inventory.ini playbook.yml -e "nginx_port=8080"

# Limit to specific hosts
ansible-playbook -i inventory.ini playbook.yml --limit web1.example.com

# With tags
ansible-playbook -i inventory.ini playbook.yml --tags "config"
```

### Templates (Jinja2)

**nginx.conf.j2**
```jinja2
server {
    listen {{ nginx_port }};
    server_name {{ ansible_hostname }};

    location / {
        root /var/www/{{ app_name }};
        index index.html;
    }

    {% if environment == 'production' %}
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
    {% endif %}
}
```

### Roles

**Directory Structure**
```
roles/
  webserver/
    tasks/
      main.yml
    handlers/
      main.yml
    templates/
      nginx.conf.j2
    files/
      index.html
    vars/
      main.yml
    defaults/
      main.yml
```

**roles/webserver/tasks/main.yml**
```yaml
---
- name: Install Nginx
  apt:
    name: nginx
    state: present

- name: Configure Nginx
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: Restart Nginx

- name: Start Nginx
  service:
    name: nginx
    state: started
    enabled: yes
```

**Using Roles**
```yaml
---
- name: Configure Servers
  hosts: webservers
  become: yes
  
  roles:
    - webserver
    - security
    - monitoring
```

### Ansible Vault (Secrets)

```bash
# Create encrypted file
ansible-vault create secrets.yml

# Edit encrypted file
ansible-vault edit secrets.yml

# Encrypt existing file
ansible-vault encrypt vars.yml

# Decrypt file
ansible-vault decrypt vars.yml

# Run playbook with vault
ansible-playbook playbook.yml --ask-vault-pass

# Use vault password file
ansible-playbook playbook.yml --vault-password-file ~/.vault_pass
```

## Best Practices

### Terraform Best Practices

1. **Use Version Control**: Store all Terraform code in Git
2. **Remote State**: Use remote backend (S3, Azure Blob)
3. **State Locking**: Prevent concurrent modifications
4. **Modules**: Reuse code with modules
5. **Variables**: Parameterize configurations
6. **Workspaces**: Manage multiple environments
7. **Formatting**: Use `terraform fmt`
8. **Validation**: Run `terraform validate`
9. **Plan Review**: Always review plans before apply
10. **Secrets**: Never commit secrets to version control

### Ansible Best Practices

1. **Use Roles**: Organize playbooks with roles
2. **Idempotency**: Ensure tasks can run multiple times
3. **Variables**: Use group_vars and host_vars
4. **Vault**: Encrypt sensitive data
5. **Tags**: Use tags for selective execution
6. **Handlers**: Use for service restarts
7. **Templates**: Use Jinja2 templates
8. **Testing**: Test playbooks in dev environment
9. **Documentation**: Document roles and playbooks
10. **Version Control**: Track all Ansible code

## Comparison: Terraform vs Ansible

| Feature | Terraform | Ansible |
|---------|-----------|---------|
| Primary Use | Infrastructure provisioning | Configuration management |
| Approach | Declarative | Procedural/Declarative |
| State | Maintains state | Stateless |
| Language | HCL | YAML |
| Agent | Agentless | Agentless |
| Multi-cloud | Excellent | Good |
| Learning Curve | Moderate | Easy |

## Interview Questions

1. What is Infrastructure as Code?
2. Explain the benefits of IaC
3. What is Terraform state and why is it important?
4. How do you manage secrets in Terraform?
5. What are Terraform modules and why use them?
6. Explain Ansible playbooks and roles
7. What is idempotency in Ansible?
8. How does Ansible Vault work?
9. What is the difference between Terraform and Ansible?
10. How do you handle multiple environments in IaC?

## Practical Exercises

1. Create a VPC with subnets using Terraform
2. Deploy a multi-tier application with Terraform
3. Create reusable Terraform modules
4. Write an Ansible playbook to configure web servers
5. Create Ansible roles for common tasks
6. Use Ansible Vault to manage secrets
7. Implement a complete infrastructure with both tools
8. Set up remote state management for Terraform

## Resources

- [Terraform Documentation](https://www.terraform.io/docs)
- [Terraform Registry](https://registry.terraform.io/)
- [Ansible Documentation](https://docs.ansible.com/)
- [Ansible Galaxy](https://galaxy.ansible.com/)
- [Infrastructure as Code Book](https://www.oreilly.com/library/view/infrastructure-as-code/9781491924358/)
- [Terraform Up & Running](https://www.terraformupandrunning.com/)
