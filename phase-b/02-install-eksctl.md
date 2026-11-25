# Phase B, Step 2: Install eksctl

## What I did
- Downloaded eksctl from the official GitHub releases
- Checked the file with the checksum list (`eksctl_Linux_amd64.tar.gz: OK`)
- Unpacked it and installed it to `/usr/local/bin`
- Checked the versions

## Commands
cd ~
ARCH=amd64
PLATFORM=(uname -s)_ ARCH
curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check
tar -xzf eksctl_PLATFORM.tar.gz -C /tmp && rm eksctl_ PLATFORM.tar.gz
sudo install -m 0755 /tmp/eksctl /usr/local/bin && rm /tmp/eksctl
eksctl version
kubectl version --client

## Result
- eksctl 0.232.0
- kubectl v1.37.2

## Notes
- kubectl should be within one minor version of the cluster. kubectl 1.37 matches clusters 1.36 and 1.37.

## Problems and fixes
| Problem | Cause | Fix |
|---------|-------|-----|
| `sha256sum: ...sbom.json: No such file` warning | The checksum list also has a second file I did not download | Harmless. The tar.gz file showed OK |
