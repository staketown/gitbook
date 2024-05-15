---
cover: ../../.gitbook/assets/picasso-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v6.6.3** is available

```bash
cd $HOME || return
rm -rf composable-testnet
git clone https://github.com/notional-labs/composable-testnet.git
cd $HOME/composable-testnet || return
git checkout v6.6.3

make build

mkdir -p $HOME/.banksy/cosmovisor/upgrades/v6_6_3/bin
mv bin/picad $HOME/.banksy/cosmovisor/upgrades/v6_6_3/bin/

rm -rf bin
```
