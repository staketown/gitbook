---
cover: ../../.gitbook/assets/gitopia-banner.png
coverY: 0
---

# Updates

> ⚠️ Version v2.1.1 is available

```bash
cd $HOME/gitopia
git fetch --all
git checkout v2.1.1
make install
gitopiad version --long | head
#version: v2.1.1
sudo systemctl restart gitopiad && sudo journalctl -u gitopiad -f -o cat
```
