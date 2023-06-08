---
cover: ../../.gitbook/assets/composable-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v3.0.0** is available

```bash
cd $HOME || return
rm -rf composable-testnet
git clone https://github.com/notional-labs/composable-testnet.git
cd $HOME/composable-testnet || return
git checkout $BINARY_VERSION_TAG
make install
centaurid version # v3.0.0

sudo systemctl restart centaurid && sudo journalctl -u centaurid -f -o cat
```
