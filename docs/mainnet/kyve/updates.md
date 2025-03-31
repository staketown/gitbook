---
cover: ../../.gitbook/assets/kyve-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v2.0.0** is available

```bash
cd $HOME || return
rm -rf $HOME/kyve
git clone https://github.com/KYVENetwork/chain.git kyve
cd $HOME/kyve || return
git checkout v2.0.0

make build ENV=mainnet

mkdir -p $HOME/.kyve/cosmovisor/upgrades/v2.0.0/bin
mv build/kyved $HOME/.kyve/cosmovisor/upgrades/v2.0.0/bin/
```
