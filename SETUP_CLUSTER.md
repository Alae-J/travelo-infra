# Travelo App - Cluster Setup Guide

## 1. Check Current State

```bash
minikube status
kubectl get namespace argocd
kubectl get pods -n travelo
```

## 2. Start Minikube

```bash
# If already running and want fresh start:
minikube delete

# Start with proper resources
minikube start --memory 8192 --cpus 4
minikube addons enable ingress
```

## 3. Install Argo CD

```bash
# Create namespace
kubectl create namespace argocd

# Install Argo CD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for ready (takes ~2 minutes)
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=argocd-server -n argocd --timeout=300s
```

## 4. Access Argo CD UI (Optional)

```bash
# Get admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo

# Port forward
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Visit: https://localhost:8080
# Username: admin
# Password: (from above)
```

## 5. Deploy Travelo Application

```bash
cd /home/xichrome/Desktop/ine2/P2/Devops/travelo-app/travelo-infra

# Apply Argo CD application
kubectl apply -f argocd-application.yaml

# Create namespace (if needed)
kubectl create namespace travelo 2>/dev/null || true

# Create MySQL secret (if not exists)
kubectl create secret generic mysql-secret \
  --from-literal=mysql-root-password=yassine2004 \
  -n travelo 2>/dev/null || echo "Secret already exists"
```

## 6. Watch Deployment

```bash
# Watch Argo CD application sync
kubectl get applications -n argocd -w

# Watch pods coming up
kubectl get pods -n travelo -w
```

## 7. Access Application

**Recommended method** - Port forward (simple, always works):

```bash
# Forward frontend service directly (run in background)
kubectl port-forward svc/frontend -n travelo 8080:80 &

# Visit in browser: http://localhost:8080
```

**Stop port-forward later**:
```bash
# Find the process
ps aux | grep port-forward

# Kill it
kill <PID>
```

**Why not use ingress/tunnel?**
- The ingress has `host: localhost` restriction in git
- Argo CD auto-syncs from git, reverting manual changes
- Minikube tunnel doesn't route properly in this setup
- Port-forward bypasses all that complexity

## 8. Verification Commands

```bash
# Check all resources
kubectl get all -n travelo

# Check Argo CD sync status
kubectl get applications -n argocd

# Check logs
kubectl logs -n travelo deployment/frontend-deployment
kubectl logs -n travelo deployment/backend-deployment
kubectl logs -n travelo statefulset/mysql
```

## Troubleshooting

**Pods not starting**:
```bash
kubectl describe pod <pod-name> -n travelo
```

**Argo CD not syncing**:
```bash
kubectl describe application travelo-app -n argocd
```

**Reset MySQL secret**:
```bash
kubectl delete secret mysql-secret -n travelo
kubectl create secret generic mysql-secret --from-literal=mysql-root-password=yassine2004 -n travelo
```

## Clean Up

```bash
# Delete application
kubectl delete -f argocd-application.yaml
kubectl delete namespace travelo

# Stop minikube
minikube stop

# Delete minikube
minikube delete
```
