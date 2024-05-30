---
cover: ../../.gitbook/assets/persistence-banner.png
coverY: 0
---

# Updates

⚠️ Version **v11.13.0-rc0** is available

```bash
cd $HOME || return
rm -rf persistenceCore
git clone https://github.com/persistenceOne/persistenceCore.git
cd persistenceCore || return
git checkout v11.13.0-rc0

make build

mkdir -p $HOME/.persistenceCore/cosmovisor/upgrades/v11.13.0-rc0/bin
mv bin/persistenceCore $HOME/.persistenceCore/cosmovisor/upgrades/v11.13.0-rc0/bin/
```
