# CI/CD Pipelines

## What is CI/CD?

**Continuous Integration (CI)** and **Continuous Delivery/Deployment (CD)** are practices that enable development teams to deliver code changes more frequently and reliably.

### Continuous Integration (CI)
- Developers integrate code into a shared repository frequently
- Each integration is verified by automated build and tests
- Detects integration bugs early

### Continuous Delivery (CD)
- Extension of CI
- Ensures code is always in a deployable state
- Deployment is manual but easy

### Continuous Deployment
- Goes one step further than Continuous Delivery
- Every change that passes tests is deployed automatically
- No human intervention

## Benefits of CI/CD

- **Faster Time to Market**: Rapid delivery of features
- **Reduced Risk**: Smaller, incremental changes
- **Better Quality**: Automated testing catches bugs early
- **Lower Costs**: Less manual intervention
- **Improved Team Morale**: Less firefighting, more innovation

## CI/CD Pipeline Stages

1. **Source**: Code is committed to version control
2. **Build**: Code is compiled and built
3. **Test**: Automated tests are run
4. **Deploy to Staging**: Deploy to testing environment
5. **Integration Tests**: Run tests in staging
6. **Deploy to Production**: Release to production
7. **Monitor**: Track application performance

## Popular CI/CD Tools

### 1. Jenkins
- Open source automation server
- Highly extensible with plugins
- Self-hosted

### 2. GitHub Actions
- Native GitHub integration
- YAML-based workflows
- Free for public repositories

### 3. GitLab CI/CD
- Built into GitLab
- Auto DevOps capabilities
- Kubernetes integration

### 4. CircleCI
- Cloud-based or self-hosted
- Docker-first approach
- Fast builds with caching

### 5. Travis CI
- Cloud-based
- Easy integration with GitHub
- Free for open source

### 6. Azure DevOps
- Microsoft's DevOps platform
- Comprehensive toolset
- Azure integration

## Jenkins

### Installation

#### Docker Installation
```bash
docker run -d -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  --name jenkins jenkins/jenkins:lts
```

#### Ubuntu Installation
```bash
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt-get update
sudo apt-get install jenkins
```

### Basic Jenkins Pipeline (Jenkinsfile)

