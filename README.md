# Hiring System GitOps Repository

This repository contains Helm charts for the Hiring System, managed by ArgoCD.

## Structure

Each service has its own Helm chart:

```
hiring-gitops/
├── gateway/
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
│       ├── deployment.yaml
│       ├── service.yaml
│       └── configmap.yaml
├── identity/
├── workflow-engine/
├── interview-service/
├── assessment-service/
├── notification-service/
├── platform-service/
├── proctoring-service/
├── scoring-service/
├── candidate-portal/
├── recruiter-portal/
├── admin-console/
├── superadmin-console/
└── ingress/
    └── templates/
        └── *.yaml (HTTPProxy configs)
```

## GitOps Workflow

1. **CI/CD Pipeline** (application repo):
   - Builds Docker images
   - Pushes images to Artifact Registry
   - Updates `imageTag` in values.yaml files in this repo
   - Commits and pushes changes

2. **ArgoCD**:
   - Watches this repository
   - Detects changes to Helm charts
   - Syncs changes to Kubernetes cluster

## ArgoCD Applications

Create one ArgoCD Application per service:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: hiring-gateway
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/<org>/hiring-gitops.git
    targetRevision: main
    path: gateway
  destination:
    server: https://kubernetes.default.svc
    namespace: hiring-system
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

## Image Tag Updates

The CI/CD pipeline updates image tags in `values.yaml`:

```yaml
service:
  imageTag: abc123def  # Updated by CI/CD to commit SHA
```

## Deployment Process

1. Push code to `deploy` branch in application repo
2. GitHub Actions builds and pushes images
3. GitHub Actions updates `imageTag` in this repo
4. ArgoCD detects change and syncs to cluster

## Rollback

```bash
# Find working commit
git log --oneline gateway/values.yaml

# Revert to that commit
git revert <commit-hash>
git push origin main

# ArgoCD will sync the rollback
```

## Security

- This repo contains production credentials in ConfigMaps
- Should be **private**
- Access control:
  - CI/CD service account: write
  - DevOps team: write
  - ArgoCD: read
