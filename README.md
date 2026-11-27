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

---

# Phase C: Kubernetes Resources and Argo CD on Amazon EKS

Region: ap-south-1 (Mumbai)
Jira Epic: KUBE-23

## Assignment
1. Set up my own Kubernetes cluster
2. Study Kubernetes architecture
3. Learn Namespace, Service, Pods, Deployment and ReplicaSet
4. Install Argo CD in Kubernetes

## What I built
- EKS cluster `devops-eks` (Kubernetes 1.36) created with eksctl
- Managed node group `devops-eks-nodegroup` with 2 t3.small nodes (min 1, max 2)
- `nginx-deployment` (nginx:1.27, 2 replicas) in namespace `dev`
- `nginx-service` of type ClusterIP
- Argo CD v3.6.3 in namespace `argocd`
- Deleted the cluster and checked AWS for leftovers

## Kubernetes architecture

### Control plane (managed by AWS in EKS)
| Part | Job |
|------|-----|
| API server | Receives all requests |
| etcd | Stores cluster data |
| Scheduler | Chooses a node for each pod |
| Controller manager | Keeps the actual state equal to the wanted state |

### Worker node
| Part | Job |
|------|-----|
| kubelet | Looks after the pods on the node |
| kube-proxy | Handles Service network rules |
| Container runtime | Runs the containers |

### Flow
1. kubectl sends the request to the API server.
2. The API server saves it in etcd.
3. Controllers create the pod records.
4. The scheduler chooses a node for each pod.
5. The kubelet asks the container runtime to start the containers.

## Kubernetes resources
| Resource | Meaning |
|----------|---------|
| Namespace | Separates resources inside a cluster. It does not secure them by itself |
| Pod | Smallest unit that is deployed. Holds one or more containers |
| ReplicaSet | Keeps the required number of pods running |
| Deployment | Manages ReplicaSets and app updates |
| Service | Gives a stable address to reach pods. Finds them using labels |

Flow: Deployment -> ReplicaSet -> Pods. The Service sends traffic to the Pods.

## Steps and notes
| Step | Topic | Notes | Jira |
|------|-------|-------|------|
| 1 | Prepare repository and plan | phase-c/01-plan.md | KUBE-24 |
| 2 | Verify AWS login | phase-c/02-aws-login.md | KUBE-25 |
| 3 | Verify tools | phase-c/03-tool-check.md | KUBE-26 |
| 4 | Create the EKS cluster | phase-c/04-eks-cluster.md | KUBE-27 |
| 5 | Add worker nodes | phase-c/05-node-group.md | KUBE-28 |
| 6 | Kubernetes resources and tests | phase-c/06-k8s-resources.md | KUBE-29 |
| 7 | Install Argo CD | phase-c/07-argocd-install.md | KUBE-30 |
| 8 | Access the Argo CD UI | phase-c/08-argocd-ui.md | KUBE-31 |
| 9 | Delete the cluster and check AWS | phase-c/09-cleanup.md | KUBE-32 |

## Step 1: Prepare repository and plan
Why: check the project folder before making changes.

```
cd ~/projects/k8s-microservices-eks
pwd
ls
git status
mkdir -p phase-c
```

Result: on branch main, working tree clean. Added the Phase C plan.

## Step 2: Verify AWS login
Why: check which AWS identity the CLI uses.

```
aws sts get-caller-identity
```

Result: successful, using the eks-admin IAM user.

## Step 3: Verify tools
| Command | Why | Output |
|---------|-----|--------|
| `aws --version` | Check AWS CLI | aws-cli/2.37.10 |
| `eksctl version` | Check eksctl | 0.232.0 |
| `kubectl version --client` | Check kubectl | v1.37.2 |
| `eksctl get cluster --region ap-south-1` | Check existing clusters | No clusters found |

## Step 4: Create the EKS cluster
Why: create the control plane first, without worker nodes.

```
eksctl create cluster --name devops-eks --region ap-south-1 --zones ap-south-1a,ap-south-1b --version 1.36 --without-nodegroup
eksctl get cluster --region ap-south-1
aws eks describe-cluster --name devops-eks --region ap-south-1 --query "cluster.[status,version]" --output text
```

