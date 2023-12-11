---
cover: ../../.gitbook/assets/aura-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v0.7.2** is available

```bash
cd $HOME || return
rm -rf aura
git clone https://github.com/aura-nw/aura.git
cd aura || return
git checkout v0.7.2

make build

mkdir -p $HOME/.aura/cosmovisor/upgrades/v0.7.2/bin
mv build/aurad $HOME/.aura/cosmovisor/upgrades/v0.7.2/bin/
```
