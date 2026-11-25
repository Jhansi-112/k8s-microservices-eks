# Phase B, Step 6: Connect kubectl to the cluster

## What I did
- Checked that kubectl can see the node
- Added the cluster to my kubeconfig with the AWS CLI

## Commands
```
kubectl get node
kubectl get node -o wide
aws eks update-kubeconfig --region ap-south-1 --name devops-eks
```

## Result
- 1 node, STATUS Ready, Kubernetes v1.36.5
- OS: Amazon Linux 2023, container runtime: containerd
- `update-kubeconfig` added the `devops-eks` context to `~/.kube/config`

## How it works
- kubectl reads `~/.kube/config`
- That file holds the API server address and how to log in
- If the file is renamed or deleted, kubectl stops working
- `eksctl create cluster` already wrote this file. `update-kubeconfig` does the same and is useful on a new machine

## Security
- `~/.kube/config` is outside this repo. Never commit it
