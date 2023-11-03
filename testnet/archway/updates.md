---
cover: ../../.gitbook/assets/archway-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v4.0.0** is available

```bash
cd $HOME/archway
git fetch --all
git checkout v4.0.0
make install
archwayd version --long | head
#version: v4.0.0

sudo systemctl restart archwayd && sudo journalctl -u archwayd -f -o cat
```
