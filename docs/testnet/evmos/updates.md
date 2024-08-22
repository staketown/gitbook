---
cover: ../../.gitbook/assets/evmos-banner.png
coverY: 0
---

# Updates

⚠️ Version **v19.2.0** is available

```bash
cd $HOME || return
rm -rf evmos
git clone https://github.com/tharsis/evmos.git
cd evmos || return
git checkout v19.2.0

make build

mkdir -p $HOME/.evmosd/cosmovisor/upgrades/v19.2.0/bin
mv build/evmosd $HOME/.evmosd/cosmovisor/upgrades/v19.2.0/bin/
```
