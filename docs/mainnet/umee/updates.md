---
cover: ../../.gitbook/assets/umee-banner.png
coverY: 0
---

# Updates

> ⚠️ **v6.1.0 is available**

```bash
cd $HOME || return
rm -rf umee
git clone https://github.com/umee-network/umee.git
cd umee || return
git checkout v6.1.0

make build

mkdir -p $HOME/.umee/cosmovisor/upgrades/v6.1/bin
mv build/umeed $HOME/.umee/cosmovisor/upgrades/v6.1/bin/
rm -rf build
```
