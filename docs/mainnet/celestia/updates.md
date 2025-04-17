---
cover: ../../.gitbook/assets/celestia-banner.jpeg
coverY: 0
---

# Updates

### Node updates
⚠️ Version **v3.8.1** is available

```bash
cd $HOME || return
rm -rf $HOME/celestia-app
git clone https://github.com/celestiaorg/celestia-app.git
cd $HOME/celestia-app || return
git checkout v3.8.1

make build

mv build/celestia-appd $HOME/.celestia-app/cosmovisor/genesis/bin/
```

### Bridge node updates

⚠️ Version **v0.21.9** is available (Shwap)

```bash
# Stop bridge node
sudo systemctl stop celestia-bridge.service

# Download bridge node binary
cd $HOME 
rm -rf celestia-node 
git clone https://github.com/celestiaorg/celestia-node.git 
cd celestia-node
git checkout v0.21.9
make build
sudo mv build/celestia $HOME/go/bin
make cel-key
sudo mv cel-key $HOME/go/bin

# Update bridge node
celestia bridge config-update --p2p.network celestia

# Start bridge node
sudo systemctl restart celestia-bridge.service
```