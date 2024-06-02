---
cover: ../../.gitbook/assets/aura-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **ibcupgrade** is available

```bash
cd $HOME || return
rm -rf aura
git clone https://github.com/aura-nw/aura.git
cd aura || return
git checkout ibcupgrade

make build

mkdir -p $HOME/.aura/cosmovisor/upgrades/ibcupgrade/bin
mv build/aurad $HOME/.aura/cosmovisor/upgrades/ibcupgrade/bin/
```
