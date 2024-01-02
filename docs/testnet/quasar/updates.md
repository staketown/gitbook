---
cover: ../../.gitbook/assets/quasar-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v1.0.0-rc1-testnet** is available

```bash
cd $HOME || return
rm -rf quasar
git clone https://github.com/quasar-finance/quasar.git
cd quasar || return
git checkout v1.0.0-rc1-testnet

make build

mkdir -p $HOME/.quasarnode/cosmovisor/upgrades//bin
mv /quasarnoded $HOME/.quasarnode/cosmovisor/upgrades//bin/
```
