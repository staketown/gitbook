---
cover: ../../.gitbook/assets/side-banner.png
coverY: 0
---

# Updates

⚠️ Version **v0.9.4** is available

```bash
cd $HOME || return
rm -rf side
git clone https://github.com/sideprotocol/side.git
cd side || return
git checkout v0.9.4

make build

mkdir -p $HOME/.side/cosmovisor/upgrades/v0.9.3/bin
mv build/sided $HOME/.side/cosmovisor/upgrades/v0.9.3/bin/
```
