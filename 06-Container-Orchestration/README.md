# Container Orchestration with Kubernetes

## What is Container Orchestration?

Container orchestration automates the deployment, management, scaling, and networking of containers. It handles:
- Container deployment and scheduling
- Load balancing and service discovery
- Scaling (up/down, in/out)
- Health monitoring and self-healing
- Rolling updates and rollbacks
- Resource allocation

## What is Kubernetes (K8s)?

Kubernetes is an open-source container orchestration platform originally developed by Google. It's now maintained by the Cloud Native Computing Foundation (CNCF).

### Key Features
- **Automated Deployment**: Deploy containers across cluster
- **Self-Healing**: Restart failed containers, replace nodes
- **Auto-Scaling**: Scale based on CPU/memory/custom metrics
- **Load Balancing**: Distribute traffic across containers
- **Rolling Updates**: Update with zero downtime
- **Service Discovery**: Containers can find each other
- **Storage Orchestration**: Automatically mount storage
- **Secret Management**: Manage sensitive information

## Kubernetes Architecture

### Control Plane Components

```
┌─────────────────────────────────────────┐
│         Control Plane (Master)          │
│                                         │
│  ┌──────────┐  ┌───────────┐          │
│  │   API    │  │ Scheduler │          │
│  │  Server  │  │           │          │
│  └──────────┘  └───────────┘          │
│                                         │
│  ┌──────────┐  ┌───────────┐          │
│  │Controller│  │   etcd    │          │
│  │ Manager  │  │(Key-Value)│          │
│  └──────────┘  └───────────┘          │
└─────────────────────────────────────────┘
            │
    ┌───────┴────────┐
    │                │
┌───▼────┐      ┌────▼───┐
│ Node 1 │      │ Node 2 │
│        │      │        │
│ Pods   │      │ Pods   │
└────────┘      └────────┘
```

#### API Server
- Front-end for Kubernetes control plane
- Exposes REST API
- All communication goes through API server

#### etcd
- Distributed key-value store
- Stores all cluster data
- Source of truth for cluster state

#### Scheduler
- Assigns pods to nodes
- Considers resources, constraints, affinity

#### Controller Manager
- Runs controller processes
- Node controller, Replication controller, etc.

### Node Components

#### kubelet
- Agent running on each node
- Ensures containers are running in pods
- Reports node status to control plane

#### kube-proxy
- Network proxy on each node
- Maintains network rules
- Enables service communication

#### Container Runtime
- Software to run containers
- Docker, containerd, CRI-O

## Core Concepts

### Pod
Smallest deployable unit in Kubernetes. A pod contains one or more containers.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
```

### ReplicaSet
Ensures a specified number of pod replicas are running.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
```

### Deployment
Manages ReplicaSets and provides declarative updates.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

### Service
Exposes pods as a network service.

#### ClusterIP (default)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

#### NodePort
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
```

#### LoadBalancer
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```

### ConfigMap
Store configuration data.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  app.properties: |
    color=blue
    environment=production
  database.url: "postgres://db:5432"
```

### Secret
Store sensitive information.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=  # base64 encoded
  password: cGFzc3dvcmQ=
```

### Namespace
Virtual clusters within a physical cluster.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: development
```

### Volume
Persistent storage for pods.

#### PersistentVolume (PV)
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-volume
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data
```

#### PersistentVolumeClaim (PVC)
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

## kubectl Commands

### Cluster Info
```bash
# Cluster information
kubectl cluster-info

# Node list
kubectl get nodes

# Component status
kubectl get componentstatuses
```

### Working with Resources

```bash
# Get resources
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get all

# Get with more details
kubectl get pods -o wide
kubectl get pods --all-namespaces
kubectl get pods -n development

# Describe resource
kubectl describe pod nginx-pod
kubectl describe deployment nginx-deployment

# Create from file
kubectl create -f deployment.yaml
kubectl apply -f deployment.yaml

# Create from directory
kubectl apply -f ./configs/

# Delete resources
kubectl delete pod nginx-pod
kubectl delete -f deployment.yaml
kubectl delete deployment nginx-deployment
```

### Pod Management

```bash
# Create pod
kubectl run nginx --image=nginx:1.21

# Execute command in pod
kubectl exec -it nginx-pod -- /bin/bash
kubectl exec nginx-pod -- ls /

# View logs
kubectl logs nginx-pod
kubectl logs -f nginx-pod  # Follow logs
kubectl logs nginx-pod --previous  # Previous instance

# Port forwarding
kubectl port-forward nginx-pod 8080:80

# Copy files
kubectl cp nginx-pod:/var/log/nginx/access.log ./access.log
kubectl cp ./config.json nginx-pod:/etc/config.json
```

### Deployment Management

```bash
# Create deployment
kubectl create deployment nginx --image=nginx:1.21

# Scale deployment
kubectl scale deployment nginx --replicas=5

# Autoscale
kubectl autoscale deployment nginx --min=2 --max=10 --cpu-percent=80

# Update image
kubectl set image deployment/nginx nginx=nginx:1.22

# Rollout status
kubectl rollout status deployment/nginx

# Rollout history
kubectl rollout history deployment/nginx

# Rollback
kubectl rollout undo deployment/nginx
kubectl rollout undo deployment/nginx --to-revision=2

# Pause/Resume rollout
kubectl rollout pause deployment/nginx
kubectl rollout resume deployment/nginx
```

