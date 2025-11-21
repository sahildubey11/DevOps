# Continuous Integration and Continuous Deployment (CI/CD)

## What is CI/CD?

CI/CD is a method to frequently deliver apps to customers by introducing automation into the stages of app development. The main concepts are Continuous Integration, Continuous Delivery, and Continuous Deployment.

## Continuous Integration (CI)

### Definition
Continuous Integration is a development practice where developers integrate code into a shared repository frequently, preferably several times a day. Each integration is verified by an automated build and automated tests.

### Benefits
- **Early Bug Detection**: Issues found quickly when code is integrated
- **Reduced Integration Problems**: Small, frequent integrations easier to debug
- **Better Code Quality**: Automated testing ensures standards
- **Faster Development**: Quick feedback loops
- **Improved Collaboration**: Shared codebase keeps everyone aligned

### Core Practices
1. Maintain a single source repository
2. Automate the build
3. Make builds self-testing
4. Everyone commits frequently
5. Every commit builds on integration machine
6. Keep the build fast
7. Test in clone of production environment
8. Make it easy to get latest deliverables
9. Everyone can see results
10. Automate deployment

## Continuous Delivery (CD)

### Definition
Continuous Delivery ensures that code is always in a deployable state. Every change goes through the automated pipeline and can be released to production at any time with a manual approval.

### Key Characteristics
- Automated deployment to staging/test environments
- Manual approval for production deployment
- Code always ready to deploy
- Low-risk releases

## Continuous Deployment

### Definition
Continuous Deployment goes one step further than Continuous Delivery. Every change that passes all stages of the production pipeline is released to customers automatically, with no manual intervention.

### Differences from Continuous Delivery
- **Continuous Delivery**: Manual approval before production
- **Continuous Deployment**: Fully automated to production

## CI/CD Pipeline

A typical CI/CD pipeline includes:

```
Source → Build → Test → Deploy to Staging → Deploy to Production
```

### Detailed Pipeline Stages

#### 1. Source Stage
- Developer commits code to version control
- Triggers the pipeline
- Tools: Git, GitHub, GitLab, Bitbucket

#### 2. Build Stage
- Compile source code
- Create build artifacts
- Dependency management
- Tools: Maven, Gradle, npm, pip

#### 3. Test Stage
- **Unit Tests**: Test individual components
- **Integration Tests**: Test component interactions
- **Code Quality**: Linting, code coverage
- **Security Scans**: Vulnerability detection
- Tools: JUnit, Jest, pytest, SonarQube

#### 4. Deploy to Staging
- Deploy to staging environment
- Run smoke tests
- Performance testing
- Tools: Ansible, Terraform, Kubernetes

#### 5. Deploy to Production
- Blue-green deployment
- Canary releases
- Rolling updates
- Tools: Kubernetes, Docker, Cloud platforms

#### 6. Monitor
- Application monitoring
- Log aggregation
- Alert on failures
- Tools: Prometheus, Grafana, ELK

## CI/CD Tools

### Jenkins

**Type**: Open-source automation server

**Features**:
- Highly extensible with plugins
- Distributed builds
- Pipeline as Code (Jenkinsfile)
- Master-slave architecture

**Example Jenkinsfile**:
```groovy
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
        
        stage('Deploy') {
            steps {
                sh 'kubectl apply -f deployment.yaml'
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
```

### GitHub Actions

**Type**: Cloud-based CI/CD platform integrated with GitHub

**Features**:
- YAML-based workflows
- Marketplace for actions
- Matrix builds
- Secrets management

**Example Workflow**:
```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run tests
      run: npm test
    
    - name: Build
      run: npm run build
    
    - name: Deploy to production
      if: github.ref == 'refs/heads/main'
      run: |
        echo "Deploying to production..."
```

### GitLab CI/CD

**Type**: Built-in CI/CD for GitLab

**Features**:
- Integrated with GitLab
- Auto DevOps
- Kubernetes integration
- Container Registry

**Example .gitlab-ci.yml**:
```yaml
stages:
  - build
  - test
  - deploy

variables:
  NODE_VERSION: "18"

build:
  stage: build
  image: node:${NODE_VERSION}
  script:
    - npm install
    - npm run build
  artifacts:
    paths:
      - dist/

test:
  stage: test
  image: node:${NODE_VERSION}
  script:
    - npm install
    - npm test
  coverage: '/Lines\s*:\s*(\d+\.\d+)%/'

deploy:
  stage: deploy
  script:
    - kubectl apply -f k8s/
  only:
    - main
```

### CircleCI

**Type**: Cloud-based CI/CD platform

**Features**:
- Fast builds with caching
- Docker support
- Workflows and orchestration
- SSH debugging

**Example config.yml**:
```yaml
version: 2.1

jobs:
  build:
    docker:
      - image: cimg/node:18.0
    steps:
      - checkout
      - restore_cache:
          keys:
            - v1-dependencies-{{ checksum "package.json" }}
      - run: npm install
      - save_cache:
          paths:
            - node_modules
          key: v1-dependencies-{{ checksum "package.json" }}
      - run: npm test
      - run: npm run build

workflows:
  version: 2
  build-and-deploy:
    jobs:
      - build
```

### Travis CI

**Type**: Cloud-based CI/CD service

**Features**:
- GitHub integration
- Multiple language support
- Build matrix
- Easy configuration

**Example .travis.yml**:
```yaml
language: node_js
node_js:
  - "18"

script:
  - npm test
  - npm run build

deploy:
  provider: heroku
  api_key: $HEROKU_API_KEY
  app: my-app-name
  on:
    branch: main
```

## CI/CD Best Practices

