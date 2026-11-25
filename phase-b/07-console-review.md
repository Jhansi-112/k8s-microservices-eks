# Phase B, Step 7: What eksctl created in AWS

| Service | What to look at | What it is for |
|---------|-----------------|----------------|
| EKS | Cluster `devops-eks`: overview, compute, networking, add-ons | The control plane, node group and add-ons |
| EC2 | One `t3.small` instance | The worker node |
| Auto Scaling Groups | Min 1, desired 1, max 2 | Keeps the right number of nodes |
| VPC | Subnets: 2 public and 2 private, route tables, internet gateway | The network for the cluster |
| VPC | NAT gateway | Lets private subnets reach the internet. Charges hourly |
| IAM | Cluster service role and node instance role | Permissions for EKS and for the node |
| CloudFormation | `eksctl-devops-eks-cluster` and the node group stack | How eksctl builds and deletes everything |
| EC2 | Load Balancers (empty for now) | A load balancer appears when a Service uses type LoadBalancer |

## Why it matters
- Deleting the CloudFormation stacks removes what eksctl created
- A load balancer created by a Kubernetes Service is not in the stacks. Delete the Service first, or the VPC delete is blocked

## Interview question
What did eksctl create in your AWS account?
VPC with public and private subnets, a NAT gateway, IAM roles, the EKS control plane, add-ons, a managed node group backed by an Auto Scaling group, and the EC2 node.
