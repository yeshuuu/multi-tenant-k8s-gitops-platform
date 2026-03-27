# Multi-Tenant Kubernetes GitOps Platform

A GitOps-based Kubernetes platform demonstrating multi-tenancy using namespaces, RBAC, resource quotas, workload isolation, and backup snapshots for multiple teams on a shared EKS cluster.

## Overview

This project provisions and manages isolated Kubernetes environments for **Team A** and **Team B** using declarative YAML manifests stored in Git. Each team gets its own namespace, resource constraints, RBAC policies, and workloads — all applied via `kubectl` as part of a GitOps workflow.

## Repository Structure

```
multi-tenant-k8s-gitops-platform/
├── day1-multi-tenant/              # Core manifests organized by team
│   ├── team-a/
│   │   ├── namespace.yaml          # Namespace: team-a
│   │   ├── resourcequota.yaml      # Namespace-level resource limits
│   │   ├── role.yaml               # RBAC Role scoped to team-a
│   │   ├── rolebinding.yaml        # Binds team-a-role → team-a-user
│   │   ├── deployment.yaml         # nginx-app Deployment (2 replicas)
│   │   └── service.yaml            # ClusterIP Service for nginx-app
│   └── team-b/
│       ├── namespace.yaml          # Namespace: team-b
│       ├── resourcequota.yaml      # Namespace-level resource limits
│       ├── role.yaml               # RBAC Role scoped to team-b
│       ├── rolebinding.yaml        # Binds team-b-role → team-b-user
│       ├── deployment.yaml         # nginx-app Deployment (2 replicas)
│       └── service.yaml            # ClusterIP Service for nginx-app
├── backups/
│   ├── team-a-backup.yaml          # Live snapshot: kubectl get all -n team-a -o yaml
│   └── team-b-backup.yaml          # Live snapshot: kubectl get all -n team-b -o yaml
├── .gitignore
└── README.md
```

## Features

- **Namespace Isolation** — Each team operates in a dedicated Kubernetes namespace (`team-a`, `team-b`)
- **Resource Quotas** — CPU and memory limits enforced at the namespace level
- **RBAC** — Least-privilege roles and bindings scoped per team namespace
- **Workload Management** — Nginx deployments with resource requests/limits and ClusterIP services
- **Backup Snapshots** — Live `kubectl` state snapshots stored per team for recovery
- **GitOps Workflow** — All cluster state is declared as YAML manifests and version-controlled in Git

## Prerequisites

- Kubernetes cluster (EKS or any compatible cluster)
- `kubectl` configured with cluster access
- Sufficient RBAC permissions to create namespaces, roles, and quotas

## Usage

### Apply all manifests for a team

```bash
# Apply everything for Team A (in dependency order)
kubectl apply -f day1-multi-tenant/team-a/namespace.yaml
kubectl apply -f day1-multi-tenant/team-a/resourcequota.yaml
kubectl apply -f day1-multi-tenant/team-a/role.yaml
kubectl apply -f day1-multi-tenant/team-a/rolebinding.yaml
kubectl apply -f day1-multi-tenant/team-a/deployment.yaml
kubectl apply -f day1-multi-tenant/team-a/service.yaml

# Apply everything for Team B
kubectl apply -f day1-multi-tenant/team-b/namespace.yaml
kubectl apply -f day1-multi-tenant/team-b/resourcequota.yaml
kubectl apply -f day1-multi-tenant/team-b/role.yaml
kubectl apply -f day1-multi-tenant/team-b/rolebinding.yaml
kubectl apply -f day1-multi-tenant/team-b/deployment.yaml
kubectl apply -f day1-multi-tenant/team-b/service.yaml
```

### Apply an entire team at once

```bash
kubectl apply -f day1-multi-tenant/team-a/
kubectl apply -f day1-multi-tenant/team-b/
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
kubectl apply -f backups/team-a-backup.yaml

# Restore Team B
kubectl apply -f backups/team-b-backup.yaml
```

## Resource Quotas

Both teams share the same namespace-level quota limits:

| Quota Field     | Team A  | Team B  |
|-----------------|---------|---------|
| Max Pods        | 4       | 4       |
| CPU Requests    | 1 core  | 1 core  |
| CPU Limits      | 2 cores | 2 cores |
| Memory Requests | 1Gi     | 1Gi     |
| Memory Limits   | 2Gi     | 2Gi     |

> Per-pod resource requests: `cpu: 100m`, `memory: 128Mi` · limits: `cpu: 300m`, `memory: 256Mi`

## RBAC Summary

Both teams have equivalent roles scoped strictly to their own namespace:

| Permission  | Resources                   | Verbs                     |
|-------------|-----------------------------|---------------------------|
| Team A Role | pods, deployments, services | get, list, create, delete |
| Team B Role | pods, deployments, services | get, list, create, delete |

- Subjects are bound via `RoleBinding` to `team-a-user` and `team-b-user` respectively
- Cross-namespace access is not permitted

## Architecture

```
EKS Cluster
├── Namespace: team-a
│   ├── ResourceQuota    (pods: 4, cpu: 1-2 cores, mem: 1-2Gi)
│   ├── Role + RoleBinding → team-a-user
│   ├── Deployment: nginx-app (2 replicas)
│   └── Service: nginx-service (ClusterIP :80)
└── Namespace: team-b
    ├── ResourceQuota    (pods: 4, cpu: 1-2 cores, mem: 1-2Gi)
    ├── Role + RoleBinding → team-b-user
    ├── Deployment: nginx-app (2 replicas)
    └── Service: nginx-service (ClusterIP :80)
```

## License

MIT
