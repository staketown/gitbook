---
cover: ../../.gitbook/assets/quicksilver-banner.png
coverY: 0
---

# Updates

⚠️ Version **v1.2.17** is available

```bash
cd $HOME || return
rm -rf quicksilve
git clone https://github.com/ingenuity-build/quicksilver.git
cd quicksilve || return
git checkout v1.2.17

make build

mkdir -p $HOME/.quicksilverd/cosmovisor/upgrades/v1.2.17/bin
mv build/quicksilverd $HOME/.quicksilverd/cosmovisor/upgrades/v1.2.17/bin/
```
