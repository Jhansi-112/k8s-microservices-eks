# Step 5: Add Worker Nodes

Jira Epic: KUBE-23
Jira Task: KUBE-28

## Command
eksctl create nodegroup --cluster devops-eks --region ap-south-1 --name devops-eks-nodegroup --node-type t3.small --managed --nodes 2 --nodes-min 1 --nodes-max 2

## Verification Commands
- kubectl get nodes
- kubectl get pods -n kube-system

## Result
- Node group: devops-eks-nodegroup (managed)
- Instance type: t3.small
- Size: desired 2, minimum 1, maximum 2
- Worker nodes: 2 Ready
- Node version: v1.36.5-eks-9adbfea

## System Pods

| Component | Pods | Status |
|---|---|---|
| aws-node | 2 | Running (2/2 each) |
| coredns | 2 | Running (1/1 each) |
| kube-proxy | 2 | Running (1/1 each) |
| metrics-server | 2 | Running (1/1 each) |
