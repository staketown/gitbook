---
cover: ../../.gitbook/assets/evmos-banner.png
coverY: 0
---

# Updates

⚠️ Version **v16.0.3** is available

```bash
cd $HOME || return
rm -rf evmos
git clone https://github.com/tharsis/evmos.git
cd evmos || return
git checkout v16.0.3

make build

mkdir -p $HOME/.evmosd/cosmovisor/upgrades/v16.0.0/bin
mv build/evmosd $HOME/.evmosd/cosmovisor/upgrades/v16.0.0/bin/
```
