---
cover: ../../.gitbook/assets/umee-banner.png
coverY: 0
---

# Updates

⚠️ Version **v6.2.0** is available

```bash
cd $HOME || return
rm -rf umee
git clone https://github.com/umee-network/umee.git
cd umee || return
git checkout v6.2.0

make build

mkdir -p $HOME/.umee/cosmovisor/upgrades/v6.2/bin
mv build/umeed $HOME/.umee/cosmovisor/upgrades/v6.2/bin/
```
