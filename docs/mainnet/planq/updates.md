---
cover: ../../.gitbook/assets/planq-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v2.1.2** is available

```bash
cd $HOME || return
rm -rf planq
git clone https://github.com/planq-network/planq.git
cd planq || return
git checkout v2.1.2

make build

mkdir -p $HOME/.planqd/cosmovisor/upgrades/v2.1.0/bin
mv build/planqd $HOME/.planqd/cosmovisor/upgrades/v2.1.0/bin/
```
