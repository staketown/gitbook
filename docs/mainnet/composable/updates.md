---
cover: ../../.gitbook/assets/composable-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version v6.5.0 is available

```bash
cd $HOME || return
rm -rf composable-centauri
git clone https://github.com/notional-labs/composable-centauri.git
cd $HOME/composable-centauri || return
git checkout v6.5.0

make build

mkdir -p $HOME/.banksy/cosmovisor/upgrades/v6_5_0/bin
mv bin/centaurid $HOME/.banksy/cosmovisor/upgrades/v6_5_0/bin/

rm -rf bin
```
