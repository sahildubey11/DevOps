# Kubernetes - Container Orchestration

## What is Kubernetes?

Kubernetes (K8s) is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications.

## Why Kubernetes?

- **Auto-scaling**: Automatically scale applications based on demand
- **Self-healing**: Automatically restart failed containers
- **Load Balancing**: Distribute traffic across containers
- **Rolling Updates**: Update applications with zero downtime
- **Service Discovery**: Automatic DNS for services
- **Storage Orchestration**: Automatically mount storage systems
- **Secret Management**: Manage sensitive information
- **Multi-cloud**: Run anywhere (AWS, Azure, GCP, on-prem)

## Kubernetes Architecture

### Control Plane Components

1. **API Server**: Frontend for Kubernetes control plane
2. **etcd**: Key-value store for cluster data
3. **Scheduler**: Assigns pods to nodes
4. **Controller Manager**: Runs controller processes
5. **Cloud Controller Manager**: Cloud-specific control logic

### Node Components

1. **kubelet**: Agent running on each node
2. **kube-proxy**: Network proxy
3. **Container Runtime**: Software to run containers (Docker, containerd)

## Core Concepts

### Pod
- Smallest deployable unit
- One or more containers
- Shared storage and network

### Deployment
- Manages replica sets
- Declarative updates
- Rolling updates and rollbacks

### Service
- Stable network endpoint
- Load balances to pods
- Service discovery

### Namespace
- Virtual clusters
- Resource isolation
- Multi-tenancy

## Installation

### Minikube (Local Development)

```bash
# Install Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Start cluster
minikube start

# Check status
minikube status

# Stop cluster
minikube stop

# Delete cluster
minikube delete
```

### kubectl (CLI Tool)

```bash
# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Verify installation
kubectl version --client

# Configure kubectl
kubectl config view
kubectl config use-context minikube
```

## Essential kubectl Commands

### Cluster Info
```bash
# View cluster info
kubectl cluster-info

# View nodes
kubectl get nodes

# Describe node
kubectl describe node node-name

# View cluster events
kubectl get events
```

### Pod Management
```bash
# List pods
kubectl get pods
kubectl get pods -A  # All namespaces
kubectl get pods -o wide  # More details

# Create pod from file
kubectl apply -f pod.yaml

# Delete pod
kubectl delete pod pod-name

# Describe pod
kubectl describe pod pod-name

# View pod logs
kubectl logs pod-name
kubectl logs pod-name -c container-name  # Specific container
kubectl logs -f pod-name  # Follow logs

# Execute command in pod
kubectl exec -it pod-name -- bash

# Port forward
kubectl port-forward pod-name 8080:80
```

### Deployment Management
```bash
# Create deployment
kubectl create deployment nginx --image=nginx

# Get deployments
kubectl get deployments

# Describe deployment
kubectl describe deployment nginx

# Scale deployment
kubectl scale deployment nginx --replicas=3

# Update image
kubectl set image deployment/nginx nginx=nginx:1.21

# Rollout status
kubectl rollout status deployment/nginx

# Rollout history
kubectl rollout history deployment/nginx

# Rollback
kubectl rollout undo deployment/nginx

# Delete deployment
kubectl delete deployment nginx
```

### Service Management
```bash
# Expose deployment
kubectl expose deployment nginx --port=80 --type=LoadBalancer

# Get services
kubectl get services

# Describe service
kubectl describe service nginx

# Delete service
kubectl delete service nginx
```

### Namespace Management
```bash
# List namespaces
kubectl get namespaces

# Create namespace
kubectl create namespace dev

# Set default namespace
kubectl config set-context --current --namespace=dev

# Delete namespace
kubectl delete namespace dev
```

## YAML Manifests

### Pod Definition

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
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
    env:
    - name: ENV
      value: "production"
```

### Deployment Definition

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
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
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

### Service Definition

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

### Service Types

1. **ClusterIP**: Internal cluster IP (default)
2. **NodePort**: Exposes on each node's IP
3. **LoadBalancer**: Cloud provider load balancer
4. **ExternalName**: Maps to external DNS name

### ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "postgres://db:5432"
  log_level: "info"
  config.json: |
    {
      "env": "production",
      "debug": false
    }
```

