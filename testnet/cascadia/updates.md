---
cover: ../../.gitbook/assets/cascadia-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v0.1.2** is available

```bash
cd $HOME/cascadia
git fetch --all
git checkout v0.1.2
make install
cascadiad version --long | head
#version: v0.1.2
sudo systemctl restart cascadiad && sudo journalctl -u cascadiad -f -o cat
```
