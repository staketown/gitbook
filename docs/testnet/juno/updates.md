---
cover: ../../.gitbook/assets/juno-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v19.0.0-alpha.3** is available

```bash
cd $HOME || return
rm -rf juno
git clone https://github.com/CosmosContracts/juno.git
cd juno || return
git checkout v19.0.0-alpha.3

make build

mkdir -p $HOME/.juno/cosmovisor/upgrades/v1900alpha3/bin
mv bin/junod $HOME/.juno/cosmovisor/upgrades/v1900alpha3/bin/
```
