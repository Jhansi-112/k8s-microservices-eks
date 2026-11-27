# Step 6: Kubernetes Resources on EKS

Jira Epic: KUBE-23
Jira Task: KUBE-29

## Resource Flow
Deployment -> ReplicaSet -> Pods
The Service sends traffic to the Pods.

## Commands
- kubectl create namespace dev
- kubectl get namespaces
- kubectl create deployment nginx-deployment --image=nginx:1.27 --replicas=2 --port=80 -n dev
- kubectl rollout status deployment/nginx-deployment -n dev
- kubectl get deployment,replicaset,pods -n dev -o wide
- kubectl expose deployment nginx-deployment --name=nginx-service --type=ClusterIP --port=80 --target-port=80 -n dev
- kubectl get service -n dev
- kubectl get endpointslices -n dev
- kubectl run curl-client --image=curlimages/curl --restart=Never --rm -i -n dev --command -- curl -sS http://nginx-service
- kubectl port-forward service/nginx-service 8081:80 -n dev
- kubectl get pods -n dev
- kubectl delete pod <pod-name> -n dev
- kubectl scale deployment nginx-deployment --replicas=3 -n dev
- kubectl scale deployment nginx-deployment --replicas=2 -n dev
- kubectl get deployment,replicaset,pods,service -n dev

## Test Results

| Test | Result |
|---|---|
| Service test from a temporary pod | Nginx welcome page received |
| Port forwarding | Page opened at http://localhost:8081 |
| Self-healing | Deleted one pod and a new pod was created |
| Scaling | Scaled from 2 to 3 pods, then back to 2 |
| Final state | Deployment 2/2, 2 Pods Running |

## AWS Console
The pods were visible under EKS Resources after giving the console login access to the cluster.
