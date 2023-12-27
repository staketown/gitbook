---
cover: ../../.gitbook/assets/quicksilver-banner.png
coverY: 0
---

# Updates

⚠️ Version **v1.4.5-rc.2** is available

```bash
cd $HOME || return
rm -rf quicksilver
git clone https://github.com/ingenuity-build/quicksilver.git
cd quicksilver || return
git checkout v1.4.5-rc.2

make build

mkdir -p $HOME/.quicksilverd/cosmovisor/upgrades/v1.4.5-rc.2/bin
mv build/quicksilverd $HOME/.quicksilverd/cosmovisor/upgrades/v1.4.5-rc.2/bin/
```