| Option | Meaning |
|--------|---------|
| `--name` | Cluster name |
| `--region` | AWS region |
| `--zones` | Two Availability Zones |
| `--version` | Kubernetes version |
| `--without-nodegroup` | No worker nodes in this step |

Output:

```
EKS cluster "devops-eks" in "ap-south-1" region is ready
ACTIVE  1.36
```

- Took about 10 minutes
- CloudFormation stack: eksctl-devops-eks-cluster
- Add-ons: vpc-cni, kube-proxy, coredns, metrics-server
- Checked in the console: EKS, CloudFormation, VPC, IAM, EC2

## Step 5: Add worker nodes
Why: pods need worker nodes to run. A t3.small normally allows 11 pods, so I used 2 nodes.

```
eksctl create nodegroup --cluster devops-eks --region ap-south-1 --name devops-eks-nodegroup --node-type t3.small --managed --nodes 2 --nodes-min 1 --nodes-max 2
kubectl get nodes
kubectl get pods -n kube-system
```

| Option | Meaning |
|--------|---------|
| `--node-type t3.small` | 2 vCPU and 2 GB memory |
| `--managed` | EKS managed node group |
| `--nodes 2` | Start with 2 nodes |
| `--nodes-min 1 --nodes-max 2` | Size limits |

Output:
- 2 nodes Ready, version v1.36.5-eks-9adbfea
- 8 system pods Running with 0 restarts

| System pod | Count | Job |
|------------|-------|-----|
| aws-node | 2 | Pod networking in the VPC |
| coredns | 2 | DNS names inside the cluster |
| kube-proxy | 2 | Service network rules |
| metrics-server | 2 | CPU and memory metrics |

## Step 6: Kubernetes resources and tests

### Namespace
Why: keep the app resources separate.

```
kubectl create namespace dev
kubectl get namespaces
```

Output: namespace/dev created, status Active.

### Deployment
Why: run the app and manage its pods.

```
kubectl create deployment nginx-deployment --image=nginx:1.27 --replicas=2 --port=80 -n dev
kubectl rollout status deployment/nginx-deployment -n dev
kubectl get deployment,replicaset,pods -n dev -o wide
```

| Option | Meaning |
|--------|---------|
| `--image` | Container image |
| `--replicas=2` | Run 2 pods |
| `--port=80` | Records the container port. It does not expose the app |
| `-n dev` | Namespace |
| `-o wide` | Show more details, such as pod IP and node |

Output: successfully rolled out. Deployment 2/2, one ReplicaSet, 2 pods Running on 2 different nodes.

### Service
Why: pod IPs can change. A Service gives a stable address.

```
kubectl expose deployment nginx-deployment --name=nginx-service --type=ClusterIP --port=80 --target-port=80 -n dev
kubectl get service -n dev
kubectl get endpointslices -n dev
```

- port: the Service port
- targetPort: the app port in the pod
- ClusterIP: access inside the cluster only

Output: nginx-service, ClusterIP, 80/TCP. The EndpointSlice showed 2 pod IPs.

### Test from inside the cluster
Why: check that another pod can reach the app by the Service name.

```
kubectl run curl-client --image=curlimages/curl --restart=Never --rm -i -n dev --command -- curl -sS http://nginx-service
```

Output: Welcome to nginx! The temporary pod was deleted afterwards.

### Test from the browser
Why: a ClusterIP Service has no public URL. Port forwarding gives temporary access.

```
kubectl port-forward service/nginx-service 8081:80 -n dev
```

Output: http://localhost:8081 showed Welcome to nginx! Stopped with Ctrl+C. Port forwarding connects to one pod, so it does not test load sharing.

### Self-healing
Why: check that a deleted pod is replaced.

```
kubectl get pods -n dev
kubectl delete pod <pod-name> -n dev
kubectl get pods -n dev
```

Output: deleted the pod ending in jnzfz. A new pod ending in d7rhk was created by the ReplicaSet.

### Scaling
Why: change the number of pods manually.

```
kubectl scale deployment nginx-deployment --replicas=3 -n dev
kubectl get pods -n dev
kubectl scale deployment nginx-deployment --replicas=2 -n dev
kubectl get deployment,replicaset,pods,service -n dev
```

