# Phase B, Step 3: Create the EC2 key pair

## Why
The node group command uses `--ssh-public-key mykeypair1`. The key pair must already exist in the same region.

## Commands
cd ~
aws ec2 describe-key-pairs --key-names mykeypair1 --region ap-south-1
mkdir -p ~/.ssh
aws ec2 create-key-pair --key-name mykeypair1 --region ap-south-1 --query 'KeyMaterial' --output text > ~/.ssh/mykeypair1.pem
chmod 400 ~/.ssh/mykeypair1.pem
ls -l ~/.ssh/mykeypair1.pem
aws ec2 describe-key-pairs --key-names mykeypair1 --region ap-south-1


## Result
- The first check returned `InvalidKeyPair.NotFound`, so I created the key pair
- The `.pem` file has permissions `-r--------`
- The last command shows `"KeyName": "mykeypair1"` with key type `rsa`

## Security rules
- The private key stays in `~/.ssh`, never in this repo
- `.gitignore` blocks `*.pem` as a second guard
- AWS shows the private key only once. If it is lost, delete the key pair and create a new one

## Problems and fixes
| Problem | Fix |
|---------|-----|
| `InvalidKeyPair.NotFound` on the first check | Expected. Create the key pair |
