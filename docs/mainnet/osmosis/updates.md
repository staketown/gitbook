---
cover: ../../.gitbook/assets/osmosis-banner.png
coverY: 0
---

# Updates

> ⚠️ **v27.0.0 is available**

```bash
cd $HOME
rm -rf osmosis
git clone https://github.com/osmosis-labs/osmosis.git
cd osmosis
git checkout v27.0.0

make build

mkdir -p $HOME/.osmosisd/cosmovisor/upgrades/v27/bin
mv build/osmosisd $HOME/.osmosisd/cosmovisor/upgrades/v27/bin/
rm -rf build
```