### 1. Version Control Everything
- Code
- Tests
- Configuration
- Infrastructure definitions
- Documentation

### 2. Build Once, Deploy Many
- Build artifacts once
- Promote same artifact through environments
- Use environment-specific configuration

### 3. Fast Feedback
- Keep builds under 10 minutes
- Run fast tests first
- Parallelize tests
- Use caching strategically

### 4. Test Pyramid

```
        /\
       /  \  E2E Tests (Few)
      /____\
     /      \
    / Integration\ Tests (Some)
   /____________\
  /              \
 /  Unit Tests    \ (Many)
/__________________\
```

- Many fast unit tests
- Some integration tests
- Few end-to-end tests

### 5. Security in Pipeline

- **SAST**: Static Application Security Testing
- **DAST**: Dynamic Application Security Testing
- **Dependency Scanning**: Check for vulnerable dependencies
- **Container Scanning**: Scan Docker images
- **Secret Detection**: Prevent committing secrets

### 6. Monitoring and Alerts
- Monitor pipeline health
- Alert on failures
- Track metrics (build time, success rate)
- Dashboard for visibility

### 7. Deployment Strategies

#### Blue-Green Deployment
```
Blue (v1) ←──── Traffic
Green (v2) ←─── Deploy new version
            Switch traffic →
Green (v2) ←──── Traffic
Blue (v1)  ←─── Keep as rollback
```

#### Canary Deployment
```
v1: 90% traffic
v2: 10% traffic (canary)
    ↓ Monitor metrics
    ↓ If successful
v1: 50% traffic
v2: 50% traffic
    ↓ Continue
v2: 100% traffic
```

#### Rolling Deployment
```
Instances: [v1] [v1] [v1] [v1]
           [v2] [v1] [v1] [v1]
           [v2] [v2] [v1] [v1]
           [v2] [v2] [v2] [v1]
           [v2] [v2] [v2] [v2]
```

### 8. Feature Flags
- Deploy code without exposing features
- Gradual rollout
- A/B testing
- Quick rollback without redeployment

## Pipeline as Code

Define CI/CD pipelines in code for:
- Version control
- Code review
- Reproducibility
- Documentation

## Artifacts and Package Management

### Artifact Repositories
- **JFrog Artifactory**: Universal artifact repository
- **Nexus Repository**: Maven, Docker, npm packages
- **Docker Registry**: Container images
- **npm Registry**: JavaScript packages
- **PyPI**: Python packages

### Best Practices
- Version all artifacts
- Immutable artifacts
- Secure artifact storage
- Clean up old artifacts

## Testing in CI/CD

### Types of Tests

#### Unit Tests
```javascript
describe('Calculator', () => {
  test('adds two numbers', () => {
    expect(add(2, 3)).toBe(5);
  });
});
```

#### Integration Tests
```javascript
describe('API Integration', () => {
  test('creates user', async () => {
    const response = await request(app)
      .post('/users')
      .send({ name: 'John' });
    expect(response.status).toBe(201);
  });
});
```

#### End-to-End Tests
```javascript
describe('User Flow', () => {
  test('user can login and view dashboard', async () => {
    await page.goto('http://localhost:3000');
    await page.click('#login-button');
    await page.type('#username', 'testuser');
    await page.type('#password', 'password');
    await page.click('#submit');
    await expect(page).toHaveText('Dashboard');
  });
});
```

## Environment Management

### Environments
1. **Development**: Developer workstations
2. **Integration**: Integration testing
3. **Staging**: Pre-production, production-like
4. **Production**: Live environment

### Configuration Management
- Environment variables
- Configuration files
- Secret management (HashiCorp Vault, AWS Secrets Manager)
- Feature flags

## Metrics and KPIs

### Key Metrics
- **Build Success Rate**: Percentage of successful builds
- **Build Duration**: Time to complete build
- **Deployment Frequency**: How often you deploy
- **Lead Time**: Code commit to production
- **Mean Time to Recovery (MTTR)**: Time to recover from failure
- **Change Failure Rate**: Percentage of changes causing issues

### DORA Metrics
1. **Deployment Frequency**
2. **Lead Time for Changes**
3. **Time to Restore Service**
4. **Change Failure Rate**

## Common Challenges and Solutions

### Challenge: Slow Builds
**Solutions**:
- Parallelize tests
- Use caching
- Optimize dependencies
- Use incremental builds

### Challenge: Flaky Tests
**Solutions**:
- Identify and fix root cause
- Retry mechanisms for genuinely flaky tests
- Better test isolation
- Use test containerization

### Challenge: Complex Deployments
**Solutions**:
- Infrastructure as Code
- Deployment automation
- Containerization
- Feature flags

### Challenge: Security Vulnerabilities
**Solutions**:
- Automated security scanning
- Dependency updates
- Container scanning
- Secret management

## CI/CD Maturity Model

### Level 1: Manual
- Manual builds and deployments
- No automation

### Level 2: Automated Build
- Automated compilation
- Basic testing

### Level 3: Continuous Integration
- Automated testing
- Regular integration

### Level 4: Continuous Delivery
- Automated deployment to staging
- Manual production deployment

### Level 5: Continuous Deployment
- Fully automated pipeline
- Automatic production deployment

## Resources

- Jenkins Documentation: https://www.jenkins.io/doc/
- GitHub Actions: https://docs.github.com/en/actions
- GitLab CI/CD: https://docs.gitlab.com/ee/ci/
- The DevOps Handbook: https://itrevolution.com/book/the-devops-handbook/
- Continuous Delivery Book: https://continuousdelivery.com/

## Next Steps

Continue to [Configuration Management](../04-Configuration-Management/) to learn about tools like Ansible, Puppet, and Chef.
