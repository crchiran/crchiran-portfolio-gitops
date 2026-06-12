## CI/CD Deployment Flow Demonstration

This repository demonstrates a complete GitOps-based deployment workflow using:

* GitHub Actions
* GitHub Container Registry (GHCR)
* FluxCD
* Kubernetes
* GitOps Repository

### Deployment Architecture

```text
Developer
    │
    │ Git Push
    ▼
Application Repository
(crchiran-portfolio)
    │
    │ GitHub Actions
    ▼
Build Docker Image
    │
    ▼
Push Image to GHCR
    │
    ▼
Update GitOps Repository
(crchiran-portfolio-gitops)
    │
    ▼
FluxCD Detects Changes
    │
    ▼
Kubernetes Cluster
    │
    ▼
Deploy New Application Version
```
---

## FluxCD Deployment

### 1. Prerequisites

Install and configure:

* Kubernetes cluster
* `kubectl`
* Flux CLI
* GitHub account
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

## 2. Install Flux CLI

### macOS

```bash
brew install fluxcd/tap/flux
```

### Linux

```bash
curl -s https://fluxcd.io/install.sh | sudo bash
```

---

## 3. Create GHCR Image Pull Secret

Create namespace first:

```bash
kubectl create namespace portfolio --dry-run=client -o yaml | kubectl apply -f -
```

Create GHCR pull secret:

```bash
kubectl create secret docker-registry ghcr-secret \
  --namespace portfolio \
  --docker-server=ghcr.io \
  --docker-username=<github-username> \
  --docker-password='<github-token-or-ghcr-token>' \
  --docker-email='<github-email>'
```

Verify secret:

```bash
kubectl get secret ghcr-secret -n portfolio
```

> Never commit real tokens, passwords, or secrets into Git.

---

## 4. Export GitHub Token

```bash
export GITHUB_TOKEN="YOUR_GITHUB_TOKEN"
export GITHUB_USER="YOUR_GITHUB_USERNAME"
export GTIOPS_REPO="YOUR_GITOPS_REPO"
```

Verify token exists:

```bash
echo $GITHUB_TOKEN
echo $GITHUB_USER
echo $GTIOPS_REPO
```

---

## 5. Bootstrap FluxCD

```bash
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=$GTIOPS_REPO\
  --branch=main \
  --path=clusters//production \
  --personal=true
```

This command installs FluxCD controllers and connects the Kubernetes cluster with the GitHub GitOps repository.

---

## 6. Repository Structure

```text
.
├── apps
│   ├── crchiran-portfolio
│   │   ├── cert.yaml
│   │   ├── deployment.yaml
│   │   ├── gw.yaml
│   │   ├── kustomization.yaml
│   │   ├── sa.yaml
│   │   ├── svc.yaml
│   │   └── vs.yaml
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
│   └── todo
│       ├── cert.yaml
│       ├── gw.yaml
│       ├── kustomization.yaml
│       ├── ns.yaml
│       ├── svc.yaml
│       ├── todo-deployment.yaml
│       └── vs.yaml
├── clusters
│   └── production
│       ├── crchiran-portfolio-prod.yaml
│       ├── flux-system
│       │   ├── gotk-components.yaml
│       │   ├── gotk-sync.yaml
│       │   └── kustomization.yaml
│       ├── monitoring.yaml
│       └── todo.yaml
└── README.md
```

---

## 7. Verify FluxCD Installation

Check Flux controllers:

```bash
kubectl get pods -n flux-system
```

Check Flux health:

```bash
flux check
```

Check all Flux resources:

```bash
flux get all -A
```

Check Git source:

```bash
flux get sources git -A
```

Check Kustomizations:

```bash
flux get kustomizations -A
```

---

## 8. Reconcile Manually

Reconcile Git source:

```bash
flux reconcile source git flux-system
```

Reconcile root Kustomization:

```bash
flux reconcile kustomization flux-system
```

---

## 9. Verify Application Deployment

Check all namespaces:

```bash
kubectl get ns
```

Check all pods:

```bash
kubectl get pods -A
```

Check services:

```bash
kubectl get svc -A
```

Check deployments:

```bash
kubectl get deploy -A
```

Check Istio gateways:

```bash
kubectl get gateway -A
```

Check Istio VirtualServices:

```bash
kubectl get virtualservice -A
```

Check certificates:

```bash
kubectl get certificate -A
```

---

## 10. Verify Portfolio Application

```bash
kubectl get pods -n portfolio
kubectl get svc -n portfolio
kubectl get gateway -n portfolio
kubectl get virtualservice -n portfolio
kubectl get certificate -n portfolio
```

Describe failed pod if needed:

```bash
kubectl describe pod -n portfolio <pod-name>
```

Check logs:

```bash
kubectl logs -n portfolio <pod-name>
```

---

## 11. Verify Todo Application

```bash
kubectl get pods -n todo
kubectl get svc -n todo
kubectl get gateway -n todo
kubectl get virtualservice -n todo
kubectl get certificate -n todo
```

---

## 12. Verify Monitoring Stack

```bash
kubectl get pods -n monitoring
kubectl get helmrelease -n monitoring
kubectl get helmrepository -n monitoring
kubectl get svc -n monitoring
```

Check HelmRelease status:

```bash
flux get helmreleases -A
```

Reconcile monitoring manually:

```bash
flux reconcile kustomization monitoring -n flux-system
```

---

## 13. Update and Deploy Changes

Edit Kubernetes YAML files, then commit and push:

```bash
git status
git add .
git commit -m "Update Kubernetes manifests"
git push origin main
```

FluxCD automatically detects and applies the changes.

Force sync manually:

```bash
flux reconcile source git flux-system
flux reconcile kustomization flux-system
```

---

## 14. Troubleshooting

Check Flux events:

```bash
flux events
```

Check Flux logs:

```bash
kubectl logs -n flux-system deployment/source-controller
kubectl logs -n flux-system deployment/kustomize-controller
kubectl logs -n flux-system deployment/helm-controller
kubectl logs -n flux-system deployment/notification-controller
```

Check failed Kustomization:

```bash
flux get kustomizations -A
```

Describe Kustomization:

```bash
kubectl describe kustomization <name> -n flux-system
```

Check image pull issue:

```bash
kubectl describe pod -n portfolio <pod-name>
kubectl get secret ghcr-secret -n portfolio
```

Common image pull error:

```text
ImagePullBackOff
ErrImagePull
unauthorized
```

Fix by recreating GHCR secret:

```bash
kubectl delete secret ghcr-secret -n portfolio

kubectl create secret docker-registry ghcr-secret \
  --namespace portfolio \
  --docker-server=ghcr.io \
  --docker-username=<github-username> \
  --docker-password='<github-token-or-ghcr-token>' \
  --docker-email='<github-email>'
```

---

## 15. Important Notes

Do not manually deploy application manifests with:

```bash
kubectl apply -f apps/
```

FluxCD should control the deployment lifecycle.

Correct workflow:

```text
Edit YAML → Commit → Push → FluxCD Sync → Kubernetes Deploy
```

Git is the source of truth.

---

## 16. Disaster Recovery

To restore this project on a new cluster:

```bash
kubectl get nodes
export GITHUB_TOKEN=<github-personal-access-token>

flux bootstrap github \
  --owner=<github-username> \
  --repository=<gitops-repository-name> \
  --branch=main \
  --path=clusters/production \
  --personal=true \
  --token-auth \
  --network-policy=false
```

FluxCD will restore all resources defined under:

```text
clusters/production
```

---

## Maintainer

**Chandan Richard Chiran**

GitHub:

```text
https://github.com/crchiran
```
