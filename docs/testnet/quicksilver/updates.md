---
cover: ../../.gitbook/assets/quicksilver-banner.png
coverY: 0
---

# Updates

⚠️ Version **v1.8.0-rc.0** is available

```bash
cd $HOME || return
rm -rf quicksilver
git clone https://github.com/ingenuity-build/quicksilver.git
cd $HOME/quicksilver || return
git checkout v1.8.0-rc.0

make install

mkdir -p $HOME/.quicksilverd/cosmovisor/upgrades/v1.8.0/bin
cp $HOME/go/bin/quicksilverd $HOME/.quicksilverd/cosmovisor/upgrades/v1.8.0/bin/
```
