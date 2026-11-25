# Step 10: Cleanup (delete everything)

Jira: KUBE-22 (Epic KUBE-12)

## Goal
Delete every resource created for the lab, so there are no AWS charges.

## Correct order
1. Delete the app and the Service (this removes the AWS load balancer).
2. Delete the node group.
3. Delete the cluster.
4. Check the AWS console for leftovers.

Why this order: a load balancer created by a Kubernetes Service is not part of the eksctl stacks. If it is left behind, it blocks the VPC delete.

## Commands used

### 10A. Delete the app and the load balancer
    kubectl delete hpa web -n dev
    kubectl delete -f deployment.yaml -f service-loadbalancer.yaml
    kubectl get all -n dev

Result: "No resources found in dev namespace". EC2 Load Balancers list was empty.

### 10B. Delete the node group and the cluster
First tries:

    eksctl delete nodegroup --cluster devops-eks --region ap-south-1 --name devops-eks-nodegroup --approve

- Error: --approve cannot be used unless a config file is specified.

    eksctl delete nodegroup --cluster devops-eks --region ap-south-1 --name devops-eks-nodegroup

- Drain timed out twice (about 25 minutes each). 2 pods were unevictable.

Fix that worked:

    eksctl delete cluster --name devops-eks --region ap-south-1 --disable-nodegroup-eviction

Result: both CloudFormation stacks were deleted. eksctl printed "all cluster resources were deleted".

## Problems and fixes
- Problem: --approve error on delete nodegroup. Fix: remove --approve (it is only for config files).
- Problem: node drain timed out ("2 pods are unevictable"). Cause: system pods (like coredns) could not move because both nodes were cordoned and no other node was available. Fix: skip the drain with --disable-nodegroup-eviction while deleting the whole cluster.
- Lost time: about 50 minutes of drain waiting, while the nodes and NAT gateway were still billing.

## Leftover check (AWS console, ap-south-1)
- EC2 instances: none running
- Load balancers: none
- Volumes: none
- Elastic IPs: none
- NAT gateways: none
- VPC: eksctl VPC gone
- CloudFormation: no eksctl stacks
- EKS clusters: none

## What I learned
- Always delete the Service first so the load balancer is removed.
- The EKS control plane, NAT gateway, EC2 nodes, and load balancer all charge by the hour.
- Check every console page after deleting. Leftover Elastic IPs and volumes still cost money.
- If drain fails while deleting the whole cluster, use --disable-nodegroup-eviction.
- Do the lab in one sitting and delete the same day.
