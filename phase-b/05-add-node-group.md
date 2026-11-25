# Phase B, Step 5: Add a managed node group

## What I did
- Checked that no node group and no nodes existed
- Created a managed node group with 1 `t3.small` node

## Commands
```
eksctl get nodegroup --cluster devops-eks --region ap-south-1
kubectl get node
eksctl create nodegroup --cluster devops-eks --region ap-south-1 --name devops-eks-nodegroup --node-type t3.small --managed --nodes 1 --nodes-min 1 --nodes-max 2 --ssh-access --ssh-public-key mykeypair1 --asg-access
eksctl get nodegroup --cluster devops-eks --region ap-south-1
```

## What the options mean
| Option | Meaning |
|--------|---------|
| `--cluster devops-eks` | Must match the cluster name exactly |
| `--node-type t3.small` | EC2 size: 2 vCPU, 2 GB RAM, about 11 pods |
| `--managed` | AWS manages the node lifecycle |
| `--nodes 1 --nodes-min 1 --nodes-max 2` | Start at 1, never below 1, never above 2 |
| `--ssh-access --ssh-public-key mykeypair1` | SSH login with the key pair from Step 3 |
| `--asg-access` | Adds the permission an autoscaler needs |

## Result
- Node group `devops-eks-nodegroup` is ACTIVE, min 1, max 2, desired 1
- Created in about 4 minutes
- AMI type AL2023_x86_64_STANDARD, managed
- One node is Ready

## Security and cost
- SSH access is on, so port 22 may be reachable from the internet, protected only by the key. Fine for a short lab
- The EC2 node charges by the hour. Delete it with the cluster at the end

## Notes
| Message | Meaning |
|---------|---------|
| `Error: No nodegroups found` before creating | Expected. There was none yet |
| `created 0 nodegroup(s)` before `created 1 managed nodegroup(s)` | Normal eksctl output |
