---
cover: ../../.gitbook/assets/archway-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v7.0.0-rc.3** is available

```bash
cd $HOME || return
rm -rf archway
git clone https://github.com/archway-network/archway.git
cd archway || return
git checkout v7.0.0-rc.3

make build

mkdir -p $HOME/.archway/cosmovisor/upgrades/v7.0.0/bin
mv build/archwayd $HOME/.archway/cosmovisor/upgrades/v7.0.0/bin/
```
