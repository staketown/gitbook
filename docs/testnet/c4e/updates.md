---
cover: ../../.gitbook/assets/c4e-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v1.3.0** is available

```bash
cd $HOME/c4e-chain || return
git fetch --all
git checkout v1.3.0
make install
c4ed version --long | head
#version: v1.3.0

sudo systemctl restart c4ed && sudo journalctl -u c4ed -f -o cat
```
