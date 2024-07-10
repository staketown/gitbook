---
cover: ../../.gitbook/assets/juno-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v23.0.0-alpha.1** is available

```bash
wget -O $HOME/go/bin/junod https://security.junonetwork.io/v23.0.0-alpha.1/junod
cd $HOME || return
rm -rf juno
git clone https://github.com/CosmosContracts/juno.git
cd juno || return
git checkout v23.0.0-alpha.1

make build

mkdir -p $HOME/.juno/cosmovisor/upgrades/v23/bin
mv bin/junod $HOME/.juno/cosmovisor/upgrades/v23/bin/
```
