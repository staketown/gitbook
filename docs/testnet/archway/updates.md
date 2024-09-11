---
cover: ../../.gitbook/assets/archway-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v9.0.0-rc1** is available

```bash
cd $HOME || return
rm -rf archway
git clone https://github.com/archway-network/archway.git
cd archway || return
git checkout v9.0.0-rc1

make build

mkdir -p $HOME/.archway/cosmovisor/upgrades/v9.0.0/bin
mv build/archwayd $HOME/.archway/cosmovisor/upgrades/v9.0.0/bin/
```
