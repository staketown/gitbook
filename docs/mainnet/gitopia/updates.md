---
cover: ../../.gitbook/assets/gitopia-banner.png
coverY: 0
---

# Updates

⚠️ Version **v5.1.0** is available

```bash
cd $HOME || return
rm -rf gitopia
git clone https://github.com/gitopia/gitopia.git
cd gitopia || return
git checkout v5.1.0

make build

mkdir -p $HOME/.gitopia/cosmovisor/upgrades/v5.1.0/bin
mv build/gitopiad $HOME/.gitopia/cosmovisor/upgrades/v5.1.0/bin/
```
