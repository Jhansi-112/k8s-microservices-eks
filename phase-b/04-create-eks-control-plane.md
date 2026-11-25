# Phase B, Step 4: Create the EKS control plane

## What I did
- Chose Kubernetes 1.36 so it matches kubectl 1.37 (kubectl works within one minor version of the cluster)
- Checked that no cluster existed
- Created the control plane only, with no worker nodes

## Commands

aws eks describe-cluster-versions --region ap-south-1 --query 'clusterVersions[].[clusterVersion,status]' --output table
eksctl get cluster --region ap-south-1
eksctl create cluster --name devops-eks --region ap-south-1 --zones ap-south-1a,ap-south-1b --version 1.36 --without-nodegroup
eksctl get cluster --region ap-south-1
aws eks describe-cluster --name devops-eks --region ap-south-1 --query cluster.version --output text
kubectl get svc


## What the options mean
| Option | Meaning |
|--------|---------|
| `--name devops-eks` | Cluster name, used in all later commands |
| `--zones` | Two Availability Zones for the VPC |
| `--version 1.36` | Kubernetes version |
| `--without-nodegroup` | Control plane only. Worker nodes come in the next step |

## Result
- Cluster `devops-eks` was ready in about 11 minutes
- `describe-cluster` returned version 1.36
- `kubectl get svc` shows the `kubernetes` service. There are no nodes yet
- eksctl made a CloudFormation stack with a VPC, public and private subnets, a NAT gateway and IAM roles
- Add-ons installed: coredns, vpc-cni, kube-proxy, metrics-server
- kubeconfig was saved in `~/.kube/config`

## Cost
- The EKS control plane and the NAT gateway charge by the hour
- Delete everything at the end of the lab (Step 10)

## Notes
| Message | Meaning |
|---------|---------|
| `Auto Mode will be enabled by default in an upcoming release` | Notice about future eksctl versions. No action |
| `recommended policies were found for "vpc-cni" addon, but OIDC is disabled` | Only matters for pod-level IAM roles (IRSA) later. No action now |
