# Cloud Platforms

## What is Cloud Computing?

Cloud computing is the delivery of computing services—including servers, storage, databases, networking, software, analytics, and intelligence—over the Internet ("the cloud") to offer faster innovation, flexible resources, and economies of scale.

## Cloud Service Models

### IaaS (Infrastructure as a Service)
Provides virtualized computing resources over the internet.

**Examples**: AWS EC2, Azure VMs, Google Compute Engine

**You Manage**: Applications, Data, Runtime, Middleware, OS

**Provider Manages**: Virtualization, Servers, Storage, Networking

### PaaS (Platform as a Service)
Provides a platform allowing customers to develop, run, and manage applications.

**Examples**: AWS Elastic Beanstalk, Azure App Service, Google App Engine, Heroku

**You Manage**: Applications, Data

**Provider Manages**: Runtime, Middleware, OS, Virtualization, Servers, Storage, Networking

### SaaS (Software as a Service)
Delivers software applications over the internet.

**Examples**: Google Workspace, Microsoft 365, Salesforce, Slack

**You Manage**: Data configuration

**Provider Manages**: Everything else

## Cloud Deployment Models

### Public Cloud
Services offered over the public internet and available to anyone.
- Examples: AWS, Azure, GCP
- Lower costs, no maintenance
- Less control, shared resources

### Private Cloud
Cloud infrastructure operated solely for a single organization.
- On-premises or hosted
- More control and security
- Higher costs, maintenance required

### Hybrid Cloud
Combination of public and private clouds.
- Flexibility
- Data and application portability
- Optimized costs

### Multi-Cloud
Using multiple cloud providers.
- Avoid vendor lock-in
- Best-of-breed services
- Complexity in management

## Major Cloud Providers

### 1. Amazon Web Services (AWS)

**Market Leader**: Largest cloud provider by market share

**Global Infrastructure**:
- Regions: 30+ worldwide
- Availability Zones: 90+
- Edge Locations: 400+

#### Core Services

**Compute**:
- **EC2**: Virtual servers
- **Lambda**: Serverless functions
- **ECS/EKS**: Container orchestration
- **Elastic Beanstalk**: PaaS for applications

**Storage**:
- **S3**: Object storage
- **EBS**: Block storage for EC2
- **EFS**: Managed file storage
- **Glacier**: Archive storage

**Database**:
- **RDS**: Managed relational databases (MySQL, PostgreSQL, etc.)
- **DynamoDB**: NoSQL database
- **Aurora**: High-performance MySQL/PostgreSQL
- **ElastiCache**: In-memory caching (Redis, Memcached)

**Networking**:
- **VPC**: Virtual Private Cloud
- **Route 53**: DNS service
- **CloudFront**: CDN
- **ELB**: Load balancing

**Security & Identity**:
- **IAM**: Identity and Access Management
- **KMS**: Key Management Service
- **Secrets Manager**: Secrets management
- **WAF**: Web Application Firewall

**Monitoring & Management**:
- **CloudWatch**: Monitoring and logging
- **CloudTrail**: API activity logging
- **Systems Manager**: Operations management

**DevOps Tools**:
- **CodePipeline**: CI/CD service
- **CodeBuild**: Build service
- **CodeDeploy**: Deployment service
- **CloudFormation**: Infrastructure as Code

#### AWS Example: Deploy Web Application

```bash
# Install AWS CLI
pip install awscli

# Configure credentials
aws configure

# Create S3 bucket for static website
aws s3 mb s3://my-website-bucket
aws s3 website s3://my-website-bucket --index-document index.html

# Upload files
aws s3 sync ./website s3://my-website-bucket --acl public-read

# Create EC2 instance
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t2.micro \
  --key-name MyKeyPair \
  --security-group-ids sg-123456 \
  --subnet-id subnet-123456 \
  --user-data file://setup-script.sh

# Create RDS database
aws rds create-db-instance \
  --db-instance-identifier mydb \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --master-username admin \
  --master-user-password MyPassword123 \
  --allocated-storage 20
```

#### AWS Lambda Example

```python
# lambda_function.py
import json

def lambda_handler(event, context):
    # Get query parameters
    name = event.get('queryStringParameters', {}).get('name', 'World')
    
    # Return response
    return {
        'statusCode': 200,
        'headers': {
            'Content-Type': 'application/json'
        },
        'body': json.dumps({
            'message': f'Hello, {name}!'
        })
    }
```

