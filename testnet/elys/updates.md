---
cover: ../../.gitbook/assets/elys-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v0.7.0** is available

```bash
cd $HOME || return
URL="https://github.com/elys-network/elys/releases/download/v0.7.0/elys._v0.7.0_linux_amd64.tar.gz"
wget $URL | tar -xf elys._v0.7.0_linux_amd64.tar.gz -C $HOME/go/bin

elysd version # v0.7.0

sudo systemctl restart elysd && sudo journalctl -u elysd -f -o cat
```
