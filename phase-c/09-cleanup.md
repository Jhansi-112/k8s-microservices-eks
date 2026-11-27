# Step 9: Delete EKS Cluster and Check AWS

Jira Epic: KUBE-23
Jira Task: KUBE-32

## Commands
- kubectl get services -A
- eksctl delete cluster --name devops-eks --region ap-south-1 --disable-nodegroup-eviction --wait
- eksctl get cluster --region ap-south-1

## Result
- Deletion message: all cluster resources were deleted
- Cluster check: No clusters found in ap-south-1

## AWS Console Check
Checked for project leftovers. Result: clean.
- EKS clusters
- CloudFormation stacks
- EC2 instances
- Auto Scaling Groups
- Load balancers
- Volumes
- NAT gateways
- Elastic IPs
- VPCs

## Note
The disable-nodegroup-eviction option was used for this practice cluster. It should not be used casually in production.
