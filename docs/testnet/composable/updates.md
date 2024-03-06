---
cover: ../../.gitbook/assets/composable-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v6.4.5-fix** is available

```bash
cd $HOME || return
rm -rf composable-testnet
git clone https://github.com/notional-labs/composable-testnet.git
cd $HOME/composable-testnet || return
git checkout v6.4.5-fix

make build

mkdir -p $HOME/.banksy/cosmovisor/upgrades/v6_4_5/bin
mv bin/centaurid $HOME/.banksy/cosmovisor/upgrades/v6_4_5/bin/

rm -rf bin
```
