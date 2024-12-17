---
cover: ../../.gitbook/assets/elys-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **fix/v1.2.0-patch-1** is available

```bash
cd $HOME || return
rm -rf elys
git clone https://github.com/elys-network/elys.git
cd elys || return
git checkout fix/v1.2.0-patch-1

make build

mkdir -p $HOME/.elys/cosmovisor/upgrades/v1.2.0/bin
mv build/elysd $HOME/.elys/cosmovisor/upgrades/v1.2.0/bin/
```
