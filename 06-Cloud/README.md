# Cloud Platforms

## Overview

Cloud computing provides on-demand access to computing resources over the internet. The three major cloud providers are AWS, Azure, and Google Cloud Platform (GCP).

## Cloud Service Models

### 1. IaaS (Infrastructure as a Service)
- Virtual machines, storage, networks
- Examples: EC2, Azure VMs, Google Compute Engine
- Most control, most management

### 2. PaaS (Platform as a Service)
- Platform for developing and deploying applications
- Examples: Elastic Beanstalk, Azure App Service, Google App Engine
- Less management, faster development

### 3. SaaS (Software as a Service)
- Complete applications over the internet
- Examples: Gmail, Office 365, Salesforce
- No management, ready to use

## Amazon Web Services (AWS)

### Core Services

#### Compute
- **EC2**: Virtual servers
- **Lambda**: Serverless functions
- **ECS**: Container service
- **EKS**: Kubernetes service
- **Elastic Beanstalk**: PaaS

#### Storage
- **S3**: Object storage
- **EBS**: Block storage for EC2
- **EFS**: Elastic file system
- **Glacier**: Archive storage

#### Database
- **RDS**: Relational databases (MySQL, PostgreSQL, etc.)
- **DynamoDB**: NoSQL database
- **Aurora**: High-performance MySQL/PostgreSQL
- **ElastiCache**: In-memory cache (Redis, Memcached)

#### Networking
- **VPC**: Virtual private cloud
- **Route 53**: DNS service
- **CloudFront**: CDN
- **ELB**: Load balancing

#### Security
- **IAM**: Identity and access management
- **KMS**: Key management service
- **Secrets Manager**: Manage secrets
- **WAF**: Web application firewall

### AWS CLI Basics

```bash
# Configure AWS CLI
aws configure

# EC2 Examples
aws ec2 describe-instances
aws ec2 start-instances --instance-ids i-1234567890abcdef0
aws ec2 stop-instances --instance-ids i-1234567890abcdef0

# S3 Examples
aws s3 ls
aws s3 cp file.txt s3://my-bucket/
aws s3 sync ./local-folder s3://my-bucket/

# IAM Examples
aws iam list-users
aws iam create-user --user-name newuser
aws iam list-roles
```

### AWS Architecture Example

```
Internet Gateway
       |
    VPC (10.0.0.0/16)
       |
   +---+---+
   |       |
Public    Private
Subnet    Subnet
   |       |
  ALB     EC2
   |       |
  ASG     RDS
```

## Microsoft Azure

### Core Services

#### Compute
- **Virtual Machines**: IaaS compute
- **App Service**: PaaS for web apps
- **Azure Functions**: Serverless
- **AKS**: Azure Kubernetes Service
- **Container Instances**: Run containers

#### Storage
- **Blob Storage**: Object storage
- **Disk Storage**: Block storage
- **File Storage**: File shares
- **Archive Storage**: Cold storage

#### Database
- **Azure SQL Database**: Managed SQL
- **Cosmos DB**: NoSQL database
- **Database for MySQL/PostgreSQL**: Managed databases
- **Redis Cache**: In-memory cache

#### Networking
- **Virtual Network**: VNet
- **Application Gateway**: Load balancer
- **CDN**: Content delivery
- **DNS**: Domain name service

#### Security
- **Azure AD**: Identity management
- **Key Vault**: Manage secrets
- **Security Center**: Security management

### Azure CLI Basics

```bash
# Login
az login

# Resource Groups
az group create --name myResourceGroup --location eastus
az group list
az group delete --name myResourceGroup

# Virtual Machines
az vm create --resource-group myResourceGroup --name myVM --image UbuntuLTS
az vm list
az vm start --resource-group myResourceGroup --name myVM
az vm stop --resource-group myResourceGroup --name myVM

# Storage
az storage account create --name mystorageaccount --resource-group myResourceGroup
az storage blob upload --account-name mystorageaccount --container-name mycontainer --name myblob --file myfile.txt
```

## Google Cloud Platform (GCP)

### Core Services

#### Compute
- **Compute Engine**: Virtual machines
- **Cloud Functions**: Serverless
- **Cloud Run**: Containerized apps
- **GKE**: Google Kubernetes Engine
- **App Engine**: PaaS

#### Storage
- **Cloud Storage**: Object storage
- **Persistent Disk**: Block storage
- **Filestore**: File storage

#### Database
- **Cloud SQL**: Managed relational databases
- **Cloud Spanner**: Globally distributed database
- **Firestore**: NoSQL document database
- **Bigtable**: Wide-column NoSQL

#### Networking
- **VPC**: Virtual private cloud
- **Cloud Load Balancing**: Load balancer
- **Cloud CDN**: Content delivery
- **Cloud DNS**: DNS service

#### Security
- **IAM**: Identity and access management
- **Secret Manager**: Manage secrets
- **Cloud Armor**: DDoS protection

### gcloud CLI Basics

```bash
# Initialize
gcloud init

# Compute Engine
gcloud compute instances list
gcloud compute instances create my-instance --zone=us-central1-a
gcloud compute instances start my-instance --zone=us-central1-a
gcloud compute instances stop my-instance --zone=us-central1-a

# Storage
gsutil ls
gsutil cp file.txt gs://my-bucket/
gsutil rsync -r ./local-folder gs://my-bucket/

# Container Registry
gcloud builds submit --tag gcr.io/my-project/my-image
```

