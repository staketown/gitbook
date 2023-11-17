---
cover: ../../.gitbook/assets/juno-banner.jpeg
coverY: 0
---

# Updates

⚠️ **v18.0.0-alpha.3 is available**

```bash
cd $HOME || return
rm -rf juno
git clone https://github.com/CosmosContracts/juno juno
cd juno || return
git checkout v18.0.0-alpha.3

make build

mkdir -p $HOME/.juno/cosmovisor/upgrades/v1800alpha3/bin
mv bin/junod $HOME/.juno/cosmovisor/upgrades/v1800alpha3/bin/
```