```groovy
pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/user/repo.git'
            }
        }
        
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
                sh './deploy.sh'
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

### Advanced Jenkins Pipeline

```groovy
pipeline {
    agent {
        docker {
            image 'node:16-alpine'
        }
    }
    
    environment {
        NODE_ENV = 'production'
        API_KEY = credentials('api-key-id')
    }
    
    stages {
        stage('Setup') {
            steps {
                sh 'npm ci'
            }
        }
        
        stage('Lint') {
            steps {
                sh 'npm run lint'
            }
        }
        
        stage('Unit Tests') {
            steps {
                sh 'npm run test:unit'
            }
        }
        
        stage('Integration Tests') {
            steps {
                sh 'npm run test:integration'
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
        
        stage('Security Scan') {
            steps {
                sh 'npm audit'
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'develop'
            }
            steps {
                sh './deploy-staging.sh'
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to production?'
                sh './deploy-production.sh'
            }
        }
    }
    
    post {
        always {
            junit 'reports/**/*.xml'
            archiveArtifacts artifacts: 'dist/**/*', fingerprint: true
        }
        failure {
            mail to: 'team@example.com',
                 subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
                 body: "Pipeline failed. Check ${env.BUILD_URL}"
        }
    }
}
```

## GitHub Actions

### Basic Workflow (.github/workflows/ci.yml)

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '16'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run tests
      run: npm test
    
    - name: Build
      run: npm run build
    
    - name: Upload artifacts
      uses: actions/upload-artifact@v3
      with:
        name: build-output
        path: dist/
```

### Advanced GitHub Actions Workflow

```yaml
name: Complete CI/CD

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  NODE_VERSION: '16'

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ env.NODE_VERSION }}
    - run: npm ci
    - run: npm run lint

  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [14, 16, 18]
    steps:
    - uses: actions/checkout@v3
    - name: Setup Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
    - run: npm ci
    - run: npm test
    - name: Upload coverage
      uses: codecov/codecov-action@v3

  security:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - run: npm audit

  build:
    needs: [lint, test, security]
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ env.NODE_VERSION }}
    - run: npm ci
    - run: npm run build
    - uses: actions/upload-artifact@v3
      with:
        name: dist
        path: dist/

  deploy-staging:
    needs: build
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    environment: staging
    steps:
    - uses: actions/checkout@v3
    - uses: actions/download-artifact@v3
      with:
        name: dist
    - name: Deploy to staging
      run: |
        echo "Deploying to staging..."
        # Add deployment commands here

  deploy-production:
    needs: build
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    steps:
    - uses: actions/checkout@v3
    - uses: actions/download-artifact@v3
      with:
        name: dist
    - name: Deploy to production
      run: |
        echo "Deploying to production..."
        # Add deployment commands here
```

## GitLab CI/CD

### .gitlab-ci.yml Example

```yaml
stages:
  - build
  - test
  - deploy

variables:
  NODE_VERSION: "16"

build:
  stage: build
  image: node:${NODE_VERSION}
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week

test:
  stage: test
  image: node:${NODE_VERSION}
  script:
    - npm ci
    - npm test
  coverage: '/Coverage: \d+\.\d+/'

lint:
  stage: test
  image: node:${NODE_VERSION}
  script:
    - npm ci
    - npm run lint

deploy-staging:
  stage: deploy
  script:
    - echo "Deploying to staging..."
    - ./deploy-staging.sh
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - develop

deploy-production:
  stage: deploy
  script:
    - echo "Deploying to production..."
    - ./deploy-production.sh
  environment:
    name: production
    url: https://example.com
  only:
    - main
  when: manual
```

## CircleCI

### .circleci/config.yml Example

```yaml
version: 2.1

orbs:
  node: circleci/node@5.0

jobs:
  build-and-test:
    docker:
      - image: cimg/node:16.0
    steps:
      - checkout
      - node/install-packages:
          pkg-manager: npm
      - run:
          name: Run tests
          command: npm test
      - run:
          name: Build
          command: npm run build
      - persist_to_workspace:
          root: .
          paths:
            - dist

  deploy:
    docker:
      - image: cimg/node:16.0
    steps:
      - attach_workspace:
          at: .
      - run:
          name: Deploy
          command: ./deploy.sh

workflows:
  build-test-deploy:
    jobs:
      - build-and-test
      - deploy:
          requires:
            - build-and-test
          filters:
            branches:
              only: main
```

## Best Practices

### 1. Keep Pipelines Fast
- Use caching
- Run tests in parallel
- Only run necessary steps

### 2. Fail Fast
- Run quick checks first
- Stop pipeline on first failure
- Clear error messages

### 3. Security
- Don't hardcode secrets
- Use secret management
- Scan for vulnerabilities

### 4. Artifact Management
- Store build artifacts
- Version artifacts properly
- Clean up old artifacts

### 5. Environment Parity
- Keep environments similar
- Use infrastructure as code
- Test in production-like environment

### 6. Monitoring and Alerts
- Monitor pipeline performance
- Alert on failures
- Track deployment frequency

### 7. Documentation
- Document pipeline stages
- Keep Jenkinsfile/workflow files in VCS
- Comment complex steps

## Testing in CI/CD

### Types of Tests

1. **Unit Tests**: Test individual components
2. **Integration Tests**: Test component interactions
3. **End-to-End Tests**: Test complete workflows
4. **Performance Tests**: Test under load
5. **Security Tests**: Scan for vulnerabilities

### Testing Strategy

```yaml
stages:
  - unit-tests        # Fast, run always
  - integration-tests # Medium speed
  - e2e-tests        # Slow, run on main branch
  - performance      # Very slow, scheduled
```

## Deployment Strategies

### 1. Blue-Green Deployment
- Two identical production environments
- Switch traffic between them
- Quick rollback

### 2. Canary Deployment
- Gradually roll out to subset of users
- Monitor for issues
- Full rollout if successful

### 3. Rolling Deployment
- Update instances gradually
- Maintain availability
- Slower than blue-green

### 4. Feature Flags
- Deploy code without enabling feature
- Enable features independently
- A/B testing capability

## Common Interview Questions

1. What is the difference between Continuous Delivery and Continuous Deployment?
2. Explain the stages of a typical CI/CD pipeline
3. How do you handle secrets in CI/CD pipelines?
4. What is the difference between declarative and scripted Jenkins pipelines?
5. How do you optimize slow CI/CD pipelines?
6. Explain blue-green deployment strategy
7. What is a canary deployment?
8. How do you handle database migrations in CI/CD?
9. What is the purpose of artifact repositories?
10. How do you implement rollback in CI/CD?

## Practical Exercises

1. Set up a basic Jenkins pipeline
2. Create a GitHub Actions workflow for a Node.js project
3. Implement parallel testing in your pipeline
4. Add security scanning to your pipeline
5. Set up multi-stage deployment (dev, staging, prod)
6. Implement automatic rollback on failure
7. Create a pipeline with manual approval for production
8. Set up notifications for pipeline failures

## Resources

- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [GitLab CI/CD Documentation](https://docs.gitlab.com/ee/ci/)
- [CircleCI Documentation](https://circleci.com/docs/)
- [The Phoenix Project (Book)](https://www.amazon.com/Phoenix-Project-DevOps-Helping-Business/dp/0988262592)
- [Continuous Delivery (Book)](https://www.amazon.com/Continuous-Delivery-Deployment-Automation-Addison-Wesley/dp/0321601912)
