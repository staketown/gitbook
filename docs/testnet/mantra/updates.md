---
cover: ../../.gitbook/assets/mantra-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v3.0.0** is available

```bash
wget https://github.com/MANTRA-Finance/public/releases/download/v3.0.0/mantrachaind-3.0.0-linux-amd64.tar.gz
tar -xvf mantrachaind-3.0.0-linux-amd64.tar.gz
rm mantrachaind-3.0.0-linux-amd64.tar.gz
mv mantrachaind $HOME/go/bin

mkdir -p $HOME/.mantrachain/cosmovisor/upgrades/v3.0.0/bin
cp $HOME/go/bin/mantrachaind $HOME/.mantrachain/cosmovisor/upgrades/v3.0.0/bin/
```
