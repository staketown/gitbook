---
cover: ../../.gitbook/assets/picasso-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v6.6.41** is available

```bash
cd $HOME || return
rm -rf composable-centauri
git clone https://github.com/notional-labs/composable-centauri.git
cd composable-centauri || return
git checkout v6.6.41

make build

mkdir -p $HOME/.banksy/cosmovisor/upgrades/v6_6_4/bin
mv bin/picad $HOME/.banksy/cosmovisor/upgrades/v6_6_4/bin/
```
