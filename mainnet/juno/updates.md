---
cover: ../../.gitbook/assets/juno-banner.jpeg
coverY: 0
---

# Updates

> ⚠️ **v17.0.0 is available**

```bash
cd $HOME || return
rm -rf juno
git clone https://github.com/CosmosContracts/juno juno
cd juno || return
git checkout v17.0.0

make build

mkdir -p $HOME/.juno/cosmovisor/upgrades/v17/bin
mv bin/junod $HOME/.juno/cosmovisor/upgrades/v17/bin/
```
