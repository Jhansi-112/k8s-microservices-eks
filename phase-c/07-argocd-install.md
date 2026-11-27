# Step 7: Install Argo CD on EKS

Jira Epic: KUBE-23
Jira Task: KUBE-30

## What is Argo CD
Argo CD is a GitOps tool that runs inside Kubernetes. It compares the files in Git with the cluster.

## Commands
- kubectl create namespace argocd
- kubectl apply -n argocd --server-side --force-conflicts -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
- kubectl get pods -n argocd
- kubectl get services -n argocd

## Pods

| Component | Status |
|---|---|
| argocd-application-controller | Running (1/1) |
| argocd-applicationset-controller | Running (1/1) |
| argocd-dex-server | Running (1/1) |
| argocd-notifications-controller | Running (1/1) |
| argocd-redis | Running (1/1) |
| argocd-repo-server | Running (1/1) |
| argocd-server | Running (1/1) |

## Services
- 9 services, all ClusterIP
- argocd-server ports: 80 and 443

## Note
The stable install file was used for practice. For production, use a fixed version.
