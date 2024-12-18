---
cover: ../../.gitbook/assets/umee-banner.png
coverY: 0
---

# Updates

⚠️ Version **v6.7.2-rc1** is available

```bash
cd $HOME || return
rm -rf umee
git clone https://github.com/umee-network/umee.git
cd umee || return
git checkout v6.7.2-rc1

make build

mkdir -p $HOME/.umee/cosmovisor/upgrades/v6.7.2-rc1/bin
mv build/umeed $HOME/.umee/cosmovisor/upgrades/v6.7.2-rc1/bin/
```
