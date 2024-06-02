---
cover: ../../.gitbook/assets/crossfi-banner.png
coverY: 0
---

# Updates

⚠️ Version **v0.3.0-prebuild9** is available

```bash
cd $HOME || return
wget https://github.com/crossfichain/crossfi-node/releases/download/v0.3.0-prebuild9/crossfi-node_0.3.0-prebuild9_linux_amd64.tar.gz
mkdir $HOME/crossfi_tmp
tar -xvf crossfi-node_0.3.0-prebuild9_linux_amd64.tar.gz -C $HOME/crossfi_tmp
mv $HOME/crossfi_tmp/bin/crossfid ~/go/bin/crossfid

rm crossfi-node_0.3.0-prebuild9_linux_amd64.tar.gz
rm -rf $HOME/crossfi_tmp

mkdir -p $HOME/.mineplex-chain/cosmovisor/upgrades/erc20-cheque-testnet/bin
cp ~/go/bin/crossfid $HOME/.mineplex-chain/cosmovisor/upgrades/erc20-cheque-testnet/bin/
```
