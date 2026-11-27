# Step 4: Create EKS Cluster

Jira Epic: KUBE-23
Jira Task: KUBE-27

## Command
eksctl create cluster --name devops-eks --region ap-south-1 --zones ap-south-1a,ap-south-1b --version 1.36 --without-nodegroup

## Verification Commands
- eksctl get cluster --region ap-south-1
- aws eks describe-cluster --name devops-eks --region ap-south-1 --query "cluster.[status,version]" --output text

## Result
- Cluster name: devops-eks
- Region: ap-south-1
- Status: ACTIVE
- Version: 1.36
- Add-ons created: vpc-cni, kube-proxy, coredns, metrics-server
- Worker node group: not added in this step

## AWS Console Pages
EKS, CloudFormation, VPC, IAM and EC2

## Observation
eksctl showed a warning about IAM permissions for the vpc-cni add-on because OIDC is disabled. Cluster creation completed. Networking will be checked after adding worker nodes.
