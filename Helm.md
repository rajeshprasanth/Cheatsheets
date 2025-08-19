# Helm Cheat Sheet

## Table of Contents

1. [Basics](#basics)
2. [Repositories](#repositories)
3. [Charts](#charts)
4. [Releases](#releases)
5. [Values](#values)
6. [Upgrade & Rollback](#upgrade--rollback)
7. [Uninstall](#uninstall)
8. [Helm Template & Debug](#helm-template--debug)
9. [Homelab Use Cases](#homelab-use-cases)

---

## Basics

```text
# Helm overview
Helm → Kubernetes package manager
Charts → Pre-configured Kubernetes resources
Release → Deployed instance of a chart
```

---

## Repositories

```bash
# Add repo
helm repo add <name> <url>

# List repos
helm repo list

# Update repos
helm repo update

# Remove repo
helm repo remove <name>
```

---

## Charts

```bash
# Search charts in repos
helm search repo <keyword>

# Download chart
helm pull <repo/chart> --untar

# Create new chart
helm create <chart-name>

# Package chart
helm package <chart-directory>
```

---

## Releases

```bash
# Install chart
helm install <release-name> <chart> [--namespace <ns>]

# List releases
helm list [--all-namespaces]

# Get release status
helm status <release-name>

# Get release history
helm history <release-name>
```

---

## Values

```bash
# View chart default values
helm show values <chart>

# Override values during install
helm install <release-name> <chart> -f custom-values.yaml

# Set single value inline
helm install <release-name> <chart> --set key=value

# Upgrade with new values
helm upgrade <release-name> <chart> -f custom-values.yaml
```

---

## Upgrade & Rollback

```bash
# Upgrade release
helm upgrade <release-name> <chart> -f values.yaml

# Rollback to previous revision
helm rollback <release-name> <revision-number>

# Dry-run upgrade
helm upgrade <release-name> <chart> --dry-run
```

---

## Uninstall

```bash
# Uninstall release
helm uninstall <release-name> [--namespace <ns>]

# Purge all releases (cleanup)
helm list --all-namespaces
helm uninstall <release-name> --namespace <ns>
```

---

## Helm Template & Debug

```bash
# Render Kubernetes manifests locally
helm template <release-name> <chart>

# Dry-run install (debug)
helm install <release-name> <chart> --dry-run --debug
```

---

## Homelab Use Cases

```bash
# Deploy Prometheus stack via Helm
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prom-stack prometheus-community/kube-prometheus-stack

# Deploy Nginx Ingress
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install nginx-ingress ingress-nginx/ingress-nginx --namespace ingress --create-namespace

# Upgrade a release with custom values
helm upgrade prom-stack prometheus-community/kube-prometheus-stack -f custom-values.yaml
```

---

## Useful Command Summary

| Command                              | Description                           |
| ------------------------------------ | ------------------------------------- |
| `helm repo add <name> <url>`         | Add Helm repository                   |
| `helm repo update`                   | Update chart repos                    |
| `helm search repo <keyword>`         | Search for charts                     |
| `helm install <release> <chart>`     | Install chart as release              |
| `helm upgrade <release> <chart>`     | Upgrade chart release                 |
| `helm rollback <release> <revision>` | Rollback release to previous revision |
| `helm uninstall <release>`           | Uninstall release                     |
| `helm show values <chart>`           | Show chart default values             |
| `helm template <release> <chart>`    | Render manifests locally              |
| `helm list`                          | List all releases                     |
| `helm status <release>`              | Show release status                   |


