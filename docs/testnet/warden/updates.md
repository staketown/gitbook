---
cover: ../../.gitbook/assets/warden-banner.jpeg
coverY: 0
---

# Updates

⚠️ Version **v0.4.1** is available

```bash
cd $HOME
wget https://github.com/warden-protocol/wardenprotocol/releases/download/v0.4.1/wardend_Linux_x86_64.zip
unzip wardend_Linux_x86_64.zip -d ~/temp_warden
mv ~/temp_warden/wardend ~/go/bin/
rm -rf ~/temp_warden
rm ~/wardend_Linux_x86_64.zip

mkdir -p $HOME/.warden/cosmovisor/upgrades/v03-to-v04/bin
cp ~/go/bin/wardend $HOME/.warden/cosmovisor/upgrades/v03-to-v04/bin/
```