## Multi-Cloud Strategy

### Benefits
- Avoid vendor lock-in
- Leverage best services from each provider
- Geographic distribution
- Cost optimization
- Disaster recovery

### Challenges
- Complexity
- Different APIs and tools
- Security management
- Cost tracking
- Skills requirements

## Cloud Design Patterns

### 1. High Availability
- Multi-AZ deployment
- Load balancing
- Auto-scaling
- Health checks

### 2. Disaster Recovery
- Backup strategies
- Multi-region replication
- Failover mechanisms
- Recovery time objectives (RTO/RPO)

### 3. Cost Optimization
- Right-sizing instances
- Reserved instances
- Spot/preemptible instances
- Auto-scaling
- Storage lifecycle policies

### 4. Security
- Least privilege access
- Network segmentation
- Encryption at rest and in transit
- Security monitoring
- Compliance

## Infrastructure Deployment on AWS

### Example: Web Application on AWS

```hcl
# Terraform configuration
provider "aws" {
  region = "us-east-1"
}

# VPC
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

# Subnets
resource "aws_subnet" "public" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-east-1a"
}

# Application Load Balancer
resource "aws_lb" "main" {
  name               = "app-lb"
  internal           = false
  load_balancer_type = "application"
  subnets            = [aws_subnet.public.id]
}

# Auto Scaling Group
resource "aws_autoscaling_group" "main" {
  name                = "app-asg"
  max_size            = 5
  min_size            = 2
  desired_capacity    = 2
  vpc_zone_identifier = [aws_subnet.public.id]

  launch_template {
    id      = aws_launch_template.main.id
    version = "$Latest"
  }
}

# RDS Database
resource "aws_db_instance" "main" {
  identifier        = "app-db"
  engine            = "postgres"
  instance_class    = "db.t3.micro"
  allocated_storage = 20
  username          = "admin"
  password          = var.db_password
}
```

## Serverless Architecture

### AWS Lambda Example

```python
import json

def lambda_handler(event, context):
    """
    AWS Lambda function handler
    """
    name = event.get('name', 'World')
    
    return {
        'statusCode': 200,
        'body': json.dumps({
            'message': f'Hello, {name}!'
        })
    }
```

### Serverless Framework

**serverless.yml**
```yaml
service: my-service

provider:
  name: aws
  runtime: python3.9
  region: us-east-1

functions:
  hello:
    handler: handler.lambda_handler
    events:
      - http:
          path: hello
          method: get
      - schedule: rate(10 minutes)

resources:
  Resources:
    MyBucket:
      Type: AWS::S3::Bucket
      Properties:
        BucketName: my-serverless-bucket
```

## Cloud Security Best Practices

### 1. Identity and Access Management
- Use MFA for all users
- Follow least privilege principle
- Rotate credentials regularly
- Use roles instead of keys when possible

### 2. Network Security
- Use VPC/VNet for isolation
- Implement security groups/NSGs
- Use private subnets for databases
- Enable VPN or Direct Connect

### 3. Data Protection
- Encrypt data at rest
- Encrypt data in transit (TLS)
- Regular backups
- Data lifecycle management

### 4. Monitoring and Logging
- Enable CloudTrail/Activity Logs
- Centralize logs
- Set up alerts
- Regular security audits

### 5. Compliance
- Understand compliance requirements
- Use compliance services
- Regular audits
- Documentation

## Cost Management

### 1. Monitoring
- Use cost management tools
- Set up billing alerts
- Tag resources
- Regular cost reviews

### 2. Optimization
- Right-size resources
- Use auto-scaling
- Reserved/committed use pricing
- Spot/preemptible instances
- Delete unused resources

### 3. Budgeting
- Set budgets
- Forecast spending
- Allocate costs to teams/projects
- Track against budget

## Interview Questions

1. What are the differences between IaaS, PaaS, and SaaS?
2. Explain the shared responsibility model in cloud computing
3. What is the difference between vertical and horizontal scaling?
4. How do you ensure high availability in the cloud?
5. What is the difference between S3 and EBS?
6. Explain VPC and its components
7. What is auto-scaling and how does it work?
8. How do you implement disaster recovery in the cloud?
9. What is serverless computing?
10. How do you optimize cloud costs?

## Practical Exercises

1. Deploy a web application on AWS/Azure/GCP
2. Set up auto-scaling for an application
3. Implement multi-region deployment
4. Create a serverless API
5. Set up VPC with public and private subnets
6. Implement backup and disaster recovery
7. Configure monitoring and alerting
8. Optimize costs for a cloud deployment

## Resources

- [AWS Documentation](https://docs.aws.amazon.com/)
- [Azure Documentation](https://docs.microsoft.com/en-us/azure/)
- [GCP Documentation](https://cloud.google.com/docs)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [Azure Architecture Center](https://docs.microsoft.com/en-us/azure/architecture/)
- [Google Cloud Architecture Framework](https://cloud.google.com/architecture/framework)
- [Cloud Computing Specializations (Coursera)](https://www.coursera.org/browse/information-technology/cloud-computing)
