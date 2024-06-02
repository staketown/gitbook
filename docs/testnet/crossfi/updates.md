---
cover: ../../.gitbook/assets/crossfi-banner.png
coverY: 0
---

# Updates

⚠️ Version **v0.3.0-prebuild9** is available

```bash
cd $HOME || return
rm -rf crossfi-node
git clone https://github.com/crossfichain/crossfi-node.git
cd crossfi-node || return
git checkout v0.3.0-prebuild9

make build

mkdir -p $HOME/.mineplex-chain/cosmovisor/upgrades/erc20-cheque-testnet/bin
mv build/crossfid $HOME/.mineplex-chain/cosmovisor/upgrades/erc20-cheque-testnet/bin/
```