### Service Management

```bash
# Expose deployment
kubectl expose deployment nginx --port=80 --type=LoadBalancer

# Get service endpoints
kubectl get endpoints nginx-service
```

### ConfigMap & Secret

```bash
# Create ConfigMap
kubectl create configmap app-config --from-file=app.properties
kubectl create configmap app-config --from-literal=color=blue

# Create Secret
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=secret

# Get secret data
kubectl get secret db-secret -o jsonpath='{.data.password}' | base64 -d
```

### Labels and Selectors

```bash
# Add label
kubectl label pod nginx-pod env=production

# Update label
kubectl label pod nginx-pod env=staging --overwrite

# Remove label
kubectl label pod nginx-pod env-

# Select by label
kubectl get pods -l app=nginx
kubectl get pods -l 'env in (production,staging)'
```

### Namespace Management

```bash
# Create namespace
kubectl create namespace development

# Set default namespace
kubectl config set-context --current --namespace=development

# Get resources in namespace
kubectl get pods -n development
```

## Advanced Concepts

### StatefulSet
For stateful applications (databases, etc.).

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

### DaemonSet
Runs a pod on every node.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: logging-agent
spec:
  selector:
    matchLabels:
      app: logging
  template:
    metadata:
      labels:
        app: logging
    spec:
      containers:
      - name: fluentd
        image: fluentd:v1.14
```

### Job
Run-to-completion tasks.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: data-import
spec:
  template:
    spec:
      containers:
      - name: importer
        image: data-importer:1.0
      restartPolicy: OnFailure
  backoffLimit: 4
```

### CronJob
Scheduled jobs.

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup
spec:
  schedule: "0 2 * * *"  # Daily at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: backup-tool:1.0
          restartPolicy: OnFailure
```

### Ingress
HTTP/HTTPS routing.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
```

## Helm - Package Manager

### What is Helm?
Helm is a package manager for Kubernetes, using "charts" to define, install, and upgrade applications.

### Installation
```bash
# macOS
brew install helm

# Linux
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### Helm Commands

```bash
# Add repository
helm repo add stable https://charts.helm.sh/stable
helm repo add bitnami https://charts.bitnami.com/bitnami

# Update repositories
helm repo update

# Search charts
helm search repo nginx

# Install chart
helm install my-nginx bitnami/nginx

# List releases
helm list

# Upgrade release
helm upgrade my-nginx bitnami/nginx

# Rollback
helm rollback my-nginx 1

# Uninstall
helm uninstall my-nginx

# Create chart
helm create mychart

# Package chart
helm package mychart/

# Install from local chart
helm install my-app ./mychart
```

### Chart Structure

```
mychart/
├── Chart.yaml          # Chart metadata
├── values.yaml         # Default configuration
├── charts/             # Dependencies
├── templates/          # Kubernetes manifests
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   └── _helpers.tpl
└── README.md
```

## Best Practices

### 1. Resource Requests and Limits
```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
```

### 2. Health Checks
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

### 3. Use Labels Effectively
```yaml
metadata:
  labels:
    app: myapp
    version: v1.0
    environment: production
    tier: frontend
```

### 4. Use Namespaces
Separate environments and teams.

### 5. Security
- Use RBAC (Role-Based Access Control)
- Network Policies
- Pod Security Policies
- Don't run as root

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 2000
```

### 6. ConfigMaps and Secrets
Externalize configuration, don't hardcode.

### 7. Rolling Updates
```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
```

## Monitoring and Logging

### Monitoring Tools
- **Prometheus**: Metrics collection
- **Grafana**: Visualization
- **Metrics Server**: Resource metrics

```bash
# Install metrics server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# View metrics
kubectl top nodes
kubectl top pods
```

### Logging
- **EFK Stack**: Elasticsearch, Fluentd, Kibana
- **ELK Stack**: Elasticsearch, Logstash, Kibana
- **Loki**: Grafana Loki

## Local Development

### Minikube
```bash
# Install
brew install minikube  # macOS

# Start cluster
minikube start

# Stop cluster
minikube stop

# Delete cluster
minikube delete

# Dashboard
minikube dashboard
```

### Kind (Kubernetes in Docker)
```bash
# Install
brew install kind

# Create cluster
kind create cluster

# Delete cluster
kind delete cluster
```

### k3s
Lightweight Kubernetes distribution.

## Resources

- Kubernetes Documentation: https://kubernetes.io/docs/
- Kubernetes GitHub: https://github.com/kubernetes/kubernetes
- CNCF: https://www.cncf.io/
- Helm: https://helm.sh/
- Kubernetes Patterns: https://www.redhat.com/en/resources/cloud-native-container-design-whitepaper

## Next Steps

Continue to [Infrastructure as Code (Terraform)](../07-Infrastructure-as-Code/) to learn about provisioning infrastructure.
