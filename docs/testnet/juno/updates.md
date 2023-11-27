---
cover: ../../.gitbook/assets/juno-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v18.0.0-alpha.4** is available

```bash
cd $HOME || return
rm -rf juno
git clone https://github.com/CosmosContracts/juno.git
cd juno || return
git checkout v18.0.0-alpha.4

make build

mkdir -p $HOME/.juno/cosmovisor/upgrades/v1800alpha4/bin
mv bin/junod $HOME/.juno/cosmovisor/upgrades/v1800alpha4/bin/
```
