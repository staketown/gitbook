---
cover: ../../.gitbook/assets/c4e-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v1.3.1** is available

```bash
cd $HOME || return
rm -rf c4e-chain
git clone https://github.com/chain4energy/c4e-chain
cd $HOME/c4e-chain || return
git checkout v1.3.1

make build

mkdir -p $HOME/.c4e-chain/cosmovisor/upgrades/v1.3.1/bin
mv build/c4ed $HOME/.c4e-chain/cosmovisor/upgrades/v1.3.1/bin/
```
