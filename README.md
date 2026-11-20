# Kubernetes Basics Lab on Minikube (Phase A)

Small hands-on project to learn Kubernetes basics before moving to Amazon EKS.
I ran a simple nginx app on a local Minikube cluster using a Deployment and a Service.

## What this project shows
- How to set up a local Kubernetes cluster with Minikube
- How to use namespaces to separate workloads
- How to write a Deployment and a Service file
- How Kubernetes self-heals and scales pods
- How to check and debug pods with kubectl

## Tools
- Windows with WSL2 Ubuntu
- Docker, kubectl, Minikube
- Image: nginx:1.27

## Files
| File | Purpose |
|------|---------|
| deployment.yaml | Runs 2 nginx pods with resource limits and a readiness probe |
| service.yaml | ClusterIP service that gives the pods one stable address |

## Architecture (short)
- Control plane: API server, etcd, scheduler, controller manager
- Worker node: kubelet, kube-proxy, container runtime, pods
- Flow: kubectl -> API server -> etcd -> scheduler -> kubelet runs the pod -> controller keeps the count

## Steps

### 1. Install the tools
```
sudo apt update && sudo apt install -y docker.io
sudo usermod -aG docker $USER && newgrp docker
sudo service docker start
# kubectl and minikube: install from the official download pages
docker --version
kubectl version --client
minikube version
```

### 2. Start the cluster
```
minikube start --driver=docker
```

### 3. Check the cluster
```
kubectl get node
kubectl get pods -n kube-system
```
Result: one node `minikube` in Ready state. The kube-system pods show the control plane parts.

### 4. Create a namespace
```
kubectl create namespace dev
```

### 5. List namespaces
```
kubectl get ns
```

### 6. Create the deployment file
See `deployment.yaml`. It has 2 replicas, CPU and memory requests and limits, and a readiness probe.

### 7. Create the service file
See `service.yaml`. Type is ClusterIP on port 80.

### 8. Apply both files
```
kubectl apply -f deployment.yaml -f service.yaml
kubectl get pods -n dev
```
Result: 2 pods in Running state.

### 9. Check the service and deployment
```
kubectl get svc -n dev
kubectl get deploy -n dev
```

### 10. Test self-healing
```
kubectl delete pod <pod-name> -n dev
kubectl get pods -n dev
```
Result: a new pod was created within seconds. The count stayed at 2.

### 11. Scale
```
kubectl scale deploy web --replicas=4 -n dev
kubectl get pods -n dev
```
Result: 4 pods running. I then scaled back to 2.

### 12. Debug
```
kubectl describe pod <pod-name> -n dev
kubectl logs <pod-name> -n dev
```
Result: Events show Scheduled, Pulled, Created, Started. Logs show the readiness probe calling `/` every 10 seconds with status 200.

### 13. Test the app
```
kubectl port-forward svc/web 8080:80 -n dev &
curl -s localhost:8080 | head -n 5
kill %1
```
Result: the page "Welcome to nginx!" is returned through the Service.

## Problems I hit and fixes
| Problem | Cause | Fix |
|---------|-------|-----|
| `connection to the server 127.0.0.1 was refused` | Minikube had stopped after the terminal closed | `sudo service docker start` then `minikube start --driver=docker` |
| `namespace "dev" already exists` | The namespace was already created | No fix needed |
| Readiness probe failed warning in events | Pod was starting after a restart | Harmless. Pod became Ready |

## Clean up
```
kubectl delete -f deployment.yaml -f service.yaml
minikube stop
```
To remove the cluster fully: `minikube delete`

## What I learned
- Docker builds and runs containers. Kubernetes manages them.
- A Deployment keeps the right number of pods and handles updates.
- A Service gives pods one stable address because pod IPs change.
- Requests are used for scheduling. Limits cap usage.
- A readiness probe keeps traffic away from a pod until it is ready.

## Next
Phase B: create an Amazon EKS cluster with eksctl, add a node group, deploy the same app, then delete everything. After that, write the same cluster in Terraform.
