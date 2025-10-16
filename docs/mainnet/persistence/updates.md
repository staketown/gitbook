---
cover: ../../.gitbook/assets/persistence-banner.png
coverY: 0
---

# Updates

⚠️ Version **v14.0.1** is available

```bash
cd $HOME || return
rm -rf persistenceCore
git clone https://github.com/persistenceOne/persistenceCore.git
cd persistenceCore || return
git checkout v14.0.1

make build

mkdir -p $HOME/.persistenceCore/cosmovisor/upgrades/v14.0.0/bin
mv bin/persistenceCore $HOME/.persistenceCore/cosmovisor/upgrades/v14.0.0/bin/
```
