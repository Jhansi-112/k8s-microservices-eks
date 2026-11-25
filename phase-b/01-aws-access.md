# Phase B, Step 1: AWS access

## What I did
- Created IAM user `eks-admin` with the AdministratorAccess policy (practice account only)
- Created an access key for the CLI
- Used the AWS CLI v2 (already installed)
- Ran `aws configure` with region `ap-south-1` and output `json`
- Rotated the access key after the secret showed on screen, and deleted the old key

## Commands
aws --version
aws configure
aws sts get-caller-identity
Result: the Arn ends with `user/eks-admin`.

## Security rules
- Keys are saved in `~/.aws/credentials`, outside this repo
- Never put keys in README files, scripts or chat
- Run `clear` before taking screenshots of the terminal
- Delete unused access keys in IAM
- AdministratorAccess is for practice only

## Problems and fixes
| Problem | Fix |
|---------|-----|
| `dpkg was interrupted` | `sudo dpkg --configure -a`, then `sudo apt update` |
| `Found preexisting AWS CLI installation` | Skip the install. Run `aws --version` |
| Commands pasted together ran out of order | Run them one at a time |
