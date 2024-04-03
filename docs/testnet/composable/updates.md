---
cover: ../../.gitbook/assets/composable-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v6.5.21** is available

```bash
cd $HOME || return
rm -rf composable-testnet
git clone https://github.com/notional-labs/composable-testnet.git
cd $HOME/composable-testnet || return
git checkout v6.5.21

make build

mkdir -p $HOME/.banksy/cosmovisor/upgrades/v6_5_2/bin
mv bin/centaurid $HOME/.banksy/cosmovisor/upgrades/v6_5_2/bin/

rm -rf bin
```
