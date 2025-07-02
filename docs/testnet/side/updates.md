---
cover: ../../.gitbook/assets/side-banner.png
coverY: 0
---

# Updates

⚠️ Version **v2.0.0-rc.6** is available

```bash
cd $HOME || return
rm -rf $HOME/side
git clone https://github.com/sideprotocol/side.git
cd $HOME/side || return
git checkout v2.0.0-rc.6

make install

mkdir -p $HOME/.side/cosmovisor/upgrades/v2.0.0-rc.6/bin
cp $HOME/go/bin/sided $HOME/.side/cosmovisor/upgrades/v2.0.0-rc.6/bin/
```
