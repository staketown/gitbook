---
cover: ../../.gitbook/assets/quicksilver-banner.png
coverY: 0
---

# Updates

⚠️ Version **v1.4.5-rc4** is available

```bash
cd $HOME || return
wget -O quicksilverd https://github.com/quicksilver-zone/quicksilver/releases/download/v1.4.5-rc4/quicksilverd-v1.4.5-rc4-amd64
chmod +x quicksilverd
mv quicksilverd $HOME/go/bin

mkdir -p $HOME/.quicksilverd/cosmovisor/upgrades/v1.4.5-rc4/bin
mv  $HOME/go/bin/quicksilverd $HOME/.quicksilverd/cosmovisor/upgrades/v1.4.5-rc4/bin/
```
