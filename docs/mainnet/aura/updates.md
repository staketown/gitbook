---
cover: ../../.gitbook/assets/aura-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **aura_v0.4.5** is available

```bash
cd $HOME || return
rm -rf aura
git clone https://github.com/aura-nw/aura.git
cd aura || return
git checkout aura_v0.4.5

make build

mkdir -p $HOME/.aura/cosmovisor/upgrades/v0.4.5/bin
mv build/aurad $HOME/.aura/cosmovisor/upgrades/v0.4.5/bin/
```
