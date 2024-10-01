---
cover: ../../.gitbook/assets/c4e-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v1.4.3** is available

```bash
cd $HOME || return
rm -rf c4e-chain
git clone https://github.com/chain4energy/c4e-chain.git
cd c4e-chain || return
git checkout v1.4.3

make install

mkdir -p $HOME/.c4e-chain/cosmovisor/upgrades/v1.4.3/bin
mv $HOME/go/bin/c4ed $HOME/.c4e-chain/cosmovisor/upgrades/v1.4.3/bin/
```
