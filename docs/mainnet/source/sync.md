---
cover: ../../.gitbook/assets/source_banner.webp
coverY: 0
---

# Sync

## **Snapshots**

> updated every 6 hours, starting from 00:00 UTC

```bash
# install dependencies, if needed
sudo apt update && sudo apt install lz4 -y
```

```bash
sudo systemctl stop sourced

cp $HOME/.source/data/priv_validator_state.json $HOME/.source/priv_validator_state.json.backup
rm -rf $HOME/.source/data

# Add snapshot here
URL="https://snapshots.stake-town.com/source/source-1_latest.tar.lz4"
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.source

mv $HOME/.source/priv_validator_state.json.backup $HOME/.source/data/priv_validator_state.json

sudo systemctl restart sourced && sudo journalctl -u sourced -f -o cat
```

## **State Sync**

```bash
sudo systemctl stop sourced

cp $HOME/.source/data/priv_validator_state.json $HOME/.source/priv_validator_state.json.backup
sourced tendermint unsafe-reset-all --home $HOME/.source --keep-addr-book

SNAP_RPC="https://source-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="afc8fa287e2b6b46bbeba57dfcb4bd6dcab6b6a3@88.99.208.54:28656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.source/config/config.toml

CONFIG_TOML=$HOME/.source/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/.source/priv_validator_state.json.backup $HOME/.source/data/priv_validator_state.json

sudo systemctl restart sourced
sudo journalctl -u sourced -f -o cat
```

## **Address Book**

```bash
curl -Ls https://snapshots.stake-town.com/source/addrbook.json > $HOME/.source/config/addrbook.json
```

## Genesis

```bash
curl -Ls https://snapshots.stake-town.com/source/genesis.json > $HOME/.source/config/genesis.json
```
