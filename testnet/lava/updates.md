---
cover: ../../.gitbook/assets/banner_cleanup.jpeg
coverY: 0
---

# Updates

> ⚠️ Version v0.13.1 is available

```bash
cd $HOME/lava
git fetch --all
git checkout v0.13.1
make install
lavad version --long | head
#version: v0.13.1
sudo systemctl restart lavad && sudo journalctl -u lavad -f -o cat
```
