---
cover: ../../.gitbook/assets/elys-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **fix/v0.31.0-restore-clock-gas-limit-value** is available

```bash
cd $HOME || return
rm -rf elys
git clone https://github.com/elys-network/elys.git
cd elys || return
git checkout fix/v0.31.0-restore-clock-gas-limit-value

make build

mkdir -p $HOME/.elys/cosmovisor/upgrades/v0.31.0/bin
mv build/elysd $HOME/.elys/cosmovisor/upgrades/v0.31.0/bin/
```
