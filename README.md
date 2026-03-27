# Multi-Tenant Kubernetes GitOps Platform

A GitOps-based Kubernetes platform that demonstrates multi-tenancy using namespaces, RBAC, resource quotas, and workload isolation for multiple teams on a shared EKS cluster.

## Overview

This project provisions and manages isolated Kubernetes environments for **Team A** and **Team B** using declarative YAML manifests stored in Git. Each team gets its own namespace, resource constraints, RBAC policies, and workloads — all applied via `kubectl` as part of a GitOps workflow.

## Repository Structure

```
multi-tenant-k8s-gitops-platform/
├── ns-team-a.yaml          # Namespace definition for Team A
├── ns-team-b.yaml          # Namespace definition for Team B
├── team-a-quota.yaml       # ResourceQuota for Team A namespace
├── team-b-quota.yaml       # ResourceQuota for Team B namespace
├── team-a-role.yaml        # RBAC Role for Team A
├── team-b-role.yaml        # RBAC Role for Team B
├── team-a-binding.yaml     # RoleBinding for Team A
├── team-b-binding.yaml     # RoleBinding for Team B
├── team-a-deploy.yaml      # Nginx Deployment for Team A (2 replicas)
├── team-b-deploy.yaml      # Nginx Deployment for Team B (2 replicas)
├── team-a-backup.yaml      # Backup snapshot of Team A resources
└── team-b-backup.yaml      # Backup snapshot of Team B resources
```

## Features

- **Namespace Isolation** — Each team operates in a dedicated Kubernetes namespace (`team-a`, `team-b`)
- **Resource Quotas** — CPU and memory limits enforced per namespace to prevent resource starvation
- **RBAC** — Least-privilege roles and bindings scoped per team namespace
- **Workload Management** — Nginx deployments with defined resource requests and limits (CPU: 100m–300m, Memory: 128Mi–256Mi)
- **GitOps Workflow** — All cluster state is declared as YAML manifests and version-controlled in Git

## Prerequisites

- Kubernetes cluster (EKS or any compatible cluster)
- `kubectl` configured with cluster access
- Sufficient RBAC permissions to create namespaces, roles, and quotas

## Usage

### Apply all manifests for both teams

```bash
# Create namespaces first
kubectl apply -f ns-team-a.yaml
kubectl apply -f ns-team-b.yaml

# Apply resource quotas
kubectl apply -f team-a-quota.yaml
kubectl apply -f team-b-quota.yaml

# Apply RBAC roles and bindings
kubectl apply -f team-a-role.yaml
kubectl apply -f team-a-binding.yaml
kubectl apply -f team-b-role.yaml
kubectl apply -f team-b-binding.yaml

# Deploy workloads
kubectl apply -f team-a-deploy.yaml
kubectl apply -f team-b-deploy.yaml
```

### Apply everything at once

```bash
kubectl apply -f .
```

### Verify deployments

```bash
kubectl get all -n team-a
kubectl get all -n team-b
kubectl describe resourcequota -n team-a
kubectl describe resourcequota -n team-b
```

### Restore from backup

```bash
# Restore Team A
kubectl apply -f team-a-backup.yaml

# Restore Team B
kubectl apply -f team-b-backup.yaml
```

## Resource Quotas

| Team   | CPU Request | CPU Limit | Memory Request | Memory Limit |
|--------|-------------|-----------|----------------|--------------|
| Team A | 100m/pod    | 300m/pod  | 128Mi/pod      | 256Mi/pod    |
| Team B | 100m/pod    | 300m/pod  | 128Mi/pod      | 256Mi/pod    |

## RBAC Summary

- **Team A** — Full role with permissions scoped to the `team-a` namespace (pods, deployments, services, configmaps, etc.)
- **Team B** — Restricted role scoped to the `team-b` namespace

## Architecture

```
EKS Cluster
├── Namespace: team-a
│   ├── ResourceQuota
│   ├── Role + RoleBinding
│   └── Deployment: nginx-app (2 replicas)
└── Namespace: team-b
    ├── ResourceQuota
    ├── Role + RoleBinding
    └── Deployment: nginx-app (2 replicas)
```

## License

MIT
