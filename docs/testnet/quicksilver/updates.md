---
cover: ../../.gitbook/assets/quicksilver-banner.png
coverY: 0
---

# Updates

⚠️ Version **v1.6.1-rc0** is available

```bash
cd $HOME || return
wget -O quicksilverd https://github.com/quicksilver-zone/quicksilver/releases/download/v1.6.1-rc0/quicksilverd-v1.6.1-rc0-amd64
chmod +x quicksilverd
mv quicksilverd $HOME/go/bin

mkdir -p $HOME/.quicksilverd/cosmovisor/upgrades/v1.6.1-rc0/bin
cp $HOME/go/bin/quicksilverd $HOME/.quicksilverd/cosmovisor/upgrades/v1.6.1-rc0/bin/
```
