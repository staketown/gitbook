---
cover: ../../.gitbook/assets/lava-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v0.27.0** is available

```bash
cd $HOME || return
rm -rf lava
git clone https://github.com/lavanet/lava
cd lava || return
git checkout v0.27.0

make build

mkdir -p $HOME/.lava/cosmovisor/upgrades/v0.27.0/bin
mv build/lavad $HOME/.lava/cosmovisor/upgrades/v0.27.0/bin/
```
