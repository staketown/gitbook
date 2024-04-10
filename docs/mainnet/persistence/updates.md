---
cover: ../../.gitbook/assets/persistence-banner.png
coverY: 0
---

# Updates

⚠️ Version **v11.9.0** is available

```bash
cd $HOME || return
rm -rf persistenceCore
git clone https://github.com/persistenceOne/persistenceCore.git
cd persistenceCore || return
git checkout v11.9.0

make build

mkdir -p $HOME/.persistenceCore/cosmovisor/upgrades/v11.9.0/bin
mv build/persistenceCore $HOME/.persistenceCore/cosmovisor/upgrades/v11.9.0/bin/
```
