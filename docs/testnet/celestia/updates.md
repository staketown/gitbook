---
cover: ../../.gitbook/assets/celestia-banner.jpeg
coverY: 0
---

# Updates

### Node updates
⚠️ **No updates so far**


### Bridge node updates

⚠️ Version **v0.13.3** is available

```bash
# Stop bridge node
sudo systemctl stop celestia-bridge.service

# Download bridge node binary
cd $HOME 
rm -rf celestia-node 
git clone https://github.com/celestiaorg/celestia-node.git 
cd celestia-node
git checkout v0.13.3
make build
sudo mv build/celestia $HOME/go/bin
make cel-key
sudo mv cel-key $HOME/go/bin

# Update bridge node
celestia bridge config-update --p2p.network mocha

# Start bridge node
sudo systemctl restart celestia-bridge.service
```