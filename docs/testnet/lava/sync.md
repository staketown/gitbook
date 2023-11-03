---
cover: ../../.gitbook/assets/banner_cleanup.jpeg
coverY: 0
---

# Sync

## **Snapshot**

> updated every 6 hours, starting from 00:00 UTC

```bash
# install dependencies, if needed
sudo apt update && sudo apt install lz4 -y
```

```bash
sudo systemctl stop lavad
cp $HOME/.lava/data/priv_validator_state.json $HOME/.lava/priv_validator_state.json.backup
rm -rf $HOME/.lava/data

URL="https://snapshots-testnet.stake-town.com/lava/lava-testnet-2_latest.tar.lz4"
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.lava

mv $HOME/.lava/priv_validator_state.json.backup $HOME/.lava/data/priv_validator_state.json 

sudo systemctl restart lavad && sudo journalctl -u lavad -f -o cat
```

## **State Sync**

> Lava Network doesn’t support state-sync at that moment

## **Address Book**

```bash
curl -Ls https://snapshots-testnet.stake-town.com/lava/addrbook.json > $HOME/.lava/config/addrbook.json
```

## Genesis

```bash
curl -Ls https://snapshots-testnet.stake-town.com/lava/genesis.json > $HOME/.lava/config/genesis.json
```
