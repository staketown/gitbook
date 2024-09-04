---
cover: ../../.gitbook/assets/lava-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v3.1.0** is available

```bash
cd $HOME || return
rm -rf lava
git clone https://github.com/lavanet/lava
cd lava || return
git checkout v3.1.0

make build

mkdir -p $HOME/.lava/cosmovisor/upgrades/v3.1.0/bin
mv build/lavad $HOME/.lava/cosmovisor/upgrades/v3.1.0/bin/
```
