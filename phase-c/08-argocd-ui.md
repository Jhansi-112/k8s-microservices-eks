# Step 8: Access Argo CD UI

Jira Epic: KUBE-23
Jira Task: KUBE-31

## Why Port Forwarding
The argocd-server service is ClusterIP. Port forwarding gives temporary access from the laptop.

## Commands
- kubectl port-forward service/argocd-server -n argocd 8080:443
- kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
- kubectl -n argocd delete secret argocd-initial-admin-secret

## Result
- UI address: https://localhost:8080
- Argo CD version: v3.6.3
- Admin login: Successful
- First admin password: Changed
- Initial admin secret: Deleted

## Note
The browser certificate warning appears because this local setup uses a self-signed certificate.

## Security
No passwords are stored in this file.
