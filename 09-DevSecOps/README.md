# DevSecOps - Security in DevOps

## What is DevSecOps?

DevSecOps integrates security practices into the DevOps process, making security a shared responsibility throughout the entire software development lifecycle.

## Why DevSecOps?

- **Early Detection**: Find vulnerabilities early in development
- **Continuous Security**: Security at every stage
- **Automation**: Automated security testing
- **Compliance**: Meet regulatory requirements
- **Reduced Risk**: Less chance of security breaches
- **Faster Remediation**: Quick response to security issues

## Security in the CI/CD Pipeline

### Pipeline Security Stages

1. **Code Security**: Static analysis, dependency scanning
2. **Build Security**: Container scanning, secrets detection
3. **Test Security**: Dynamic analysis, penetration testing
4. **Deploy Security**: Configuration validation, compliance checks
5. **Runtime Security**: Monitoring, intrusion detection

## Security Tools

### 1. Static Application Security Testing (SAST)

**SonarQube**
```yaml
# GitLab CI Example
sonarqube-check:
  stage: test
  image: 
    name: sonarsource/sonar-scanner-cli:latest
  variables:
    SONAR_HOST_URL: "https://sonarqube.example.com"
    SONAR_TOKEN: $SONAR_TOKEN
  script:
    - sonar-scanner
      -Dsonar.projectKey=my-project
      -Dsonar.sources=src
      -Dsonar.host.url=$SONAR_HOST_URL
      -Dsonar.login=$SONAR_TOKEN
```

**ESLint for JavaScript**
```json
{
  "extends": ["eslint:recommended", "plugin:security/recommended"],
  "plugins": ["security"],
  "rules": {
    "security/detect-object-injection": "error",
    "security/detect-non-literal-regexp": "error",
    "security/detect-unsafe-regex": "error"
  }
}
```

### 2. Dependency Scanning

**npm audit**
```bash
# Scan for vulnerabilities
npm audit

# Fix vulnerabilities
npm audit fix

# Force fix (breaking changes)
npm audit fix --force
```

**OWASP Dependency-Check**
```yaml
# GitHub Actions
- name: OWASP Dependency Check
  uses: dependency-check/Dependency-Check_Action@main
  with:
    project: 'my-project'
    path: '.'
    format: 'HTML'
    
- name: Upload results
  uses: actions/upload-artifact@v3
  with:
    name: dependency-check-report
    path: 'reports/'
```

**Snyk**
```bash
# Install Snyk
npm install -g snyk

# Authenticate
snyk auth

# Test for vulnerabilities
snyk test

# Monitor project
snyk monitor

# Fix vulnerabilities
snyk fix
```

### 3. Container Security

**Trivy Scanner**
```bash
# Install Trivy
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin

# Scan Docker image
trivy image myapp:latest

# Scan with severity filter
trivy image --severity HIGH,CRITICAL myapp:latest

# Output as JSON
trivy image -f json -o results.json myapp:latest
```

**Docker Bench Security**
```bash
docker run -it --net host --pid host --userns host --cap-add audit_control \
  -e DOCKER_CONTENT_TRUST=$DOCKER_CONTENT_TRUST \
  -v /etc:/etc:ro \
  -v /usr/bin/containerd:/usr/bin/containerd:ro \
  -v /usr/bin/runc:/usr/bin/runc:ro \
  -v /usr/lib/systemd:/usr/lib/systemd:ro \
  -v /var/lib:/var/lib:ro \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  --label docker_bench_security \
  docker/docker-bench-security
```

**CI/CD Integration**
```yaml
# GitHub Actions
- name: Run Trivy scanner
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'myapp:${{ github.sha }}'
    format: 'sarif'
    output: 'trivy-results.sarif'
    severity: 'CRITICAL,HIGH'

- name: Upload Trivy results
  uses: github/codeql-action/upload-sarif@v2
  with:
    sarif_file: 'trivy-results.sarif'
```

### 4. Secrets Management

**git-secrets**
```bash
# Install git-secrets
brew install git-secrets  # macOS
# or
git clone https://github.com/awslabs/git-secrets

# Initialize in repository
git secrets --install
git secrets --register-aws

# Scan repository
git secrets --scan
git secrets --scan-history
```

**TruffleHog**
```bash
# Install
pip install truffleHog

# Scan repository
trufflehog --regex --entropy=True https://github.com/user/repo
```

**HashiCorp Vault**
```bash
# Start Vault server (dev mode)
vault server -dev

# Set Vault address
export VAULT_ADDR='http://127.0.0.1:8200'

# Store secret
vault kv put secret/myapp/config \
  username='admin' \
  password='secret123'

# Read secret
vault kv get secret/myapp/config

# Use in application
vault kv get -field=password secret/myapp/config
```

**Vault Integration in CI/CD**
```yaml
# GitLab CI
deploy:
  stage: deploy
  script:
    - export VAULT_TOKEN=$(vault write -field=token auth/jwt/login role=myapp-role jwt=$CI_JOB_JWT)
    - export DB_PASSWORD=$(vault kv get -field=password secret/myapp/db)
    - ./deploy.sh
```

### 5. Infrastructure Security Scanning

**Checkov (IaC Security)**
```bash
# Install
pip install checkov

# Scan Terraform
checkov -d /path/to/terraform

# Scan Kubernetes manifests
checkov -d /path/to/k8s

# Scan with specific framework
checkov -f main.tf --framework terraform

# Output as JSON
checkov -d . -o json > results.json
```

**tfsec (Terraform Security)**
```bash
# Install
brew install tfsec

# Scan Terraform
tfsec .

# Scan with specific checks
tfsec --minimum-severity HIGH .

# Exclude checks
tfsec --exclude-rule aws-s3-enable-bucket-encryption .
```

