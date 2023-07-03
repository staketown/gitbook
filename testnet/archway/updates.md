---
cover: ../../.gitbook/assets/archway-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v1.0.0-rc.4** is available

```bash
cd $HOME/archway
git fetch --all
git checkout v1.0.0-rc.4
make install
archwayd version --long | head
#version: v1.0.0-rc.4

sudo systemctl restart archwayd && sudo journalctl -u archwayd -f -o cat
```
