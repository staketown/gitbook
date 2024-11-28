---
cover: ../../.gitbook/assets/gitopia-banner.png
coverY: 0
---

# Updates

⚠️ Version **v5.0.1** is available

```bash
cd $HOME || return
rm -rf gitopia
git clone https://github.com/gitopia/gitopia.git
cd gitopia || return
git checkout v5.0.1

make build

mkdir -p $HOME/.gitopia/cosmovisor/upgrades/v5/bin
mv build/gitopiad $HOME/.gitopia/cosmovisor/upgrades/v5/bin/
```
