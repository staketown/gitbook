---
cover: ../../.gitbook/assets/andromeda-banner.jpeg
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
sudo systemctl stop andromedad
cp $HOME/.andromeda/data/priv_validator_state.json $HOME/.andromeda/priv_validator_state.json.backup
rm -rf $HOME/.andromeda/data

URL=https://snapshots.stake-town.com/andromeda/andromeda-1_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.andromeda

mv $HOME/.andromeda/priv_validator_state.json.backup $HOME/.andromeda/data/priv_validator_state.json

sudo systemctl restart andromedad && sudo journalctl -u andromedad -f -o cat
```

## **State Sync**

```bash
sudo systemctl stop andromedad

cp $HOME/.andromeda/data/priv_validator_state.json $HOME/.andromeda/priv_validator_state.json.backup
andromedad tendermint unsafe-reset-all --home $HOME/.andromeda --keep-addr-book

SNAP_RPC="https://andromeda-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="28876b3094518bef97a1250ef641c26b7d4a658d@88.99.208.54:39656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.andromeda/config/config.toml

CONFIG_TOML=$HOME/.andromeda/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/.andromeda/priv_validator_state.json.backup $HOME/.andromeda/data/priv_validator_state.json

sudo systemctl restart andromedad && sudo journalctl -u andromedad -f -o cat
```

## **Address Book**

```bash
curl -Ls https://snapshots.stake-town.com/andromeda/addrbook.json > $HOME/.andromeda/config/addrbook.json
```

## Genesis

```bash
curl -Ls https://snapshots.stake-town.com/andromeda/genesis.json > $HOME/.andromeda/config/genesis.json
```
