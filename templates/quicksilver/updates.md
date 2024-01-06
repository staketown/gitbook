---
cover: ../../.gitbook/assets/{{BANNER_NAME}}
coverY: 0
---

# Updates

⚠️ Version **{{BINARY_VERSION}}** is available

```bash
cd $HOME || return
wget -O quicksilverd https://github.com/quicksilver-zone/quicksilver/releases/download/{{BINARY_VERSION}}/quicksilverd-{{BINARY_VERSION}}-amd64
chmod +x quicksilverd
mv quicksilverd $HOME/go/bin

mkdir -p $HOME/{{WORKING_DIR}}/cosmovisor/upgrades/{{BUILD_UPGRADE_DIR}}/bin
cp $HOME/go/bin/{{BINARY}} $HOME/{{WORKING_DIR}}/cosmovisor/upgrades/{{BUILD_UPGRADE_DIR}}/bin/
```
