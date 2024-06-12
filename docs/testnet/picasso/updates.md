---
cover: ../../.gitbook/assets/picasso-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v7.0.3** is available

```bash
cd $HOME || return
rm -rf composable-testnet
git clone https://github.com/notional-labs/composable-testnet.git
cd $HOME/composable-testnet || return
git checkout v7.0.3

make build

mkdir -p $HOME/.banksy/cosmovisor/upgrades/v7_0_3/bin
mv bin/picad $HOME/.banksy/cosmovisor/upgrades/v7_0_3/bin/

rm -rf bin
```