Output: 3 pods Running, then back to 2. Final: Deployment 2/2, ReplicaSet 2, nginx-service present.

### View in the AWS Console
EKS -> devops-eks -> Resources
- Workloads: Deployments, ReplicaSets, Pods
- Cluster: Namespaces
- Service and networking: Services, EndpointSlices
- Worker nodes are under the Compute tab

## Step 7: Install Argo CD
Argo CD is a GitOps tool that runs inside Kubernetes. It compares the files in Git with the cluster and helps keep them the same.

```
kubectl create namespace argocd
kubectl apply -n argocd --server-side --force-conflicts -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl get pods -n argocd
kubectl get services -n argocd
```

| Option | Meaning |
|--------|---------|
| `--server-side` | The API server handles the apply. Needed for large Argo CD definitions |
| `--force-conflicts` | Takes control of conflicting fields. Can overwrite changes on an existing setup |
| `-f` | File or URL to apply |

Output: 7 pods Running (1/1) with 0 restarts. 9 services, all ClusterIP. argocd-server uses ports 80 and 443.

| Component | Job |
|-----------|-----|
| argocd-server | Web UI and API |
| argocd-repo-server | Reads Git and prepares Kubernetes files |
| argocd-application-controller | Compares and syncs applications |
| argocd-applicationset-controller | Creates applications from templates |
| argocd-redis | Cache |
| argocd-dex-server | External login |
| argocd-notifications-controller | Notifications |

The stable install file was used for practice. For production, use a fixed version.

## Step 8: Access the Argo CD UI
Why: argocd-server is ClusterIP, so port forwarding is needed.

Terminal 1:

```
kubectl port-forward service/argocd-server -n argocd 8080:443
```

Terminal 2 (get the first password):

```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

After changing the password and logging in again:

```
kubectl -n argocd delete secret argocd-initial-admin-secret
```

Output: opened https://localhost:8080 and logged in as admin (Argo CD v3.6.3). Changed the password and deleted the initial secret. The certificate warning is expected here because the certificate is self-signed.

## Step 9: Delete the cluster and check AWS
Why: stop the cluster charges.

```
kubectl get services -A
eksctl delete cluster --name devops-eks --region ap-south-1 --disable-nodegroup-eviction --wait
eksctl get cluster --region ap-south-1
```

| Option | Meaning |
|--------|---------|
| `--disable-nodegroup-eviction` | Skips safe pod eviction. Used for this practice cluster. Not for casual use in production |
| `--wait` | Waits for deletion to finish |

Output:

```
all cluster resources were deleted
No clusters found
```

Console check was clean: EKS, CloudFormation, EC2 instances, Auto Scaling Groups, load balancers, volumes, NAT gateways, Elastic IPs and VPCs.

## Problems and fixes
| Problem or message | Meaning and fix |
|--------------------|-----------------|
| Warning about vpc-cni permissions because OIDC is disabled | Cluster creation completed. Later, system pods ran and the Service test passed |
| `created 0 nodegroup(s)` | A later line showed that 1 managed node group was created |
| `couldn't attach to pod/curl-client, falling back to streaming logs` | Harmless here. The container finished before the terminal attached |
| `Handling connection for 8081` | Normal while port forwarding. Stop with Ctrl+C |
| Console showed `Error loading resources` | The console login needed its own EKS access entry. Added it, then resources were visible |
| An Argo CD pod showed 0/1 at first | It was still starting. It became 1/1 |

## What I learned
- The CLI login and the console login can be different. Each needs access to see resources in the cluster.
- A ReplicaSet replaces a deleted pod.
- A ClusterIP Service works inside the cluster. Port forwarding is useful for testing from a laptop.
- Installing Argo CD creates its own pods and services in the argocd namespace.
- EKS resources cost money while they exist, so delete them after practice and check for leftovers.
- Never store keys or passwords in Git, chat or screenshots.

## Not covered yet
- Deploying an application through Argo CD
- Rolling update and rollback
- Blue-green deployment
- Writing manifest files myself
- Creating EKS from the AWS Console
- Terraform for EKS, Terraform modules and CI/CD
