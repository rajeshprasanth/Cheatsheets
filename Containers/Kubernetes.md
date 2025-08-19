# Kubernetes Cheat Sheet

## Basic kubectl Commands

### Cluster Information
```bash
# Check cluster info
kubectl cluster-info

# Get cluster nodes
kubectl get nodes

# Describe a node
kubectl describe node <node-name>

# Get all resources in all namespaces
kubectl get all --all-namespaces
```

### Namespaces
```bash
# List namespaces
kubectl get namespaces
kubectl get ns

# Create namespace
kubectl create namespace <namespace>

# Delete namespace
kubectl delete namespace <namespace>

# Set default namespace for current context
kubectl config set-context --current --namespace=<namespace>
```

## Pod Management

### Basic Pod Operations
```bash
# List pods
kubectl get pods
kubectl get po

# List pods in specific namespace
kubectl get pods -n <namespace>

# Get detailed pod information
kubectl describe pod <pod-name>

# Create pod from YAML
kubectl create -f pod.yaml
kubectl apply -f pod.yaml

# Delete pod
kubectl delete pod <pod-name>

# Get pod logs
kubectl logs <pod-name>

# Follow logs
kubectl logs -f <pod-name>

# Execute command in pod
kubectl exec <pod-name> -- <command>

# Interactive shell in pod
kubectl exec -it <pod-name> -- /bin/bash

# Port forwarding
kubectl port-forward pod/<pod-name> <local-port>:<pod-port>
```

### Pod YAML Example
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: my-app
spec:
  containers:
  - name: my-container
    image: nginx:1.20
    ports:
    - containerPort: 80
    env:
    - name: ENV_VAR
      value: "production"
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

## Deployments

### Deployment Commands
```bash
# Create deployment
kubectl create deployment <name> --image=<image>

# Get deployments
kubectl get deployments
kubectl get deploy

# Describe deployment
kubectl describe deployment <deployment-name>

# Scale deployment
kubectl scale deployment <deployment-name> --replicas=<count>

# Update deployment image
kubectl set image deployment/<deployment-name> <container>=<new-image>

# Rollout status
kubectl rollout status deployment/<deployment-name>

# Rollout history
kubectl rollout history deployment/<deployment-name>

# Rollback deployment
kubectl rollout undo deployment/<deployment-name>

# Delete deployment
kubectl delete deployment <deployment-name>
```

### Deployment YAML Example
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: my-app:1.0
        ports:
        - containerPort: 8080
        env:
        - name: DATABASE_URL
          value: "postgres://localhost:5432/mydb"
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

## Services

### Service Commands
```bash
# Get services
kubectl get services
kubectl get svc

# Describe service
kubectl describe service <service-name>

# Expose deployment as service
kubectl expose deployment <deployment-name> --port=80 --target-port=8080

# Delete service
kubectl delete service <service-name>
```

### Service YAML Examples

#### ClusterIP Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: ClusterIP
```

#### NodePort Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-nodeport
spec:
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
    nodePort: 30080
  type: NodePort
```

#### LoadBalancer Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-lb
spec:
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: LoadBalancer
```

## ConfigMaps and Secrets

### ConfigMaps
```bash
# Create configmap from literal
kubectl create configmap <name> --from-literal=key1=value1 --from-literal=key2=value2

# Create configmap from file
kubectl create configmap <name> --from-file=<file-path>

# Get configmaps
kubectl get configmaps
kubectl get cm

# Describe configmap
kubectl describe configmap <name>

# Delete configmap
kubectl delete configmap <name>
```

#### ConfigMap YAML
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database.host: "localhost"
  database.port: "5432"
  app.properties: |
    debug=true
    log.level=info
```

### Secrets
```bash
# Create secret from literal
kubectl create secret generic <name> --from-literal=username=admin --from-literal=password=secret

# Create secret from file
kubectl create secret generic <name> --from-file=<file-path>

# Get secrets
kubectl get secrets

# Describe secret
kubectl describe secret <name>

# Delete secret
kubectl delete secret <name>
```

#### Secret YAML
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  username: YWRtaW4=  # base64 encoded 'admin'
  password: c2VjcmV0  # base64 encoded 'secret'
```

## Volumes and Persistent Storage

### Persistent Volume (PV)
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
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

### Persistent Volume Claim (PVC)
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: standard
```

