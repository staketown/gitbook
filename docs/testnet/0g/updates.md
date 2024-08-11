---
cover: ../../.gitbook/assets/0g-banner.jpg
coverY: 0
---

# Updates

⚠️ Version **v0.3.1.alpha.0** is available

```bash
cd $HOME || return
rm -rf 0g-chain
git clone https://github.com/0glabs/0g-chain.git
cd 0g-chain || return
git checkout v0.3.1.alpha.0

make build

mkdir -p $HOME/.0gchain/cosmovisor/upgrades/v0.3.1/bin
mv out/linux/0gchaind $HOME/.0gchain/cosmovisor/upgrades/v0.3.1/bin/
```
