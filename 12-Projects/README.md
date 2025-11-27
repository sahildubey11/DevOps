# Practical DevOps Projects

This section contains hands-on projects to practice DevOps skills. Each project builds on concepts from previous sections.

## Project 1: CI/CD Pipeline for Web Application

### Objective
Build a complete CI/CD pipeline for a Node.js web application using GitHub Actions.

### Requirements
- Automated testing on every commit
- Build Docker image
- Push to container registry
- Deploy to production on main branch
- Rollback capability

### Implementation Steps

1. **Setup Application**
```javascript
// Simple Express app (app.js)
const express = require('express');
const app = express();
const port = process.env.PORT || 3000;

app.get('/', (req, res) => {
  res.json({ message: 'Hello DevOps!' });
});

app.get('/health', (req, res) => {
  res.json({ status: 'healthy' });
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});

module.exports = app;
```

2. **Dockerfile**
```dockerfile
FROM node:16-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --production

COPY . .

USER node

EXPOSE 3000

CMD ["node", "app.js"]
```

3. **GitHub Actions Workflow**
```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '16'
      - run: npm ci
      - run: npm test
      - run: npm run lint

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      - name: Push image
        run: |
          docker tag myapp:${{ github.sha }} ${{ secrets.DOCKER_USERNAME }}/myapp:${{ github.sha }}
          docker push ${{ secrets.DOCKER_USERNAME }}/myapp:${{ github.sha }}

  deploy:
    needs: build
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to production
        run: |
          # Add deployment commands here
          echo "Deploying version ${{ github.sha }}"
```

### Learning Outcomes
- CI/CD pipeline configuration
- Automated testing
- Docker containerization
- Container registry management

---

## Project 2: Infrastructure as Code with Terraform

### Objective
Deploy a complete web application infrastructure on AWS using Terraform.

### Architecture
- VPC with public and private subnets
- Application Load Balancer
- Auto Scaling Group
- RDS Database
- S3 bucket for static assets

### Implementation

**main.tf**
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }
  
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "webapp/terraform.tfstate"
    region = "us-east-1"
  }
}

provider "aws" {
  region = var.aws_region
}

# VPC
module "vpc" {
  source = "./modules/vpc"
  
  vpc_cidr = var.vpc_cidr
  project_name = var.project_name
  environment = var.environment
}

# Application Load Balancer
module "alb" {
  source = "./modules/alb"
  
  vpc_id = module.vpc.vpc_id
  public_subnet_ids = module.vpc.public_subnet_ids
  project_name = var.project_name
}

# Auto Scaling Group
module "asg" {
  source = "./modules/asg"
  
  vpc_id = module.vpc.vpc_id
  private_subnet_ids = module.vpc.private_subnet_ids
  target_group_arn = module.alb.target_group_arn
  instance_type = var.instance_type
  min_size = var.min_size
  max_size = var.max_size
}

# RDS Database
module "rds" {
  source = "./modules/rds"
  
  vpc_id = module.vpc.vpc_id
  private_subnet_ids = module.vpc.private_subnet_ids
  db_name = var.db_name
  db_username = var.db_username
  db_password = var.db_password
}
```

### Learning Outcomes
- Infrastructure as Code
- AWS services
- Terraform modules
- State management

---

## Project 3: Kubernetes Microservices Deployment

### Objective
Deploy a microservices application on Kubernetes with monitoring and logging.

### Components
- Frontend service (React)
- Backend API (Node.js)
- Database (PostgreSQL)
- Redis cache
- Prometheus monitoring
- Grafana dashboards
- ELK stack for logging

### Implementation

**namespace.yaml**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: myapp
```

**frontend-deployment.yaml**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: myapp/frontend:1.0
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: myapp
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 80
```

**backend-deployment.yaml**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: myapp/backend:1.0
        ports:
        - containerPort: 3000
        env:
        - name: DB_HOST
          value: postgres
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
        - name: REDIS_HOST
          value: redis
---
apiVersion: v1
kind: Service
metadata:
  name: backend
  namespace: myapp
spec:
  selector:
    app: backend
  ports:
  - port: 3000
    targetPort: 3000
```

