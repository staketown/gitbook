---
cover: ../../.gitbook/assets/archway-banner.jpeg
coverY: 0
---

# Updates

⚠️ **v4.0.2 is available**

```bash
cd $HOME || return
rm -rf archway
git clone https://github.com/archway-network/archway.git
cd archway || return
git checkout v4.0.2

make build

mkdir -p $HOME/.archway/cosmovisor/upgrades/v4.0.2/bin
mv build/archwayd $HOME/.archway/cosmovisor/upgrades/v4.0.2/bin/

rm -rf build
```
