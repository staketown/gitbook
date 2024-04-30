---
cover: ../../.gitbook/assets/andromeda-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **andromeda-1-v0.1.1** is available

```bash
cd $HOME || return
rm -rf andromedad
git clone https://github.com/andromedaprotocol/andromedad.git
cd andromedad || return
git checkout andromeda-1-v0.1.1

make build

mkdir -p $HOME/.andromeda/cosmovisor/upgrades/v0.1.1/bin
mv bin/andromedad $HOME/.andromeda/cosmovisor/upgrades/v0.1.1/bin/
```
