# Phase B, Step 8: Deploy the app on EKS

## What I did
- Created the `dev` namespace on the EKS cluster
- Reused `deployment.yaml` from Phase A
- Made `service-loadbalancer.yaml`: same as `service.yaml` but `type: LoadBalancer`
- Applied both and tested through the load balancer

## Commands
```
kubectl create namespace dev
kubectl apply -f deployment.yaml -f service-loadbalancer.yaml
kubectl get pods -n dev -o wide
kubectl get svc -n dev
curl -s http://$(kubectl get svc web -n dev -o jsonpath='{.status.loadBalancer.ingress[0].hostname}') | head -n 5
kubectl get all -n dev
```

## The difference from Phase A
| File | Service type | Use |
|------|--------------|-----|
| `service.yaml` | ClusterIP | Internal only (Minikube) |
| `service-loadbalancer.yaml` | LoadBalancer | AWS creates a public load balancer (EKS) |

Both Services are named `web`. Apply only one of them.

## Result
- 2 pods Running 1/1, both on the one worker node
- The Service got an `.elb.amazonaws.com` address after about 3 minutes
- `curl` returned the "Welcome to nginx!" page
- The load balancer is visible in EC2 → Load Balancers

## How the traffic flows
Browser -> load balancer (port 80) -> node port -> Service `web` -> one of the 2 pods

## Cost and cleanup
- The load balancer charges by the hour
- It is not in the eksctl CloudFormation stacks. A Kubernetes Service created it
- Delete the Service first (`kubectl delete svc web -n dev`), or the VPC delete will be blocked
