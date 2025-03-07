---
cover: ../../.gitbook/assets/warden-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v0.6.2** is available

```bash
cd $HOME
wget -O ~/go/bin/wardend https://github.com/warden-protocol/wardenprotocol/releases/download/v0.6.2/wardend-0.6.2-linux-amd64
chmod +x ~/go/bin/wardend

mkdir -p $HOME/.warden/cosmovisor/upgrades/v054-to-v060/bin
cp ~/go/bin/wardend $HOME/.warden/cosmovisor/upgrades/v054-to-v060/bin/
```
