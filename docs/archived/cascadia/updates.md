---
cover: ../../.gitbook/assets/cascadia-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v0.3.0** is available

```bash
cd $HOME || return
rm -rf cascadia
git clone https://github.com/cascadiafoundation/cascadia.git
cd cascadia || return
git checkout v0.3.0

make build

mkdir -p $HOME/.cascadiad/cosmovisor/upgrades/v0.3.0/bin
mv build/cascadiad $HOME/.cascadiad/cosmovisor/upgrades/v0.3.0/bin/
```