**Deploy Lambda**:
```bash
# Package function
zip function.zip lambda_function.py

# Create function
aws lambda create-function \
  --function-name HelloWorld \
  --runtime python3.9 \
  --role arn:aws:iam::123456789012:role/lambda-role \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip

# Invoke function
aws lambda invoke \
  --function-name HelloWorld \
  --payload '{"queryStringParameters":{"name":"DevOps"}}' \
  response.json
```

### 2. Microsoft Azure

**Enterprise Focus**: Strong integration with Microsoft products

**Global Infrastructure**:
- Regions: 60+
- Availability Zones: Multiple per region

#### Core Services

**Compute**:
- **Virtual Machines**: IaaS compute
- **Azure Functions**: Serverless
- **Azure Kubernetes Service (AKS)**: Managed Kubernetes
- **App Service**: PaaS web hosting

**Storage**:
- **Blob Storage**: Object storage
- **Disk Storage**: VM disks
- **File Storage**: Managed file shares
- **Archive Storage**: Cold storage

**Database**:
- **Azure SQL Database**: Managed SQL Server
- **Cosmos DB**: Multi-model NoSQL
- **Azure Database for PostgreSQL/MySQL**: Managed databases
- **Azure Cache for Redis**: In-memory cache

**Networking**:
- **Virtual Network**: Network isolation
- **Azure DNS**: Domain name system
- **Azure CDN**: Content delivery
- **Load Balancer**: Traffic distribution

**Identity & Security**:
- **Azure Active Directory**: Identity management
- **Key Vault**: Secrets management
- **Azure Security Center**: Security management

**Monitoring**:
- **Azure Monitor**: Monitoring service
- **Application Insights**: APM
- **Log Analytics**: Log management

**DevOps**:
- **Azure DevOps**: Complete DevOps solution
- **Azure Pipelines**: CI/CD
- **Azure Repos**: Git repositories

#### Azure CLI Examples

```bash
# Install Azure CLI
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Login
az login

# Create resource group
az group create --name MyResourceGroup --location eastus

# Create virtual machine
az vm create \
  --resource-group MyResourceGroup \
  --name MyVM \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys

# Create storage account
az storage account create \
  --name mystorageaccount \
  --resource-group MyResourceGroup \
  --location eastus \
  --sku Standard_LRS

# Create AKS cluster
az aks create \
  --resource-group MyResourceGroup \
  --name MyAKSCluster \
  --node-count 3 \
  --enable-addons monitoring \
  --generate-ssh-keys
```

#### Azure Functions Example

```csharp
// Function code (C#)
using System.IO;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;

public static class HelloWorld
{
    [FunctionName("HelloWorld")]
    public static async Task<IActionResult> Run(
        [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)] HttpRequest req,
        ILogger log)
    {
        string name = req.Query["name"];
        return new OkObjectResult($"Hello, {name ?? "World"}!");
    }
}
```

### 3. Google Cloud Platform (GCP)

**Innovation Leader**: Strong in data analytics, ML, and containers

**Global Infrastructure**:
- Regions: 35+
- Zones: 100+

#### Core Services

**Compute**:
- **Compute Engine**: Virtual machines
- **Cloud Functions**: Serverless
- **Google Kubernetes Engine (GKE)**: Managed Kubernetes
- **App Engine**: PaaS

**Storage**:
- **Cloud Storage**: Object storage
- **Persistent Disk**: Block storage
- **Filestore**: Managed file storage

**Database**:
- **Cloud SQL**: Managed MySQL/PostgreSQL/SQL Server
- **Cloud Spanner**: Globally distributed database
- **Firestore**: NoSQL document database
- **Bigtable**: Wide-column NoSQL
- **Memorystore**: Redis/Memcached

**Networking**:
- **VPC**: Virtual Private Cloud
- **Cloud DNS**: DNS service
- **Cloud CDN**: Content delivery
- **Cloud Load Balancing**: Load balancing

**Security**:
- **Cloud IAM**: Identity and Access Management
- **Cloud KMS**: Key management
- **Secret Manager**: Secrets management

**Monitoring**:
- **Cloud Monitoring**: Metrics and monitoring
- **Cloud Logging**: Log management
- **Cloud Trace**: Distributed tracing

**DevOps**:
- **Cloud Build**: CI/CD
- **Artifact Registry**: Package repository
- **Cloud Deploy**: Deployment pipelines

