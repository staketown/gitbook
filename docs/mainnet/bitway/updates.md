---
cover: ../../.gitbook/assets/bitway-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v2.0.1** is available

```bash
cd $HOME || return 
rm -rf $HOME/bitway
git clone https://github.com/bitwaylabs/bitway.git
cd $HOME/bitway || return
git checkout v2.0.1

make build

mkdir -p $HOME/.bitway/cosmovisor/upgrades/v2.0.1/bin
mv build/bitwayd $HOME/.bitway/cosmovisor/upgrades/v2.0.1/bin/
```

