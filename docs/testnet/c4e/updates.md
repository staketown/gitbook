---
cover: ../../.gitbook/assets/c4e-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v1.3.0** is available

```bash
cd $HOME || return
rm -rf c4e-chain
git clone https://github.com/chain4energy/c4e-chain.git
cd c4e-chain || return
git checkout v1.3.0

make build

mkdir -p $HOME/.c4e-chain/cosmovisor/upgrades/v1.3.0/bin
mv build/c4ed $HOME/.c4e-chain/cosmovisor/upgrades/v1.3.0/bin/
```