**Data & AI**:
- **BigQuery**: Data warehouse
- **Dataflow**: Stream/batch processing
- **Vertex AI**: ML platform
- **Cloud AI APIs**: Pre-trained ML models

#### GCloud CLI Examples

```bash
# Install gcloud CLI
curl https://sdk.cloud.google.com | bash

# Initialize and authenticate
gcloud init

# Create VM instance
gcloud compute instances create my-instance \
  --zone=us-central1-a \
  --machine-type=e2-medium \
  --image-family=ubuntu-2004-lts \
  --image-project=ubuntu-os-cloud

# Create GKE cluster
gcloud container clusters create my-cluster \
  --zone us-central1-a \
  --num-nodes 3

# Create Cloud Storage bucket
gsutil mb gs://my-unique-bucket-name

# Upload file to bucket
gsutil cp file.txt gs://my-unique-bucket-name/

# Deploy to App Engine
gcloud app deploy app.yaml
```

#### Cloud Functions Example

```javascript
// index.js (Node.js)
exports.helloWorld = (req, res) => {
  const name = req.query.name || req.body.name || 'World';
  res.status(200).send(`Hello, ${name}!`);
};
```

```bash
# Deploy function
gcloud functions deploy helloWorld \
  --runtime nodejs18 \
  --trigger-http \
  --allow-unauthenticated
```

## Cloud Architecture Patterns

### 1. Multi-Tier Architecture

```
┌─────────────┐
│  Users      │
└──────┬──────┘
       │
┌──────▼──────┐
│ Load        │
│ Balancer    │
└──────┬──────┘
       │
   ┌───┴───┐
   │       │
┌──▼──┐ ┌──▼──┐
│Web  │ │Web  │  Presentation Tier
│Server│ │Server│
└──┬──┘ └──┬──┘
   │       │
   └───┬───┘
       │
┌──────▼──────┐
│ Application │  Application Tier
│ Servers     │
└──────┬──────┘
       │
┌──────▼──────┐
│  Database   │  Data Tier
│  Cluster    │
└─────────────┘
```

### 2. Microservices Architecture

```
┌──────────┐
│   API    │
│ Gateway  │
└─────┬────┘
      │
  ┌───┴────┬────────┬────────┐
  │        │        │        │
┌─▼──┐  ┌─▼──┐  ┌─▼──┐  ┌─▼──┐
│User│  │Order│ │Pay-│  │Inv-│
│Svc │  │Svc  │ │ment│  │entory│
└────┘  └────┘  └────┘  └────┘
  │       │       │        │
  └───┬───┴───┬───┴────┬───┘
      │       │        │
   ┌──▼───┐ ┌▼────┐ ┌─▼────┐
   │User  │ │Order│ │Product│
   │DB    │ │DB   │ │DB    │
   └──────┘ └─────┘ └──────┘
```

### 3. Serverless Architecture

```
┌──────────┐
│   API    │
│ Gateway  │
└─────┬────┘
      │
  ┌───┴────┬────────┬────────┐
  │        │        │        │
┌─▼──────┐ ┌▼──────┐ ┌▼──────┐
│Lambda/ │ │Lambda/│ │Lambda/│
│Function│ │Function│ │Function│
└─────┬──┘ └───┬───┘ └───┬───┘
      │        │         │
   ┌──▼────┐ ┌▼─────┐ ┌─▼────┐
   │DynamoDB│ │S3    │ │SQS   │
   └────────┘ └──────┘ └──────┘
```

### 4. Event-Driven Architecture

```
┌──────────┐
│ Event    │
│ Source   │
└─────┬────┘
      │
┌─────▼────────┐
│ Event Bus    │
│ (EventBridge)│
└─────┬────────┘
      │
  ┌───┴───┬────────┐
  │       │        │
┌─▼───┐ ┌▼────┐ ┌─▼────┐
│Func │ │Func │ │Queue │
│tion │ │tion │ │      │
└─────┘ └─────┘ └──────┘
```

## Cloud Best Practices

### 1. Security

#### Identity and Access Management
- Principle of least privilege
- Use roles, not root/admin accounts
- Enable MFA (Multi-Factor Authentication)
- Regular access reviews

#### Network Security
- Use VPCs/VNets for isolation
- Security groups and network ACLs
- Private subnets for databases
- VPN or Direct Connect for hybrid

#### Data Security
- Encrypt data at rest
- Encrypt data in transit (TLS/SSL)
- Use managed encryption services
- Regular backups

