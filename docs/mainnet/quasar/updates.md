---
cover: ../../.gitbook/assets/quasar-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v3.0.0** is available

```bash
cd $HOME || return
rm -rf quasar
git clone https://github.com/quasar-finance/quasar.git
cd quasar || return
git checkout v3.0.0

make build

mkdir -p $HOME/.quasarnode/cosmovisor/upgrades/v3/bin
mv build/quasard $HOME/.quasarnode/cosmovisor/upgrades/v3/bin/
```
