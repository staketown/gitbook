---
cover: ../../.gitbook/assets/elys-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v0.26.0** is available

```bash
cd $HOME || return
rm -rf elys
git clone https://github.com/elys-network/elys.git
cd elys || return
git checkout v0.26.0

make build

mkdir -p $HOME/.elys/cosmovisor/upgrades/v0.26.0/bin
mv build/elysd $HOME/.elys/cosmovisor/upgrades/v0.26.0/bin/
```
