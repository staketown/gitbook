---
cover: ../../.gitbook/assets/evmos-banner.png
coverY: 0
---

# Updates

⚠️ **v15.0.0 is available**

```bash
cd $HOME || return
rm -rf evmos
git clone https://github.com/tharsis/evmos
cd evmos || return
git checkout v15.0.0

make build

mkdir -p $HOME/.evmosd/cosmovisor/upgrades/v15.0.0/bin
mv build/evmosd $HOME/.evmosd/cosmovisor/upgrades/v15.0.0/bin/

rm -rf build
```
