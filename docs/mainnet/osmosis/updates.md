---
cover: ../../.gitbook/assets/osmosis-banner.png
coverY: 0
---

# Updates

> ⚠️ **v20.1.2 is available**

```bash
cd $HOME
rm -rf osmosis
git clone https://github.com/osmosis-labs/osmosis.git
cd osmosis
git checkout v20.1.2

make build

mkdir -p $HOME/.osmosisd/cosmovisor/upgrades/v20/bin
mv build/osmosisd $HOME/.osmosisd/cosmovisor/upgrades/v20/bin/
rm -rf build
```
