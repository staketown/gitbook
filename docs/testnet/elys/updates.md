---
cover: ../../.gitbook/assets/elys-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v5.0.0-rc0** is available

```bash
cd $HOME || return
rm -rf elys
git clone https://github.com/elys-network/elys.git
cd elys || return
git checkout v5.0.0-rc0

make build

mkdir -p $HOME/.elys/cosmovisor/upgrades/v5-rc0/bin
mv build/elysd $HOME/.elys/cosmovisor/upgrades/v5-rc0/bin/
```