**Monitoring with Prometheus**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: monitoring
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
    
    scrape_configs:
    - job_name: 'kubernetes-pods'
      kubernetes_sd_configs:
      - role: pod
      relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
```

### Learning Outcomes
- Kubernetes deployments
- Microservices architecture
- Service mesh
- Monitoring and logging

---

## Project 4: Automated Infrastructure Monitoring

### Objective
Set up comprehensive monitoring and alerting for infrastructure and applications.

### Components
- Prometheus for metrics collection
- Grafana for visualization
- AlertManager for notifications
- Node Exporter for system metrics
- Application metrics

### Implementation

**docker-compose.yml**
```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./alerts.yml:/etc/prometheus/alerts.yml
      - prometheus-data:/prometheus
    ports:
      - "9090:9090"
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'

  grafana:
    image: grafana/grafana:latest
    volumes:
      - grafana-data:/var/lib/grafana
      - ./grafana/dashboards:/etc/grafana/provisioning/dashboards
      - ./grafana/datasources:/etc/grafana/provisioning/datasources
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin

  alertmanager:
    image: prom/alertmanager:latest
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
    ports:
      - "9093:9093"

  node-exporter:
    image: prom/node-exporter:latest
    ports:
      - "9100:9100"

volumes:
  prometheus-data:
  grafana-data:
```

### Learning Outcomes
- Monitoring setup
- Alert configuration
- Dashboard creation
- Metrics collection

---

## Project 5: GitOps with ArgoCD

### Objective
Implement GitOps workflow for Kubernetes deployments using ArgoCD.

### Implementation

1. **Install ArgoCD**
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

2. **Application Definition**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/user/myapp-config
    targetRevision: HEAD
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: myapp
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### Learning Outcomes
- GitOps principles
- ArgoCD configuration
- Automated deployments
- Declarative configuration

---

## Project 6: Complete DevOps Pipeline

### Objective
Build an end-to-end DevOps pipeline incorporating all concepts.

### Pipeline Stages
1. Source code management (Git)
2. Continuous integration (GitHub Actions)
3. Security scanning (Trivy, Snyk)
4. Container builds (Docker)
5. Infrastructure provisioning (Terraform)
6. Configuration management (Ansible)
7. Container orchestration (Kubernetes)
8. Monitoring (Prometheus/Grafana)
9. Logging (ELK Stack)
10. GitOps (ArgoCD)

### Architecture
```
[GitHub] -> [GitHub Actions]
    |
    v
[Build & Test] -> [Security Scan]
    |
    v
[Docker Build] -> [Container Registry]
    |
    v
[Terraform] -> [AWS Infrastructure]
    |
    v
[Ansible] -> [Configuration]
    |
    v
[Kubernetes] -> [Production]
    |
    v
[Monitoring & Logging]
```

---

## Additional Project Ideas

1. **Multi-Cloud Deployment**: Deploy same application to AWS, Azure, and GCP
2. **Disaster Recovery Setup**: Implement backup and recovery procedures
3. **Blue-Green Deployment**: Zero-downtime deployments
4. **Service Mesh**: Implement Istio for microservices
5. **Chaos Engineering**: Use Chaos Monkey to test resilience
6. **Cost Optimization**: Implement cloud cost monitoring and optimization
7. **Compliance Automation**: Automate security and compliance checks
8. **Developer Portal**: Build internal developer platform

## Best Practices for Projects

1. **Version Control**: Track all code and configurations
2. **Documentation**: Document architecture and processes
3. **Security**: Implement security best practices
4. **Monitoring**: Set up comprehensive monitoring
5. **Testing**: Include automated testing
6. **Disaster Recovery**: Plan for failures
7. **Cost Management**: Monitor and optimize costs
8. **Collaboration**: Use PR reviews and documentation

## Resources

- [DevOps Project Examples](https://github.com/topics/devops-project)
- [Kubernetes Examples](https://github.com/kubernetes/examples)
- [Terraform Examples](https://github.com/hashicorp/terraform-guides)
- [Docker Examples](https://docs.docker.com/samples/)
- [GitHub Actions Examples](https://github.com/actions/starter-workflows)
