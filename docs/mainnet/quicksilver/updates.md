---
cover: ../../.gitbook/assets/quicksilver-banner.png
coverY: 0
---

# Updates

⚠️ Version **v1.7.7** is available

```bash
cd $HOME || return
rm -rf quicksilver
git clone https://github.com/ingenuity-build/quicksilver.git
cd $HOME/quicksilver || return
git checkout v1.7.7

make install

mkdir -p $HOME/.quicksilverd/cosmovisor/upgrades/v1.7.7/bin
cp $HOME/go/bin/quicksilverd $HOME/.quicksilverd/cosmovisor/upgrades/v1.7.7/bin/
```
