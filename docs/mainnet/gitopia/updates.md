---
cover: ../../.gitbook/assets/gitopia-banner.png
coverY: 0
---

# Updates

> ⚠️ Version **v4.0.0** is available

```bash
cd $HOME || return
rm -rf gitopia
git clone https://github.com/gitopia/gitopia
cd gitopia || return
git checkout v4.0.0

make build

mkdir -p $HOME/.gitopia/cosmovisor/upgrades/v4/bin
mv build/gitopiad $HOME/.gitopia/cosmovisor/upgrades/v4/bin/

rm -rf build
```
