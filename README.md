# Cluster Apps

This repository contains **cluster-level applications and platform components** deployed via ArgoCD using a GitOps model.

It is responsible for bootstrapping and managing core Kubernetes services such as:

- Ingress controller (Traefik)
- Monitoring stack (Prometheus)
- Storage provisioning (TopolVM)
- Database operators (PGO)
- Alerting rules
- ArgoCD itself and app-of-apps structure

This is the **foundation layer of the cluster**, not application workloads.

---

## What this repo does

This repository is the **cluster control plane for GitOps apps**.

It defines and manages:

- Cluster add-ons
- Platform services
- Observability stack
- Networking layer
- Storage layer
- Database operators

Everything is deployed and reconciled automatically by ArgoCD.

---

## App-of-Apps Model

This repo uses the **ArgoCD App-of-Apps pattern**.

### Bootstrap flow:

```
argocd/
  ↓
app-of-apps/
  ↓
apps/
  ↓
actual cluster services
```

---

### `app-of-apps/`

This is the entry point for ArgoCD.

It defines two main roots:

- `cluster-apps.yaml` → core platform components
- `stage-apps.yaml` → stage environment bootstrap

This allows ArgoCD to recursively manage all applications from a single entry point.

---

## Apps Layer

```
apps/
```

This directory contains ArgoCD Application manifests for each platform component.

Examples:

- `kube-prometheus-stack.yaml` → monitoring stack
- `traefik.yaml` → ingress controller
- `topolvm.yaml` → storage CSI driver
- `pgo.yaml` → PostgreSQL operator
- `alerting.yaml` → alert rules

Each file defines:

- Source Helm chart
- Values override
- Target namespace
- Sync policies

---

## Platform Components

### Monitoring

```
kube-prometheus-stack/
```

Deploys the full observability stack:

- Prometheus
- Grafana
- Alertmanager
- Node exporters

Includes custom values tuned for the cluster.

---

### Alerting

```
alerting/
```

Contains PrometheusRule definitions for cluster-wide alerts.

Used for:

- Node health
- Resource pressure
- Service availability

---

### Ingress

```
traefik/
```

Manages cluster ingress via Traefik:

- HTTP routing
- TLS termination
- Ingress controller configuration

---

### Storage

```
topolvm/
```

CSI-based storage provisioning layer.

Provides:

- Dynamic volume provisioning
- LVM-backed storage
- PVC automation

---

### Database Operator

```
pgo/
```

Deploys PostgreSQL operator (PGO):

- PostgreSQL cluster lifecycle management
- Backup/restore automation
- HA database provisioning

---

## ArgoCD Installation

```
argocd/
```

This folder contains the ArgoCD installation itself using Helm.

Includes:

- ArgoCD Helm chart
- Bootstrap templates
- Initial app registration

This is what enables the GitOps loop for the cluster.

---

## How it works

1. ArgoCD is installed from `argocd/`
2. It loads `app-of-apps/`
3. App-of-apps registers all platform services from `apps/`
4. Each app pulls its Helm chart + values
5. Cluster state continuously reconciles to Git

---

## Key design principles

- Everything is declarative (Git is the source of truth)
- Platform services are separated from application workloads
- App-of-apps ensures single bootstrap entry point
- Helm is used for all installable components
- Each system component is isolated per directory

---

## What this repo is NOT

This repository does **not** contain:

- Business applications (APIs, web services, cronjobs)
- Application-level Helm charts
- Terraform or infrastructure provisioning
- VM or cluster creation logic

Those live in separate repositories.

---

## Deployment flow

```
Git push → ArgoCD → Helm charts → Kubernetes cluster
```

No manual kubectl-based installs are expected after bootstrap.

---

## Typical usage

### Bootstrap cluster

Apply only the root ArgoCD application:

```
helm dependency build
helm install argocd ./argocd \
  -n argocd --create-namespace \
  -f ./argocd/values.yaml
```

This initializes everything else automatically.

---

## Notes

- This repo represents the **platform layer of the cluster**
- Designed for reproducible cluster bootstrapping
- Fully GitOps-driven via ArgoCD