### Using PVC in Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-storage
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: storage
      mountPath: /data
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: my-pvc
```

## Ingress

### Ingress Commands
```bash
# Get ingress
kubectl get ingress
kubectl get ing

# Describe ingress
kubectl describe ingress <ingress-name>
```

### Ingress YAML
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
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
            name: my-app-service
            port:
              number: 80
  - host: api.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
```

## Horizontal Pod Autoscaler (HPA)

```bash
# Create HPA
kubectl autoscale deployment <deployment-name> --cpu-percent=70 --min=1 --max=10

# Get HPA
kubectl get hpa

# Describe HPA
kubectl describe hpa <hpa-name>
```

### HPA YAML
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

## Jobs and CronJobs

### Job YAML
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: my-job
spec:
  template:
    spec:
      containers:
      - name: job-container
        image: busybox
        command: ["sh", "-c", "echo Hello World && sleep 30"]
      restartPolicy: Never
  backoffLimit: 4
```

### CronJob YAML
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: my-cronjob
spec:
  schedule: "0 2 * * *"  # Run at 2 AM every day
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: cron-container
            image: busybox
            command: ["sh", "-c", "echo Scheduled job running"]
          restartPolicy: OnFailure
```

## RBAC (Role-Based Access Control)

### Role YAML
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

### RoleBinding YAML
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

## Common kubectl Commands

### Resource Management
```bash
# Apply configuration
kubectl apply -f <file.yaml>
kubectl apply -f <directory>/

# Delete resources
kubectl delete -f <file.yaml>
kubectl delete <resource-type> <resource-name>

# Get resources with wide output
kubectl get <resource> -o wide

# Get resource in YAML/JSON
kubectl get <resource> <name> -o yaml
kubectl get <resource> <name> -o json

# Edit resource
kubectl edit <resource> <name>

# Patch resource
kubectl patch <resource> <name> -p '<patch-data>'
```

### Debugging and Troubleshooting
```bash
# Get events
kubectl get events --sort-by=.metadata.creationTimestamp

# Debug pod issues
kubectl describe pod <pod-name>
kubectl logs <pod-name> --previous

# Check resource usage
kubectl top nodes
kubectl top pods

# Debug network issues
kubectl exec -it <pod-name> -- nslookup <service-name>
kubectl exec -it <pod-name> -- wget -qO- <service-name>

# Copy files to/from pod
kubectl cp <file-path> <pod-name>:<container-path>
kubectl cp <pod-name>:<container-path> <file-path>
```

### Context and Configuration
```bash
# View current context
kubectl config current-context

# List all contexts
kubectl config get-contexts

# Switch context
kubectl config use-context <context-name>

# Set namespace for context
kubectl config set-context --current --namespace=<namespace>

# View configuration
kubectl config view
```

## Useful Aliases
Add these to your shell configuration:
```bash
alias k=kubectl
alias kgp='kubectl get pods'
alias kgs='kubectl get svc'
alias kgd='kubectl get deployment'
alias kdp='kubectl describe pod'
alias kds='kubectl describe svc'
alias kdd='kubectl describe deployment'
alias kaf='kubectl apply -f'
alias kdf='kubectl delete -f'
```

## Common Patterns

### Complete Application Stack
```yaml
# Namespace
apiVersion: v1
kind: Namespace
metadata:
  name: my-app

---
# ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: my-app
data:
  DATABASE_HOST: postgres-service
  DATABASE_PORT: "5432"

---
# Secret
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
  namespace: my-app
type: Opaque
data:
  DATABASE_PASSWORD: cGFzc3dvcmQ=

---
# Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: my-app:latest
        ports:
        - containerPort: 8080
        envFrom:
        - configMapRef:
            name: app-config
        - secretRef:
            name: app-secret

---
# Service
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
  namespace: my-app
spec:
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP

---
# Ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  namespace: my-app
spec:
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app-service
            port:
              number: 80
```

## Troubleshooting Guide

| Issue | Command | Description |
|-------|---------|-------------|
| Pod not starting | `kubectl describe pod <name>` | Check events and conditions |
| Container crashing | `kubectl logs <pod> --previous` | View logs from previous container |
| Service not accessible | `kubectl get endpoints <service>` | Check if endpoints exist |
| DNS issues | `kubectl exec -it <pod> -- nslookup kubernetes.default` | Test DNS resolution |
| Resource constraints | `kubectl top pods` | Check resource usage |
| Permission issues | `kubectl auth can-i <verb> <resource>` | Check RBAC permissions |
