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

---

# Phase B: Running the app on Amazon EKS

Epic: KUBE-12 (tasks KUBE-13 to KUBE-22). Region: ap-south-1 (Mumbai).

## What I built
- EKS cluster `devops-eks` (Kubernetes 1.36) created with eksctl
- Managed node group `devops-eks-nodegroup` (t3.small, min 1, max 2, later max 3)
- nginx Deployment (2 replicas) in namespace `dev`
- Service of type LoadBalancer, reachable from the internet ("Welcome to nginx!")
- Scaled nodes from 1 to 2 and added an HPA (2 to 10 pods at 70% CPU)
- Deleted everything and verified no leftover resources

## Steps and notes
| Step | Topic | Notes | Jira |
|------|-------|-------|------|
| 1 | AWS access (IAM user, CLI, keys) | phase-b/01-aws-access.md | KUBE-13 |
| 2 | Install eksctl and verify tools | phase-b/02-install-eksctl.md | KUBE-14 |
| 3 | Create EC2 key pair for the node group | phase-b/03-ec2-key-pair.md | KUBE-15 |
| 4 | Create the EKS cluster | phase-b/04-create-eks-control-plane.md | KUBE-16 |
| 5 | Create the node group | phase-b/05-add-node-group.md | KUBE-17 |
| 6 | Connect kubectl and check the cluster | phase-b/06-connect-kubectl.md | KUBE-18 |
| 7 | Review in AWS console | phase-b/07-console-review.md | KUBE-19 |
| 8 | Deploy the app with a LoadBalancer | phase-b/08-deploy-app-on-eks.md | KUBE-20 |
| 9 | Scaling (nodes and HPA) | phase-b/09-scaling.md | KUBE-21 |
| 10 | Cleanup | phase-b/10-cleanup.md | KUBE-22 |

## Key commands
    eksctl create cluster --name devops-eks --region ap-south-1 --zones ap-south-1a,ap-south-1b --version 1.36 --without-nodegroup
    eksctl create nodegroup --cluster devops-eks --region ap-south-1 --name devops-eks-nodegroup --node-type t3.small --managed --nodes 1 --nodes-min 1 --nodes-max 2
    aws eks update-kubeconfig --region ap-south-1 --name devops-eks
    kubectl apply -f deployment.yaml -f service-loadbalancer.yaml
    eksctl scale nodegroup --cluster devops-eks --name devops-eks-nodegroup --nodes 2 --nodes-min 1 --nodes-max 3 --region ap-south-1
    kubectl autoscale deploy web --cpu-percent=70 --min=2 --max=10 -n dev
    eksctl delete cluster --name devops-eks --region ap-south-1 --disable-nodegroup-eviction

## Problems and fixes
- kubectl 1.37 vs cluster version: kubectl works within one minor version, so I chose cluster 1.36.
- Console Nodes tab said "Unauthorized": the console user had no access entry. Added one with AmazonEKSAdminViewPolicy. kubectl was never affected.
- `eksctl delete nodegroup --approve` failed: --approve needs a config file. Removed it.
- Node drain timed out twice during cleanup (system pods could not be evicted). Fixed with `--disable-nodegroup-eviction` on delete cluster.

## What I learned
- EKS: the control plane is managed by AWS. I manage worker nodes (managed node group), and eksctl builds the VPC, subnets, NAT gateway, and IAM roles through CloudFormation.
- A Service of type LoadBalancer creates an AWS load balancer. Delete the Service before deleting the cluster.
- HPA scales pods and needs metrics-server and CPU requests. A node autoscaler scales nodes.
- The EKS control plane, NAT gateway, nodes, and load balancer all charge hourly. Delete the same day.
- Security: use an IAM user (not root) for the CLI, keep keys only in ~/.aws/credentials, keep .pem files out of Git, rotate any exposed key.

## Cleanup
All Phase B resources were deleted and checked in the console (EC2, load balancers, volumes, Elastic IPs, NAT gateways, VPC, CloudFormation, EKS). Nothing is left running.
