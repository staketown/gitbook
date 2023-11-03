---
cover: ../../.gitbook/assets/gitopia-banner.png
coverY: 0
---

# Updates

> ⚠️ Version **v3.3.0** is available

```bash
cd $HOME || return
rm -rf gitopia
git clone https://github.com/gitopia/gitopia
cd gitopia || return
git checkout v3.3.0

make build

mkdir -p $HOME/.gitopia/cosmovisor/upgrades/v3.3.0/bin
mv build/gitopiad $HOME/.gitopia/cosmovisor/upgrades/v3.3.0/bin/

rm -rf build
```
