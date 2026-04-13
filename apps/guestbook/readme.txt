# Argo CD + Minikube GitOps Demo (guestbook)

This repo is a simple GitOps demo using **Argo CD** running on **Minikube**.  
Argo CD watches this Git repository and syncs Kubernetes YAML from `apps/guestbook/` into the cluster.

## 1) Prerequisites (Mac)
- `minikube`
- `kubectl`
- `git`
- (recommended) `argocd` CLI

Install CLIs:
```bash
brew install argocd helm
```

Start Minikube:
```bash
minikube start
kubectl get nodes
```

## 2) Install Argo CD into Minikube
Create namespace:
```bash
kubectl create namespace argocd
```

Install Argo CD:
```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Wait for Argo CD pods:
```bash
kubectl get pods -n argocd
kubectl wait --for=condition=Ready pods --all -n argocd --timeout=300s
```

## 3) Access Argo CD UI locally
Port-forward Argo CD API server (keep this terminal running):
```bash
kubectl port-forward svc/argocd-server -n argocd 8484:443
```

Open:
- https://localhost:8484

> Browser may warn about a self-signed certificate; proceed.

## 4) Login to Argo CD
Get initial admin password:
```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode; echo
```

(Optional) Login with CLI:
```bash
argocd login localhost:8484 --username admin --password <PASTE_PASSWORD> --insecure
```

## 5) Add the guestbook Kubernetes manifests to this repo
Kubernetes YAML lives here:
- `apps/guestbook/deployment.yaml`
- `apps/guestbook/service.yaml`

Commit and push:
```bash
git add apps/guestbook
git commit -m "Add guestbook deployment and service"
git push
```

## 6) Create the Argo CD Application (using UI)
In Argo CD UI → **NEW APP**:

**General**
- Application Name: `guestbook`
- Project: `default`

**Source**
- Repository URL: `https://github.com/dineshsala/argocd-demo.git`
- Revision: `HEAD` (or `main`)
- Path: `apps/guestbook`

**Destination**
- Cluster URL: `https://kubernetes.default.svc`
- Namespace: `default`

Create the app.

## 7) Sync and observe GitOps behavior
- After pushing changes to Git, Argo CD should show **OutOfSync**.
- Click **Sync** to apply Git state to the cluster.
- When done, Argo CD should show **Synced** and **Healthy**.

## 8) Verify resources in Kubernetes (Minikube)
```bash
kubectl get deploy,rs,pods,svc -n default
kubectl get pods -n default -l app=guestbook
```

## 9) Access the guestbook service locally
Port-forward the service:
```bash
kubectl port-forward svc/guestbook 8081:80 -n default
```

Open:
- http://localhost:8081

## What this demo teaches (GitOps)
- **Git is the source of truth** for Kubernetes manifests.
- Argo CD continuously compares:
  - desired state (Git) vs actual state (cluster)
- When different, Argo CD shows **OutOfSync**.
- Sync (manual or auto-sync) makes the cluster match Git again.
