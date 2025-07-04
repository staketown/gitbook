---
cover: ../../.gitbook/assets/celestia-banner.jpeg
coverY: 0
---

# Updates

### Node updates
⚠️ Version **v4.0.7-mocha** is available

```bash
cd $HOME || return
rm -rf $HOME/celestia-app
git clone https://github.com/celestiaorg/celestia-app.git
cd $HOME/celestia-app || return
git checkout v4.0.7-mocha

make build

mv build/celestia-appd $HOME/.celestia-app/cosmovisor/genesis/bin/
```

### Bridge node updates

⚠️ Version **vv0.23.3-mocha** is available

```bash
# Stop bridge node
sudo systemctl stop celestia-bridge.service

# Download bridge node binary
cd $HOME 
rm -rf celestia-node 
git clone https://github.com/celestiaorg/celestia-node.git 
cd celestia-node
git checkout v0.23.3-mocha
make build
sudo mv build/celestia $HOME/go/bin
make cel-key
sudo mv cel-key $HOME/go/bin

# Update bridge node
celestia bridge config-update --p2p.network mocha

# Start bridge node
sudo systemctl restart celestia-bridge.service
```