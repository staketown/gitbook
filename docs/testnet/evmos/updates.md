---
cover: ../../.gitbook/assets/evmos-banner.png
coverY: 0
---

# Updates

⚠️ **v16.0.0-rc2.1 is available**

```bash
cd $HOME || return
rm -rf evmos
git clone https://github.com/tharsis/evmos
cd evmos || return
git checkout v16.0.0-rc2.1

make build

mkdir -p $HOME/.evmosd/cosmovisor/upgrades/v16.0.0-rc2/bin
mv build/evmosd $HOME/.evmosd/cosmovisor/upgrades/v16.0.0-rc2/bin/

rm -rf build
```

