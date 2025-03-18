---
cover: ../../.gitbook/assets/archway-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v10.1.0** is available

```bash
cd $HOME || return
rm -rf archway
git clone https://github.com/archway-network/archway.git
cd archway || return
git checkout v10.1.0

make build

mkdir -p $HOME/.archway/cosmovisor/upgrades/v10.0.0/bin
mv build/archwayd $HOME/.archway/cosmovisor/upgrades/v10.0.0/bin/
```
