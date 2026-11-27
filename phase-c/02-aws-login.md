# Step 2: Verify AWS Login

Jira Epic: KUBE-23
Jira Task: KUBE-25

## Command
aws sts get-caller-identity

## Purpose
Check which AWS identity the CLI is using before creating EKS.

## Result
- Login check: Successful
- Identity type: IAM user (EKS admin)

## Security
Access keys and AWS account details are not stored in this file.
