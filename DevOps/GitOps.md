# GitOps Cheat Sheet

## Table of Contents

1. [Basics](#basics)
2. [FluxCD](#fluxcd)
3. [ArgoCD](#argocd)
4. [Repositories & GitOps Workflow](#repositories--gitops-workflow)
5. [Sync & Rollback](#sync--rollback)
6. [CLI & Troubleshooting](#cli--troubleshooting)
7. [Homelab Use Cases](#homelab-use-cases)

---

## Basics

```text
# GitOps overview
GitOps → Manage Kubernetes infrastructure via Git as the source of truth
All changes are pushed to Git → automatically applied to clusters
Key tools: FluxCD, ArgoCD
```

---

## FluxCD

```bash
# Install Flux CLI
brew install fluxcd/tap/flux       # macOS
sudo apt install flux-cli           # Debian/Ubuntu

# Bootstrap Flux in cluster (Git repo)
flux bootstrap github \
  --owner=<github-user> \
  --repository=<repo-name> \
  --branch=main \
  --path=clusters/my-cluster

# Check Flux components status
flux check

# List applied resources
flux get all

# Apply changes manually
flux reconcile source git <source-name>
flux reconcile kustomization <kustomization-name>
```

### Key Concepts

```text
# Source → Git repository or Helm repo
# Kustomization → Kubernetes manifests to apply
# GitRepository → Flux pulls manifests from Git
# HelmRepository → Flux pulls Helm charts
# Notification → Flux alerts on changes or failures
```

---

## ArgoCD

```bash
# Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Access ArgoCD UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Login (CLI)
argocd login <server-ip>:8080 --username admin --password <password> --insecure

# Create application (sync Git repo to cluster)
argocd app create <app-name> \
  --repo https://github.com/<user>/<repo>.git \
  --path ./clusters/my-cluster \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default

# Sync application
argocd app sync <app-name>

# Check application status
argocd app get <app-name>

# Rollback application
argocd app rollback <app-name> <revision>
```

---

## Repositories & GitOps Workflow

```text
# GitOps workflow
1. Define cluster manifests in Git repo
2. Push changes to branch
3. FluxCD or ArgoCD detects changes
4. Apply changes to cluster automatically
5. Monitor status and alerts
```

---

## Sync & Rollback

```bash
# Force sync (FluxCD)
flux reconcile kustomization <name> --with-source

# Automatic rollback (ArgoCD)
argocd app rollback <app-name> <revision-number>

# Check history of deployments (ArgoCD)
argocd app history <app-name>
```

---

## CLI & Troubleshooting

```bash
# FluxCD
flux logs
flux get all
flux reconcile source git <source-name>

# ArgoCD
argocd app list
argocd app get <app-name>
kubectl logs -n argocd deployment/argocd-application-controller

# Common issues
# - Wrong Git path
# - Incorrect branch
# - Manifest syntax errors
```

---

## Homelab Use Cases

```text
# Deploy home lab apps via GitOps
- Home Assistant manifests in Git → auto-deployed
- Monitoring stack (Prometheus/Grafana) via FluxCD
- Ingress controller via ArgoCD
- Centralized repo as single source of truth
```

---

## Useful Command Summary

| Command                               | Description                     |
| ------------------------------------- | ------------------------------- |
| `flux bootstrap github ...`           | Bootstrap cluster with FluxCD   |
| `flux get all`                        | List Flux-managed resources     |
| `flux reconcile kustomization <name>` | Force sync FluxCD kustomization |
| `argocd login <server>`               | Login to ArgoCD                 |
| `argocd app create <app>`             | Create ArgoCD application       |
| `argocd app sync <app>`               | Sync application to cluster     |
| `argocd app rollback <app> <rev>`     | Rollback application            |
| `argocd app get <app>`                | Get status of application       |
| `argocd app list`                     | List all ArgoCD apps            |
| `kubectl logs -n argocd ...`          | Check ArgoCD controller logs    |
