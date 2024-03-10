---
cover: ../../.gitbook/assets/juno-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v21.0.0-alpha.1** is available

```bash
wget -O $HOME/go/bin/junod https://security.junonetwork.io/v21.0.0/junod
cd $HOME || return
rm -rf juno
git clone https://github.com/CosmosContracts/juno.git
cd juno || return
git checkout v21.0.0-alpha.1

make build

mkdir -p $HOME/.juno/cosmovisor/upgrades/v2100alpha1/bin
mv bin/junod $HOME/.juno/cosmovisor/upgrades/v2100alpha1/bin/
```
