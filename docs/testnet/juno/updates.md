---
cover: ../../.gitbook/assets/juno-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v29.0.0** is available

```bash
cd $HOME || return
rm -rf juno
git clone https://github.com/CosmosContracts/juno.git
cd juno || return
git checkout v29.0.0

make build

mkdir -p $HOME/.juno/cosmovisor/upgrades/v29/bin
mv bin/junod $HOME/.juno/cosmovisor/upgrades/v29/bin/
```
