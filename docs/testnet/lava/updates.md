---
cover: ../../.gitbook/assets/banner_cleanup.jpeg
coverY: 0
---

# Updates

> ⚠️ version: v0.26.1

```bash
cd $HOME || return
rm -rf lava
git clone https://github.com/lavanet/lava
cd $HOME/lava || return
git checkout v0.26.1

export LAVA_BINARY=lavad && make build

mkdir -p $HOME/.lava/cosmovisor/upgrades/v0.26.1/bin
mv build/lavad $HOME/.lava/cosmovisor/upgrades/v0.26.1/bin/
```
