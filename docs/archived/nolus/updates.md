---
cover: ../../.gitbook/assets/nolus-defi-banner.png
coverY: 0
---

# Updates

> ⚠️ Version **v0.2.2-equalize-store-heights** is available

```bash
cd $HOME/nolus-core
git fetch --all
git checkout v0.3.0
make install
nolusd version --long | head
#version: v0.3.0
sudo systemctl restart nolusd && sudo journalctl -u nolusd -f -o cat
```
