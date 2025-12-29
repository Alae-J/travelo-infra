Kubernetes manifests for Travelo (namespace: travelo)

Apply steps (Minikube)

1. Start minikube (example):
```
minikube start --memory 8192 --cpus 4
minikube addons enable ingress
```

2. Create namespace:
```
kubectl apply -f k8s/namespace.yaml
```

3. Create MySQL secret (recommended) or rely on the given stringData in the manifest:
```
kubectl create secret generic mysql-secret --from-literal=mysql-root-password=yassine2004 -n travelo
```

4. Deploy DB, backend and frontend:
```
kubectl apply -f k8s/mysql.yaml -n travelo
kubectl apply -f k8s/backend.yaml -n travelo
kubectl apply -f k8s/frontend.yaml -n travelo
kubectl apply -f k8s/ingress.yaml -n travelo
```

5. Add local hosts entry (replace IP with `minikube ip`):
```
# on Windows edit C:\Windows\System32\drivers\etc\hosts as admin
<MINIKUBE_IP> travelo.local
```

6. Verify:
```
kubectl get all -n travelo
kubectl get pvc -n travelo
kubectl logs deploy/backend-deployment -n travelo
```

Notes and caveats:
- The MySQL StatefulSet uses a PVC per replica. Running multiple MySQL replicas requires replication setup (not covered here); for simple setups use replica: 1.
- The backend expects an actuator health endpoint at `/actuator/health`. If your app doesn't expose it, change readiness/liveness probes to a suitable path.
