---
cover: ../../.gitbook/assets/composable-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version v6.6.4 is available

```bash
cd $HOME || return
rm -rf composable-centauri
git clone https://github.com/notional-labs/composable-centauri.git
cd $HOME/composable-centauri || return
git checkout v6.6.4

make build

mkdir -p $HOME/.banksy/cosmovisor/upgrades/v6_6_4/bin
mv bin/picad $HOME/.banksy/cosmovisor/upgrades/v6_6_4/bin/centaurid

rm -rf bin
```
