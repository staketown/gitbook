---
cover: ../../.gitbook/assets/elys-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v4.0.0-rc1** is available

```bash
cd $HOME || return
rm -rf elys
git clone https://github.com/elys-network/elys.git
cd elys || return
git checkout v4.0.0-rc1

make build

mkdir -p $HOME/.elys/cosmovisor/upgrades/v4-rc1/bin
mv build/elysd $HOME/.elys/cosmovisor/upgrades/v4-rc1/bin/
```