#### Compliance
- Use compliance frameworks (HIPAA, PCI-DSS, etc.)
- Enable audit logging
- Regular security assessments
- Compliance monitoring tools

### 2. Cost Optimization

#### Right-Sizing
- Match instance types to workload
- Use reserved instances for steady workloads
- Spot instances for fault-tolerant workloads

#### Auto-Scaling
- Scale based on demand
- Schedule scaling for predictable patterns
- Set appropriate thresholds

#### Storage Optimization
- Use appropriate storage classes
- Lifecycle policies for old data
- Delete unused resources

#### Monitoring
- Use cost management tools
- Set budgets and alerts
- Tag resources for cost allocation

### 3. Reliability

#### High Availability
- Multi-AZ deployments
- Load balancing
- Auto-scaling
- Health checks

#### Disaster Recovery
- Regular backups
- Multi-region replication
- Tested recovery procedures
- RTO and RPO defined

#### Resilience
- Retry logic with exponential backoff
- Circuit breakers
- Graceful degradation
- Chaos engineering

### 4. Performance

#### Caching
- Use CDN for static content
- In-memory caching (Redis, Memcached)
- Database query caching
- API response caching

#### Database Optimization
- Read replicas
- Connection pooling
- Query optimization
- Appropriate indexing

#### Content Delivery
- Use CDN
- Compress assets
- Lazy loading
- Edge computing

### 5. Operational Excellence

#### Automation
- Infrastructure as Code
- CI/CD pipelines
- Automated testing
- Auto-remediation

#### Monitoring and Logging
- Centralized logging
- Metrics and dashboards
- Alerting
- Distributed tracing

#### Documentation
- Architecture diagrams
- Runbooks
- Change logs
- Knowledge base

## Cloud Migration Strategies (6 R's)

### 1. Rehost ("Lift and Shift")
Move applications without changes
- Fastest migration
- Minimal risk
- Limited cloud benefits

### 2. Replatform ("Lift, Tinker, and Shift")
Minor optimizations without changing core architecture
- Some cloud benefits
- Moderate effort
- Examples: Managed databases, containers

### 3. Repurchase ("Drop and Shop")
Move to SaaS
- Quick migration
- Reduced maintenance
- May require process changes

### 4. Refactor/Re-architect
Redesign using cloud-native features
- Maximum cloud benefits
- Highest effort
- Best long-term solution

### 5. Retire
Decommission applications
- Reduce costs
- Simplify portfolio

### 6. Retain
Keep on-premises
- Compliance requirements
- Not ready to migrate
- Review periodically

## Comparison: AWS vs Azure vs GCP

| Aspect | AWS | Azure | GCP |
|--------|-----|-------|-----|
| Market Share | ~32% | ~23% | ~10% |
| Best For | Everything | Microsoft stack | Data & ML |
| Regions | 30+ | 60+ | 35+ |
| Compute | EC2, Lambda | VMs, Functions | Compute Engine, Functions |
| Containers | ECS, EKS | AKS | GKE |
| Pricing | Complex | Complex | Simpler |
| Free Tier | 12 months | 12 months | Always free tier |
| Certifications | Many | Many | Growing |

## Cloud Certifications

### AWS
- **Foundational**: Cloud Practitioner
- **Associate**: Solutions Architect, Developer, SysOps Administrator
- **Professional**: Solutions Architect, DevOps Engineer
- **Specialty**: Security, Machine Learning, etc.

### Azure
- **Fundamentals**: AZ-900
- **Associate**: Administrator (AZ-104), Developer (AZ-204)
- **Expert**: Solutions Architect (AZ-305), DevOps Engineer (AZ-400)

### GCP
- **Associate**: Cloud Engineer
- **Professional**: Cloud Architect, Data Engineer, Cloud Developer

## Resources

- AWS Documentation: https://docs.aws.amazon.com/
- Azure Documentation: https://docs.microsoft.com/azure/
- GCP Documentation: https://cloud.google.com/docs
- AWS Well-Architected Framework: https://aws.amazon.com/architecture/well-architected/
- Azure Architecture Center: https://docs.microsoft.com/azure/architecture/
- GCP Architecture Framework: https://cloud.google.com/architecture/framework

## Conclusion

Cloud platforms are essential for modern DevOps practices. Understanding the core services, architecture patterns, and best practices across major cloud providers enables you to build scalable, reliable, and cost-effective solutions.

---

**Congratulations!** You've completed the DevOps study material from basics. Continue practicing with hands-on projects to reinforce your learning.
