---
cover: ../../.gitbook/assets/cascadia-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v0.1.8** is available

```bash
cd $HOME || return
rm -rf cascadia
git clone https://github.com/cascadiafoundation/cascadia
cd $HOME/cascadia || return
git checkout v0.1.8

make build

mkdir -p $HOME/.cascadiad/cosmovisor/upgrades/v0.1.8/bin
mv build/cascadiad $HOME/.cascadiad/cosmovisor/upgrades/v0.1.8/bin/
```
