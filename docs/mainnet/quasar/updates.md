---
cover: ../../.gitbook/assets/quasar-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v2.0.1** is available

```bash
cd $HOME || return
rm -rf quasar
git clone https://github.com/quasar-finance/quasar.git
cd quasar || return
git checkout v2.0.1

make build

mkdir -p $HOME/.quasarnode/cosmovisor/upgrades/v2/bin
mv build/quasarnoded $HOME/.quasarnode/cosmovisor/upgrades/v2/bin/
```
