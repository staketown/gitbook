---
cover: ../../.gitbook/assets/osmosis-banner.png
coverY: 0
---

# Updates

> ⚠️ **v22.0.0 is available**

```bash
cd $HOME
rm -rf osmosis
git clone https://github.com/osmosis-labs/osmosis.git
cd osmosis
git checkout v22.0.0

make build

mkdir -p $HOME/.osmosisd/cosmovisor/upgrades/v22/bin
mv build/osmosisd $HOME/.osmosisd/cosmovisor/upgrades/v22/bin/
rm -rf build
```
