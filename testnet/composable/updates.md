---
cover: ../../.gitbook/assets/composable-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v3.0.3-testnet** is available

```bash
cd $HOME || return
rm -rf composable-testnet
git clone https://github.com/notional-labs/composable-testnet.git
cd $HOME/composable-testnet || return
git checkout v3.0.3-testnet
make install
centaurid version # v3.0.3-testnet

sudo systemctl restart centaurid && sudo journalctl -u centaurid -f -o cat
```
