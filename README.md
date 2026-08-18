# k8s-gitops

Kubernetes manifests reconciled onto the [k8s-cluster](https://github.com/<your-username>/k8s-cluster) lab cluster by ArgoCD.

ArgoCD watches this repo via a bootstrap `Application` resource (created by the `gitops` Ansible role in k8s-cluster) pointing at:

- **repo:** this repository
- **path:** `apps`
- **branch:** `main`
- **destination namespace:** `default`
- **sync policy:** automated, with `prune` and `selfHeal` enabled — any change pushed here is applied automatically, and manual in-cluster drift is reverted.

## Layout

```
apps/
  sample-app/
    deployment.yaml   # nginx Deployment
    service.yaml      # ClusterIP Service
    ingress.yaml       # Ingress via the ingress-nginx controller already installed on the cluster
```

Everything under `apps/` is applied as-is by the root `Application` — add new manifests or subdirectories there to deploy more workloads.

## Setup

```bash
git init
git add .
git commit -m "Initial GitOps manifests"
gh repo create k8s-gitops --source=. --public --push
```

Then set `gitops_repo_url` in `k8s-cluster/ansible/group_vars/all.yml` to this repo's clone URL before running the `gitops` Ansible role.

## Verifying a sync

```bash
kubectl get application -n argocd
kubectl get deploy,svc,ingress -n default
```
