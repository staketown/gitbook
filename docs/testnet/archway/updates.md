---
cover: ../../.gitbook/assets/archway-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v6.0.0** is available

```bash
cd $HOME || return
rm -rf archway
git clone https://github.com/archway-network/archway.git
cd archway || return
git checkout v6.0.0

make build

mkdir -p $HOME/.archway/cosmovisor/upgrades/v6.0.0/bin
mv build/archwayd $HOME/.archway/cosmovisor/upgrades/v6.0.0/bin/
```
