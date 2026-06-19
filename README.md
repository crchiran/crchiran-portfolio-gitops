# Portfolio Platform - Kubernetes GitOps Repository

![Kubernetes](https://img.shields.io/badge/Kubernetes-GitOps-blue)
![FluxCD](https://img.shields.io/badge/FluxCD-Continuous%20Delivery-blue)
![Istio](https://img.shields.io/badge/Istio-Service%20Mesh-green)
![Prometheus](https://img.shields.io/badge/Prometheus-Monitoring-orange)
![Grafana](https://img.shields.io/badge/Grafana-Dashboards-orange)
![Loki](https://img.shields.io/badge/Loki-Logging-yellow)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-CI%2FCD-black)

A production-inspired Kubernetes GitOps platform demonstrating automated application delivery using FluxCD, Istio, cert-manager, Prometheus, Grafana, Loki, and GitHub Actions.

This repository acts as the **single source of truth** for Kubernetes deployments and platform configuration. Every change is managed through Git and automatically synchronized into the cluster by FluxCD.

---

## Project Summary

| Component         | Purpose                       |
| ----------------- | ----------------------------- |
| Portfolio Website | Static application deployment |
| Todo Application  | Example workload              |
| FluxCD            | GitOps continuous delivery    |
| Istio             | Traffic management & ingress  |
| cert-manager      | TLS certificate automation    |
| Prometheus        | Metrics collection            |
| Grafana           | Dashboard visualization       |
| Loki              | Centralized logging           |
| GitHub Actions    | CI/CD automation              |
| GHCR              | Container registry            |

---

## Why GitOps?

GitOps uses Git as the single source of truth for Kubernetes deployments.

Benefits:

* Version-controlled infrastructure
* Auditable deployment history
* Automated synchronization
* Faster recovery and rollback
* Consistent environments
* Reduced operational risk
* Improved deployment reliability
* Fully declarative platform management

---

## Platform Features

* GitOps-driven deployments
* Automated CI/CD pipelines
* Automated container image publishing
* Automated Kubernetes synchronization
* Container vulnerability scanning
* TLS certificate automation
* Service mesh ingress management
* Centralized logging
* Metrics and dashboards
* Declarative infrastructure management
* Disaster recovery through Git

---

## Repository Relationships

This project uses two repositories.

### Application Repository

Repository:

```text
https://github.com/crchiran/crchiran-portfolio
```

Responsibilities:

* Application source code
* Docker image creation
* GitHub Actions CI/CD
* Trivy security scanning
* Publishing images to GHCR
* Updating GitOps manifests

---

### GitOps Repository (Current Repository)

Repository:

```text
https://github.com/crchiran/crchiran-portfolio-gitops
```

Responsibilities:

* Kubernetes manifests
* FluxCD resources
* Istio resources
* TLS certificates
* Monitoring stack
* Deployment automation

---

## What Happens When Code Is Pushed?

```text
Developer
    │
    │ Git Push
    ▼
Application Repository
(crchiran-portfolio)
    │
    ▼
GitHub Actions
    │
    ├── Build Docker Image
    ├── Trivy Security Scan
    ├── Push Image to GHCR
    └── Update GitOps Repository
                │
                ▼
GitOps Repository
(crchiran-portfolio-gitops)
                │
                ▼
FluxCD
                │
                ▼
Kubernetes Cluster
                │
                ▼
Deploy New Application Version
```

---

## Platform Architecture

```text
Internet
    │
    ▼
Istio Ingress Gateway
    │
    ├── Portfolio Website
    └── Todo Application

Monitoring Stack
    ├── Prometheus
    ├── Grafana
    ├── Loki
    └── Promtail

FluxCD
    ├── Source Controller
    ├── Kustomize Controller
    ├── Helm Controller
    └── Notification Controller

cert-manager
    └── TLS Certificate Automation
```

---

## Repository Structure

```text
.
├── README.md
├── apps
│   ├── crchiran-portfolio
│   │   ├── cert.yaml
│   │   ├── deployment.yaml
│   │   ├── gw.yaml
│   │   ├── kustomization.yaml
│   │   ├── sa.yaml
│   │   ├── svc.yaml
│   │   └── vs.yaml
│   │
│   ├── monitoring
│   │   ├── grafana-admin-secret.yaml
│   │   ├── grafana-virtualservice.yaml
│   │   ├── kube-prom-stack-helmrelease.yaml
│   │   ├── kustomization.yaml
│   │   ├── loki-helmrelease.yaml
│   │   ├── loki-memberlist-svc.yaml
│   │   ├── loki-repo.yaml
│   │   ├── monitoring-gateway.yaml
│   │   ├── monitoring-repo.yaml
│   │   ├── namespace.yaml
│   │   └── promtail-helmrelease.yaml
│   │
│   └── todo
│       ├── cert.yaml
│       ├── gw.yaml
│       ├── kustomization.yaml
│       ├── ns.yaml
│       ├── svc.yaml
│       ├── todo-deployment.yaml
│       └── vs.yaml
│
└── clusters
    └── production
        ├── crchiran-portfolio-prod.yaml
        ├── monitoring.yaml
        ├── todo.yaml
        └── kustomization.yaml
```

---

## Managed Applications

### Portfolio Website

Location:

```text
apps/crchiran-portfolio
```

Resources:

* Deployment
* Service
* Istio Gateway
* VirtualService
* TLS Certificate
* ServiceAccount

---

### Todo Application

Location:

```text
apps/todo
```

Resources:

* Deployment
* Service
* Istio Gateway
* VirtualService
* TLS Certificate

---

### Monitoring Stack

Location:

```text
apps/monitoring
```

Components:

* Prometheus
* Grafana
* Loki
* Promtail

Provides:

* Metrics collection
* Dashboard visualization
* Centralized logging
* Kubernetes observability

---

## Production Environment

Location:

```text
clusters/production
```

Contains:

```text
crchiran-portfolio-prod.yaml
monitoring.yaml
todo.yaml
kustomization.yaml
```

Responsibilities:

* Environment composition
* Application orchestration
* Deployment ordering
* FluxCD synchronization

---

## Getting Started

### Prerequisites

Install and configure:

* Kubernetes Cluster
* kubectl
* Flux CLI
* GitHub Account
* GitHub Personal Access Token

Verify cluster access:

```bash
kubectl get nodes
```

Verify Flux CLI:

```bash
flux --version
flux check --pre
```

---

## Bootstrap FluxCD

### Export Variables

```bash
export GITHUB_TOKEN="<github-personal-access-token>"
export GITHUB_USER="<github-username>"
export GITOPS_REPO="<gitops-repository-name>"
```

Verify:

```bash
echo $GITHUB_TOKEN
echo $GITHUB_USER
echo $GITOPS_REPO
```

---

### Bootstrap Kubernetes Cluster

```bash
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=$GITOPS_REPO \
  --branch=main \
  --path=clusters/production \
  --personal=true \
  --token-auth
```

FluxCD will automatically:

* Install FluxCD controllers
* Create the `flux-system` namespace
* Configure Git synchronization
* Create GitRepository resources
* Create Kustomization resources
* Start continuous reconciliation

---

## Verification

### Verify FluxCD Controllers

```bash
kubectl get pods -n flux-system
```

---

### Verify Flux Health

```bash
flux check
```

---

### View Flux Resources

```bash
flux get all -A
```

---

### Verify Git Sources

```bash
flux get sources git -A
```

---

### Verify Kustomizations

```bash
flux get kustomizations -A
```

---

### Verify Workloads

```bash
kubectl get deploy -A
kubectl get pods -A
kubectl get svc -A
```

---

## Manual Reconciliation

Force Git synchronization:

```bash
flux reconcile source git flux-system
```

Force Kustomization reconciliation:

```bash
flux reconcile kustomization flux-system
```

---

## Troubleshooting

### View Flux Events

```bash
flux events
```

---

### View Controller Logs

```bash
kubectl logs -n flux-system deployment/source-controller
kubectl logs -n flux-system deployment/kustomize-controller
kubectl logs -n flux-system deployment/helm-controller
kubectl logs -n flux-system deployment/notification-controller
```

---

### Verify Git Sources

```bash
flux get sources git -A
```

---

### Verify Kustomizations

```bash
flux get kustomizations -A
```

---

### Describe Failed Resource

```bash
kubectl describe kustomization <name> -n flux-system
```

---

## Disaster Recovery

One of the primary benefits of GitOps is rapid cluster recovery.

```text
Git Repository
      │
      ▼
FluxCD Bootstrap
      │
      ▼
Restore Cluster State
      │
      ▼
Applications
Monitoring
Networking
Certificates
```

Restore the platform on a new Kubernetes cluster:

```bash
export GITHUB_TOKEN="<github-personal-access-token>"
export GITHUB_USER="<github-username>"
export GITOPS_REPO="<gitops-repository-name>"
```

```bash
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=$GITOPS_REPO \
  --branch=main \
  --path=clusters/production \
  --personal=true \
  --token-auth
```

FluxCD automatically restores all resources defined in Git.

---

## Technology Stack

| Category     | Technologies                     |
| ------------ | -------------------------------- |
| GitOps       | FluxCD, Git                      |
| Platform     | Kubernetes                       |
| Service Mesh | Istio                            |
| Certificates | cert-manager                     |
| Monitoring   | Prometheus                       |
| Dashboards   | Grafana                          |
| Logging      | Loki, Promtail                   |
| CI/CD        | GitHub Actions                   |
| Registry     | GitHub Container Registry (GHCR) |

---

## Learning Outcomes

This project demonstrates:

* GitOps workflows
* FluxCD operations
* Kubernetes deployments
* Istio traffic management
* TLS certificate automation
* Monitoring and observability
* Centralized logging
* Declarative infrastructure
* Disaster recovery through Git
* Production-inspired platform operations

---

## Future Enhancements

Potential future improvements:

* Multi-environment GitOps
* Progressive delivery
* Canary deployments
* OpenTelemetry tracing
* Service Level Objectives (SLOs)
* Policy enforcement with Kyverno
* Runtime security with Falco
* Multi-cluster management
* Automated backup and recovery workflows

---

## License

This project is provided for educational and demonstration purposes.
