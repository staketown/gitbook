---
cover: ../../.gitbook/assets/archway-banner.jpeg
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
sudo systemctl stop archwayd
cp $HOME/.archway/data/priv_validator_state.json $HOME/.archway/priv_validator_state.json.backup
rm -rf $HOME/.archway/data

URL=https://snapshots.stake-town.com/archway/archway-1_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.archway

mv $HOME/.archway/priv_validator_state.json.backup $HOME/.archway/data/priv_validator_state.json

sudo systemctl restart archwayd && sudo journalctl -u archwayd -f -o cat
```

## **State Sync**

```bash
sudo systemctl stop archwayd

cp $HOME/.archway/data/priv_validator_state.json $HOME/.archway/priv_validator_state.json.backup
archwayd tendermint unsafe-reset-all --home $HOME/.archway --keep-addr-book

SNAP_RPC="https://archway-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="218eed47b5472642034e81fdf408dec8b79dcba7@88.99.208.54:31656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.archway/config/config.toml

CONFIG_TOML=$HOME/.archway/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/.archway/priv_validator_state.json.backup $HOME/.archway/data/priv_validator_state.json

sudo systemctl restart archwayd && sudo journalctl -u archwayd -f -o cat
```

## **Address Book**

```bash
curl -Ls https://snapshots.stake-town.com/archway/addrbook.json > $HOME/.archway/config/addrbook.json
```

## Genesis

```bash
curl -Ls https://snapshots.stake-town.com/archway/genesis.json > $HOME/.archway/config/genesis.json
```
