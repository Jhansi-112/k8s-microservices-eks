# Step 9: Scaling on EKS

Jira: KUBE-21 (Epic KUBE-12)

## Goal
Scale the cluster in two ways:
1. Nodes: add a second worker node (more servers).
2. Pods: add a Horizontal Pod Autoscaler (HPA) for the web app.

## Commands used

### 9A. Scale nodes from 1 to 2
    eksctl scale nodegroup --cluster devops-eks --name devops-eks-nodegroup --nodes 2 --nodes-min 1 --nodes-max 3 --region ap-south-1

- nodes 2 = desired number of nodes
- nodes-min 1 = lowest allowed
- nodes-max 3 = highest allowed

### 9B. Check nodes
    kubectl get node
    eksctl get nodegroup --cluster devops-eks --region ap-south-1

### 9C. Create the pod autoscaler
    kubectl autoscale deploy web --cpu-percent=70 --min=2 --max=10 -n dev

- Keeps at least 2 pods and at most 10
- Adds pods when average CPU goes above 70% of the requested CPU (100m)

### 9D. Check the autoscaler
    kubectl get hpa -n dev

## Results

kubectl get node:
- 2 nodes, both Ready, version v1.36.5
- first node age about 3h, second node age about 36m

kubectl get hpa -n dev:
- web, Deployment/web, cpu 1%/70%, MINPODS 2, MAXPODS 10, REPLICAS 2

kubectl get all -n dev:
- 2 pods Running (1/1, 0 restarts)
- service/web type LoadBalancer
- deployment 2/2, replicaset 2/2
- horizontalpodautoscaler.autoscaling/web present

AWS console:
- Node group: desired 2, min 1, max 3

## Problem and fix
- Problem: the EKS console Nodes tab showed "Unauthorized".
- Cause: the console user had no Kubernetes access entry on the cluster.
- Fix: Access tab, Create access entry for the console user, policy AmazonEKSAdminViewPolicy, scope Cluster.
- Note: this only affects viewing in the console. kubectl worked the whole time because the user who created the cluster already had an access entry.

## What I learned
- Nodes and pods scale separately. HPA scales pods. A node autoscaler (Cluster Autoscaler or Karpenter) scales nodes.
- HPA needs metrics-server and CPU requests on the pods. Without requests it cannot calculate the percentage.
- TARGETS may show <unknown> for the first minute.
- Each extra node costs money while it runs, so scale down or delete after the lab.
- Pod limit per node depends on the instance type (t3.small is about 11 pods).
- I did not run a load test, to avoid extra cost.
