---
cover: ../../.gitbook/assets/gitopia-banner.png
coverY: 0
---

# Updates

> ⚠️ Version v2.1.0 is available

```bash
cd $HOME/gitopia
git fetch --all
git checkout v2.1.0
make install
gitopiad version --long | head
#version: v2.1.0
sudo systemctl restart gitopiad && sudo journalctl -u gitopiad -f -o cat
```
