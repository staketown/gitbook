---
cover: ../../.gitbook/assets/andromeda-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v0.1.1-fixgov** is available

```bash
cd $HOME || return
rm -rf andromedad
git clone https://github.com/andromedaprotocol/andromedad.git
cd andromedad || return
git checkout v0.1.1-fixgov

make build

mkdir -p $HOME/.andromeda/cosmovisor/upgrades/v0.1.1/bin
mv bin/andromedad $HOME/.andromeda/cosmovisor/upgrades/v0.1.1/bin/
```
