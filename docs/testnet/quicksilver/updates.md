---
cover: ../../.gitbook/assets/quicksilver-banner.png
coverY: 0
---

# Updates

⚠️ Version **v1.6.0-beta0-hotfix** is available

```bash
cd $HOME || return
wget -O quicksilverd https://github.com/quicksilver-zone/quicksilver/releases/download/v1.6.0-beta0-hotfix/quicksilverd-v1.6.0-beta0-hotfix-amd64
chmod +x quicksilverd
mv quicksilverd $HOME/go/bin

mkdir -p $HOME/.quicksilverd/cosmovisor/upgrades/v1.6.0-beta0/bin
cp $HOME/go/bin/quicksilverd $HOME/.quicksilverd/cosmovisor/upgrades/v1.6.0-beta0/bin/
```
