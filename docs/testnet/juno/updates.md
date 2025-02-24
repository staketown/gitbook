---
cover: ../../.gitbook/assets/juno-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v28.0.1** is available

```bash
cd $HOME || return
rm -rf juno
git clone https://github.com/CosmosContracts/juno.git
cd juno || return
git checkout v28.0.1

make build

mkdir -p $HOME/.juno/cosmovisor/upgrades/v28/bin
mv bin/junod $HOME/.juno/cosmovisor/upgrades/v28/bin/
```