**Example: Secure Terraform**
```hcl
# Bad: Unencrypted S3 bucket
resource "aws_s3_bucket" "bad_example" {
  bucket = "my-bucket"
}

# Good: Encrypted S3 bucket
resource "aws_s3_bucket" "good_example" {
  bucket = "my-bucket"

  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "AES256"
      }
    }
  }

  versioning {
    enabled = true
  }

  logging {
    target_bucket = aws_s3_bucket.logs.id
    target_prefix = "access-logs/"
  }

  lifecycle_rule {
    enabled = true
    
    transition {
      days          = 30
      storage_class = "STANDARD_IA"
    }
  }
}

# Block public access
resource "aws_s3_bucket_public_access_block" "good_example" {
  bucket = aws_s3_bucket.good_example.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

## Security Best Practices

### 1. Code Security

**Input Validation**
```python
# Bad: SQL Injection vulnerable
def get_user(user_id):
    query = f"SELECT * FROM users WHERE id = {user_id}"
    return db.execute(query)

# Good: Parameterized query
def get_user(user_id):
    query = "SELECT * FROM users WHERE id = ?"
    return db.execute(query, (user_id,))
```

**XSS Prevention**
```javascript
// Bad: XSS vulnerable
function displayMessage(msg) {
  document.getElementById('output').innerHTML = msg;
}

// Good: Escaped output
function displayMessage(msg) {
  const div = document.getElementById('output');
  div.textContent = msg;  // Auto-escapes
}
```

### 2. Container Security

**Dockerfile Security**
```dockerfile
# Good practices
FROM node:16-alpine AS build  # Use specific version, minimal base

# Don't run as root
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

# Copy only necessary files
COPY package*.json ./
RUN npm ci --only=production

COPY --chown=nodejs:nodejs . .

# Switch to non-root user
USER nodejs

# Use read-only root filesystem
# Use --read-only flag when running container

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node healthcheck.js

CMD ["node", "server.js"]
```

### 3. Kubernetes Security

**Pod Security**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 1000
    seccompProfile:
      type: RuntimeDefault
  
  containers:
  - name: app
    image: myapp:1.0
    
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
          - ALL
        add:
          - NET_BIND_SERVICE
    
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
      requests:
        memory: "64Mi"
        cpu: "250m"
    
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 30
      periodSeconds: 10
```

**Network Policy**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-specific
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

### 4. Secrets Management

**Never Commit Secrets**
```bash
# .gitignore
.env
.env.local
*.pem
*.key
secrets.yml
config/secrets.yml
```

**Environment Variables (Kubernetes)**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
stringData:
  database-password: "supersecret"
  api-key: "my-api-key"
---
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - name: app
    image: myapp:1.0
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secrets
          key: database-password
    - name: API_KEY
      valueFrom:
        secretKeyRef:
          name: app-secrets
          key: api-key
```

### 5. Authentication & Authorization

**RBAC in Kubernetes**
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-service-account
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-role-binding
subjects:
- kind: ServiceAccount
  name: app-service-account
roleRef:
  kind: Role
  name: app-role
  apiGroup: rbac.authorization.k8s.io
```

## Security Compliance

### CIS Benchmarks
- Industry-accepted security standards
- Available for various technologies
- Automated compliance checking

### OWASP Top 10
1. Broken Access Control
2. Cryptographic Failures
3. Injection
4. Insecure Design
5. Security Misconfiguration
6. Vulnerable Components
7. Authentication Failures
8. Software and Data Integrity Failures
9. Logging and Monitoring Failures
10. Server-Side Request Forgery (SSRF)

### Compliance as Code

**Open Policy Agent (OPA)**
```rego
# Policy: All containers must not run as root
package kubernetes.admission

deny[msg] {
  input.request.kind.kind == "Pod"
  container := input.request.object.spec.containers[_]
  not container.securityContext.runAsNonRoot
  msg := sprintf("Container %v must set runAsNonRoot to true", [container.name])
}

# Policy: All images must be from approved registry
deny[msg] {
  input.request.kind.kind == "Pod"
  container := input.request.object.spec.containers[_]
  not startswith(container.image, "myregistry.io/")
  msg := sprintf("Container %v uses unapproved registry", [container.name])
}
```

## Security Monitoring

### Falco (Runtime Security)
```yaml
# Falco rule
- rule: Unauthorized Process
  desc: Detect unauthorized process execution
  condition: >
    spawned_process and
    container and
    not proc.name in (allowed_processes)
  output: >
    Unauthorized process started
    (user=%user.name command=%proc.cmdline container=%container.name)
  priority: WARNING
```

## Interview Questions

1. What is DevSecOps and how is it different from DevOps?
2. What is the difference between SAST and DAST?
3. How do you manage secrets in a CI/CD pipeline?
4. Explain the OWASP Top 10
5. How do you secure Docker containers?
6. What is the principle of least privilege?
7. How do you implement security scanning in CI/CD?
8. What is shift-left security?
9. How do you handle security vulnerabilities in dependencies?
10. Explain security in Kubernetes

## Practical Exercises

1. Implement SAST in CI/CD pipeline
2. Set up dependency scanning
3. Scan Docker images for vulnerabilities
4. Implement secrets management with Vault
5. Create secure Kubernetes deployments
6. Set up security monitoring with Falco
7. Implement compliance checks as code
8. Conduct security audit of infrastructure

## Resources

- [OWASP](https://owasp.org/)
- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks/)
- [Kubernetes Security](https://kubernetes.io/docs/concepts/security/)
- [Docker Security](https://docs.docker.com/engine/security/)
- [HashiCorp Vault](https://www.vaultproject.io/)
- [Falco Documentation](https://falco.org/docs/)
- [DevSecOps Manifesto](https://www.devsecops.org/)
