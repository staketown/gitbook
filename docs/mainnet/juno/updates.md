---
cover: ../../.gitbook/assets/juno-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v22.0.1** is available

```bash
cd $HOME || return
rm -rf juno
git clone https://github.com/CosmosContracts/juno.git
cd juno || return
git checkout v22.0.1

make build

mkdir -p $HOME/.juno/cosmovisor/upgrades/v22/bin
mv bin/junod $HOME/.juno/cosmovisor/upgrades/v22/bin/
```