### Using ConfigMap

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    envFrom:
    - configMapRef:
        name: app-config
    volumeMounts:
    - name: config
      mountPath: /etc/config
  volumes:
  - name: config
    configMap:
      name: app-config
```

### Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  username: YWRtaW4=  # base64 encoded
  password: cGFzc3dvcmQ=  # base64 encoded
```

### Using Secret

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    env:
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: password
```

### PersistentVolume

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-data
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: standard
  hostPath:
    path: /data
```

### PersistentVolumeClaim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-data
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: standard
```

### Using PVC

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    volumeMounts:
    - name: data
      mountPath: /app/data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: pvc-data
```

## Ingress

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
            name: app-service
            port:
              number: 80
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
```

## StatefulSet

For stateful applications (databases, etc.)

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:14
        ports:
        - containerPort: 5432
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
```

## DaemonSet

Runs one pod on each node

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      containers:
      - name: node-exporter
        image: prom/node-exporter
        ports:
        - containerPort: 9100
```

## Jobs and CronJobs

### Job

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: data-migration
spec:
  template:
    spec:
      containers:
      - name: migration
        image: myapp:1.0
        command: ["python", "migrate.py"]
      restartPolicy: Never
  backoffLimit: 4
```

### CronJob

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup
spec:
  schedule: "0 2 * * *"  # 2 AM daily
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: backup:1.0
            command: ["./backup.sh"]
          restartPolicy: OnFailure
```

## Horizontal Pod Autoscaler

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

## Resource Management

### Resource Requests and Limits

```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
```

### ResourceQuota

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: dev
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    pods: "10"
```

## Best Practices

### 1. Resource Management
- Always set resource requests and limits
- Use namespaces for isolation
- Implement resource quotas

### 2. Health Checks
- Configure liveness probes
- Configure readiness probes
- Set appropriate timeouts

### 3. Configuration
- Use ConfigMaps for configuration
- Use Secrets for sensitive data
- Never hardcode configuration

### 4. Security
- Use RBAC for access control
- Run as non-root user
- Scan images for vulnerabilities
- Use NetworkPolicies

### 5. High Availability
- Run multiple replicas
- Use anti-affinity rules
- Distribute across zones
- Implement PodDisruptionBudget

### 6. Monitoring
- Collect metrics
- Centralize logs
- Set up alerts
- Monitor resource usage

## Helm - Package Manager

### Install Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### Helm Commands

```bash
# Add repository
helm repo add bitnami https://charts.bitnami.com/bitnami

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
```

## Troubleshooting

### Debug Commands

```bash
# View pod events
kubectl describe pod pod-name

# Check logs
kubectl logs pod-name
kubectl logs pod-name --previous  # Previous container

# Debug with ephemeral container
kubectl debug pod-name -it --image=busybox

# Check resource usage
kubectl top nodes
kubectl top pods

# View events
kubectl get events --sort-by=.metadata.creationTimestamp

# Network debugging
kubectl run debug --image=nicolaka/netshoot -it --rm

# Check API resources
kubectl api-resources
```

## Interview Questions

1. What is Kubernetes and why is it used?
2. Explain Kubernetes architecture
3. What is the difference between a Pod and a Deployment?
4. Explain different types of Services
5. What is the difference between ConfigMap and Secret?
6. How does auto-scaling work in Kubernetes?
7. What is the difference between StatefulSet and Deployment?
8. Explain Kubernetes networking
9. What are Persistent Volumes and PersistentVolumeClaims?
10. How do you perform rolling updates and rollbacks?

## Practical Exercises

1. Deploy a multi-tier application (frontend, backend, database)
2. Configure auto-scaling for an application
3. Set up Ingress for multiple services
4. Implement ConfigMaps and Secrets
5. Create a StatefulSet for a database
6. Set up a CronJob for periodic tasks
7. Configure resource limits and requests
8. Practice troubleshooting failed deployments

## Resources

- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Kubernetes Tutorials](https://kubernetes.io/docs/tutorials/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Play with Kubernetes](https://labs.play-with-k8s.com/)
- [Kubernetes the Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way)
- [Helm Documentation](https://helm.sh/docs/)
